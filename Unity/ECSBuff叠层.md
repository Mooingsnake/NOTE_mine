# ECS Buff 叠层、持续时间刷新与周期伤害实现指南

## 1. 系统入口

本项目的新版 Buff 使用 Friflo ECS 实现。业务侧通过以下接口添加 Buff：

```csharp
gameLogic.coreEcsWorld.buffModule.TryCreateBuff(
    templateId,
    owner,
    source,
    applicator,
    out Entity buffEntity);
```

相关核心代码：

- `ECSBuffSystem/000ECS/ECSBuffModule.cs`
- `ECSBuffSystem/000ECS/BuffSystem/BuffDurationSystem.cs`
- `ECSBuffSystem/000ECS/BuffSystem/BuffRepeatInvokeAfterSomeTimeTriggerSystem.cs`
- `ECSBuffSystem/000ECS/BuffSystem/BuffEffectExecuteSystem.cs`
- `Jinn/000ECS/ECSBuffEffectExecutor/`

战斗阶段每帧调用 `GameLogic.TickCoreEcsWorld(dt)`，最终执行：

```csharp
coreEcsWorld.Update(dt);
```

## 2. Buff Entity 组成

一个支持叠层、持续时间和周期伤害的 Buff Entity 通常包含：

```text
Buff Entity
├── BuffTag
├── BuffTemplateComponent
├── BuffOwnerComponent
├── BuffSourceComponent（可选）
├── BuffApplicatorComponent（可选）
├── BuffDurationComponent
├── BuffStackComponent
└── Timer Entity
    └── BuffRepeatInvokeAfterSomeTimeComponent
```

各组件职责：

| 组件 | 职责 |
| --- | --- |
| `BuffDurationComponent` | 保存配置时长和剩余时长 |
| `BuffStackComponent` | 保存当前层数、最大层数和每层计时策略 |
| `BuffOwnerComponent` | 保存 Buff 承受者 |
| `BuffSourceComponent` | 保存伤害或效果来源 |
| `BuffApplicatorComponent` | 保存 Buff 施加者 |
| `BuffRepeatInvokeAfterSomeTimeComponent` | 保存周期触发间隔和累计时间 |

## 3. Buff 叠层配置

在 `ecs_component_template_table` 中配置叠层组件：

```text
type = BuffStackComponent
parameter_floats[0] = 最大层数
parameter_int[0] = BuffStackStrategyEnum
```

在 `ecs_buff_entity_template_table` 中配置同类 Buff 策略：

```text
strategy_handle_owner_got_same_id = AllowStack
```

当同一个 Owner 再次获得相同 `templateId` 时，`ECSBuffModule` 会找到原 Buff Entity，而不是创建新实体：

```csharp
if (stack.stackCount < stack.maxStack)
{
    stack.stackCount++;
}

BuffEffectTriggerEntityFactory.Invoke(
    buffEntity,
    triggerType: ECSBuffTriggerType.WhenStack);
```

达到 `maxStack` 后不会继续增加层数，但仍会按照计时策略处理持续时间，并触发 `WhenStack`。

> `AllowStack` 必须同时配置 `BuffStackComponent`，否则叠层操作会被跳过。

## 4. 持续时间策略

### 4.1 UsingNewest：刷新为完整时长

每次叠层都将剩余时间重置为配置时长：

```csharp
duration.remainingDuration = duration.duration;
```

示例：Buff 持续 10 秒，在剩余 4 秒时再次施加，剩余时间重新变为 10 秒。

该策略适用于中毒、燃烧等“续层时整体刷新”的效果。

### 4.2 Independent：每层独立计时

每层维护自己的剩余时间：

```text
第 1 层：剩余 3 秒
第 2 层：剩余 6 秒
第 3 层：剩余 9 秒
```

某一层到期时，只移除该层：

```csharp
stack.independentRemainingDurations.RemoveAt(i);
stack.stackCount--;
```

所有层均到期后才销毁整个 Buff。满层后再次施加时，当前实现会刷新剩余时间最短的一层。

### 4.3 UsingOldest：保留首次时长

后续叠层不刷新持续时间。例如持续 10 秒的 Buff 在第 6 秒叠层后，仍然只剩 4 秒。

### 4.4 AddPrevDuration：累计持续时间

如果重复获得 Buff 时只需要增加时长，不增加层数，可配置：

```text
strategy_handle_owner_got_same_id = AddPrevDuration
```

对应实现：

```csharp
duration.remainingDuration += duration.duration;
```

例如原来剩余 3 秒，Buff 配置时长为 5 秒，再次施加后剩余 8 秒。

## 5. Buff 到期与销毁

`BuffDurationSystem` 每帧扣减剩余时间：

```csharp
duration.remainingDuration -= dt;
```

普通 Buff 时间归零后执行：

```csharp
BuffEffectTriggerEntityFactory.Invoke(
    entity,
    triggerType: ECSBuffTriggerType.WhenExpire);

ecsWorld.buffModule.DestroyBuffEntity(entity);
```

所以自然到期的触发顺序通常是：

```text
WhenExpire
    ↓
删除 Buff Entity
    ↓
WhenDestroy
```

对于 `Independent` 策略，系统先逐层扣减时间；只有 `stackCount` 降为 `0` 时，整个 Buff 才会过期。

## 6. 周期伤害触发器

在 `ecs_buff_trigger_table` 中增加周期触发器：

```text
name_id = poison_tick
type = RepeatInvokeAfterSomeTime
parameter_floats = [1.0]
```

`parameter_floats[0]` 表示触发间隔，单位为秒。上例表示每秒触发一次。

然后在 Buff 模板中把触发器映射到伤害效果：

```text
trigger_and_effects = poison_tick[poison_damage]
```

创建 Buff 时，`ECSBuffModule` 会为周期触发器创建 Timer 子实体：

```csharp
var component = new BuffRepeatInvokeAfterSomeTimeComponent(
    buffEntity.Id,
    triggerConfigId,
    intervalSeconds);
```

周期系统累积帧时间并补齐应触发的次数：

```csharp
component.elapsedSeconds += dt;

while (component.elapsedSeconds >= component.delaySeconds)
{
    component.elapsedSeconds -= component.delaySeconds;
    invokeCount++;
}
```

使用 `while` 可以在某一帧 `dt` 较大时补发多次触发，避免丢失伤害 Tick。

## 7. 周期伤害 Effect

### 7.1 当前实现状态

周期触发框架已经完成，但现有 `CreateBattleAttackExecutor` 仍为空实现：

```csharp
public bool Execute(in ECSBuffEffectContext context)
{
    return false;
}
```

因此，仅配置 `RepeatInvokeAfterSomeTime` 不会自动造成伤害，还需要实现对应的 `IECSBuffEffectExecutor`。

### 7.2 读取 Buff 上下文和层数

执行器可以通过触发实体找到原 Buff Entity：

```csharp
Entity buffEntity =
    ECSBuffEffectUtility.GetBuffEntity(context.triggerEntity);

ref BuffOwnerComponent ownerComponent =
    ref buffEntity.GetComponent<BuffOwnerComponent>();

IEntityLogicConnector source = null;
if (buffEntity.HasComponent<BuffSourceComponent>())
{
    source = buffEntity.GetComponent<BuffSourceComponent>().source;
}

int stackCount = buffEntity.HasComponent<BuffStackComponent>()
    ? buffEntity.GetComponent<BuffStackComponent>().stackCount
    : 1;
```

推荐的基础伤害公式：

```text
本次周期伤害 = 单层伤害 × 当前层数
```

```csharp
float damagePerStack = context.config.ParameterFloats[0];
float finalDamage = damagePerStack * stackCount;
```

之后应构造项目统一的 `DamageRuntimeState`，并调用 `DamageModule.ApplyDamage`：

```text
defender = BuffOwnerComponent.owner
source    = BuffSourceComponent.source
attacker  = BuffApplicatorComponent.applicator 或 BuffSourceComponent.source
damage    = effect 参数 × 当前层数
```

不要在执行器中直接扣减生命值，否则会绕过以下逻辑：

- 伤害增减和免疫
- `BeforeOwnerApplyDamage`
- `BeforeOwnerTakeDamage`
- `AfterOwnerApplyDamage`
- `AfterOwnerTakeDamage`
- 原版 Buff 伤害处理
- 死亡及伤害结果处理

可参考 `ChangeBaseDamageMulInRuntimeWithStackExecutor` 获取 Buff 层数的方式。

## 8. 完整配置示例

以下示例实现一个“最多 5 层、持续 10 秒、续层刷新时间、每秒造成每层 8 点伤害”的中毒 Buff。

### 8.1 持续时间组件

```text
name_id = duration_10s
type = BuffDurationComponent
parameter_floats = [10.0]
```

### 8.2 叠层组件

```text
name_id = stack_5_using_newest
type = BuffStackComponent
parameter_floats = [5]
parameter_int = [UsingNewest]
```

### 8.3 周期触发器

```text
name_id = poison_tick
type = RepeatInvokeAfterSomeTime
parameter_floats = [1.0]
```

### 8.4 周期伤害效果

```text
name_id = poison_damage
effect_enum = 自定义周期伤害 Effect
parameter_floats = [8.0]
```

### 8.5 Buff 模板

```text
strategy_handle_owner_got_same_id = AllowStack
components = duration_10s | stack_5_using_newest
trigger_and_effects = poison_tick[poison_damage]
```

运行效果：

| 操作 | 层数 | 剩余时间 | 每次周期伤害 |
| --- | ---: | ---: | ---: |
| 第一次施加 | 1 | 10 秒 | 8 |
| 第二次施加 | 2 | 刷新为 10 秒 | 16 |
| 第五次施加 | 5 | 刷新为 10 秒 | 40 |
| 满层后再次施加 | 5 | 刷新为 10 秒 | 40 |
| 10 秒内未续层 | Buff 到期销毁 | 0 | 停止触发 |

## 9. 执行流程

```text
TryCreateBuff
    ↓
检查同 Owner、同 templateId 的 Buff
    ↓
AllowStack → 增加层数并刷新/增加独立计时
    ↓
每帧 CoreEcsWorld.Update(dt)
    ├── BuffDurationSystem：更新持续时间和层数到期
    └── BuffRepeatInvokeAfterSomeTimeTriggerSystem：产生周期触发
            ↓
        BuffEffectExecuteSystem
            ↓
        IECSBuffEffectExecutor
            ↓
        DamageModule.ApplyDamage
```

## 10. 注意事项

- Buff 时间和周期触发间隔的单位都是秒。
- 战斗固定帧间隔由 `GameConst.UpdateInterval` 控制，当前为 20ms。
- `RepeatInvokeAfterSomeTime` 的间隔必须大于 `0`，否则系统不会触发。
- 周期伤害执行器应处理 Owner、Source 或 Applicator 已失效的情况。
- Buff 销毁后，周期 Timer Entity 会停止工作并被清理。
- 若需要快照或断线重连，应同时保存层数、剩余时间和独立层计时列表。
- 不要手工修改 `Model/Generate/Config/GameConfig/` 下的生成文件；应修改 ECS Buff 配置表后重新生成。

