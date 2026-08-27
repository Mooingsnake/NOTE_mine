# C# 垃圾回收机制与 Unity 内存管理

> 本文重点介绍 C#/.NET 垃圾回收器的工作原理、分代 GC、对象存活判定，以及 Unity 开发中如何减少托管分配、正确释放非托管资源并避免大对象引起的 GC 卡顿。

## 目录

- [1. 基本概念：GC 管理什么](#1-基本概念gc-管理什么)
- [2. GC 如何判断对象是垃圾](#2-gc-如何判断对象是垃圾)
- [3. 标准 .NET 的分代 GC](#3-标准-net-的分代-gc)
- [4. 大对象堆 LOH](#4-大对象堆-loh)
- [5. 压缩、碎片与固定对象](#5-压缩碎片与固定对象)
- [6. 终结器、Dispose 与弱引用](#6-终结器dispose-与弱引用)
- [7. Unity GC 与标准 .NET GC 的区别](#7-unity-gc-与标准-net-gc-的区别)
- [8. Unity 中常见的托管分配来源](#8-unity-中常见的托管分配来源)
- [9. Unity 非托管资源管理](#9-unity-非托管资源管理)
- [10. Addressables、AssetBundle 与 Resources](#10-addressablesassetbundle-与-resources)
- [11. 大对象与对象池策略](#11-大对象与对象池策略)
- [12. 手动 GC 与增量 GC](#12-手动-gc-与增量-gc)
- [13. Profiler 排查流程](#13-profiler-排查流程)
- [14. 实践检查清单](#14-实践检查清单)
- [15. 总结](#15-总结)

---

## 1. 基本概念：GC 管理什么

C# 程序中的内存可以粗略分为三类。

### 1.1 栈内存

栈通常保存方法参数、局部变量、对象引用本身和方法调用信息。方法返回后，栈帧会直接弹出，不需要 GC 逐个回收。

```csharp
void Foo()
{
    int number = 10;
    Player player = new Player();
}
```

一般可以理解为：

- `number` 和 `player` 引用位于当前栈帧；
- `new Player()` 创建的对象位于托管堆。

但“值类型一定在栈上”并不正确。值类型作为引用类型对象的字段时，会成为该堆对象的一部分；运行时优化也可能改变具体存储位置。

### 1.2 托管堆

通过 `new` 创建的引用类型对象通常位于托管堆，例如：

```csharp
new Player();
new int[100];
new List<int>();
```

这些对象由 GC 管理。程序不能像 C/C++ 那样直接 `free` 某个普通托管对象。

### 1.3 非托管内存与 Unity Native 内存

以下资源不能依赖普通托管 GC 及时释放：

- `NativeArray<T>` 等 NativeContainer；
- `UnsafeUtility.Malloc` 分配的内存；
- 文件、Socket 和操作系统句柄；
- 原生插件创建的资源；
- Texture、Mesh、Material、AudioClip 等 Unity Native 资源；
- AssetBundle 和部分 Addressables 底层资源。

它们通常需要显式调用 `Dispose()`、`Destroy()`、`Addressables.Release()` 或 `AssetBundle.Unload()`。

---

## 2. GC 如何判断对象是垃圾

GC 的核心判断规则是：

> 从 GC Roots 出发，如果无法通过引用关系到达某个对象，那么该对象就是垃圾。

这称为**可达性分析（Reachability Analysis）**。

### 2.1 常见 GC Roots

GC Roots 通常包括：

- 当前线程栈上的有效对象引用；
- 静态字段引用；
- CPU 寄存器中的引用；
- `GCHandle` 和运行时内部句柄；
- 终结队列涉及的对象。

静态字段和静态事件是 Unity 中常见的意外保活来源：即使场景已经卸载，只要静态字段、单例或事件仍然引用某个对象，它就仍然可达。

### 2.2 对象引用图

```text
GC Root
   │
   ▼
Player
   │
   ▼
Inventory
   │
   ▼
Item[]
```

只要 `Player` 可达，`Inventory` 和 `Item[]` 也都可达，不能被 GC 回收。

如果另一组对象是：

```text
OldEnemy → OldWeapon
```

而任何 GC Root 都无法到达 `OldEnemy`，那么 `OldEnemy` 和 `OldWeapon` 都属于垃圾。

### 2.3 循环引用不是问题

```text
A → B
↑   ↓
└───┘
```

只要没有 GC Root 能到达 `A` 或 `B`，整个循环都可以回收。C# GC 是追踪式 GC，不是简单引用计数，因此能够处理循环引用。

### 2.4 简化的标记过程

```text
1. 将所有对象视为未标记。
2. 从每一个 GC Root 开始遍历引用。
3. 标记所有能够到达的对象。
4. 未被标记的对象就是垃圾。
```

实际实现还会涉及并行标记、后台 GC、增量标记、写屏障、卡表、记忆集和固定对象等机制，但核心仍是“从根是否可达”。

---

## 3. 标准 .NET 的分代 GC

标准现代 .NET/CoreCLR 使用分代 GC。它建立在一个经验规律上：

> 大多数新对象很快死亡，而已经存活较长时间的对象通常会继续存活。

### 3.1 Gen 0

绝大多数新建小对象首先进入 Gen 0。Gen 0 容量较小、收集最频繁。

当 Gen 0 的分配预算耗尽时，GC 会标记其中的存活对象并回收死亡对象。幸存对象通常提升到 Gen 1。

### 3.2 Gen 1

Gen 1 是短命对象和长命对象之间的缓冲层。收集 Gen 1 时，也会同时收集 Gen 0。再次幸存的对象通常提升到 Gen 2。

### 3.3 Gen 2

Gen 2 保存长期存活的对象，例如应用级服务、全局配置和长期缓存。Gen 2 收集通常是 Full GC 的一部分，扫描范围和存活对象数量更大，因此停顿往往更明显。

### 3.4 跨代引用和写屏障

老年代对象可能引用新对象：

```csharp
longLivedObject.Current = temporaryObject;
```

如果收集 Gen 0 时完全不检查老年代，就可能错误回收 `temporaryObject`。运行时通过**写屏障**记录老对象对年轻对象的引用，并借助卡表或记忆集，在年轻代 GC 时只检查相关老年代区域。

### 3.5 分代流转示意

```text
新建小对象 → Gen 0 → Gen 1 → Gen 2
                         │
                         └─ 长期存活
```

收集某一代时，会同时收集所有更年轻的代。Gen 2 GC 通常意味着全托管堆收集。

---

## 4. 大对象堆 LOH

在标准现代 .NET 中，对象大小达到或超过约 **85,000 字节**时，通常进入大对象堆（Large Object Heap，LOH）。

```csharp
byte[] buffer = new byte[100_000];
```

阈值看的是对象总大小，包括对象头、数组元数据和内存对齐，而不只是元素数据之和。

LOH 的特点包括：

- 通常随 Gen 2 一起收集；
- 默认不会像小对象堆那样频繁压缩；
- 大对象分配需要初始化和清零内存；
- 频繁分配不同尺寸的大对象容易造成碎片；
- Full GC 和进程工作集可能因此增大。

危险示例：

```csharp
for (int i = 0; i < count; i++)
{
    byte[] data = new byte[2 * 1024 * 1024];
    Process(data);
}
```

常见优化方式：

- 复用固定缓冲区；
- 使用 `ArrayPool<T>`；
- 使用有上限的对象池；
- 分块或流式处理；
- 控制并发加载和解码数量。

> 85KB LOH 是标准 .NET/CoreCLR 的规则，不应直接视为 Unity Boehm GC 的统一跨版本契约。但频繁创建大数组、大字符串或大型对象图，在 Unity 中同样会造成严重分配和扫描压力。

---

## 5. 压缩、碎片与固定对象

标准 .NET GC 可以在回收后移动存活对象，将空闲空间合并：

```text
压缩前：[A][空][B][空][空][C]
压缩后：[A][B][C][连续空闲区域]
```

压缩的优点：

- 减少碎片；
- 形成连续可分配空间；
- 改善内存局部性。

代价是对象地址发生变化，GC 必须更新所有相关引用。

当托管对象被固定时，GC 不能移动它：

```csharp
var handle = GCHandle.Alloc(buffer, GCHandleType.Pinned);
try
{
    IntPtr pointer = handle.AddrOfPinnedObject();
}
finally
{
    handle.Free();
}
```

大量、长期固定的小对象会阻碍压缩并增加碎片。标准 .NET 中应尽量缩短固定时间；长期 Native 交互可以考虑专用的非托管缓冲区。

---

## 6. 终结器、Dispose 与弱引用

### 6.1 终结器

有终结器的对象首次变为不可达后，通常不会立即释放，而是先进入终结队列：

```csharp
class NativeResource
{
    ~NativeResource()
    {
        // 最后的非托管资源清理保险
    }
}
```

终结器会延长对象生命周期，而且执行时机不可预测，因此不应作为正常释放路径。

### 6.2 Dispose 模式

非托管资源应优先显式释放：

```csharp
using var stream = File.OpenRead(path);
// 使用 stream
```

自定义资源类可实现 `IDisposable`，成功释放后调用 `GC.SuppressFinalize(this)`，终结器只作为遗漏 `Dispose` 时的最后保险。

### 6.3 弱引用

`WeakReference<T>` 不会像普通强引用一样保活对象。只剩弱引用时，对象仍然可以被 GC 回收。弱引用适合部分缓存场景，但不能代替明确的生命周期管理。

---

## 7. Unity GC 与标准 .NET GC 的区别

Unity 2021 LTS+ 常用的 Mono/IL2CPP 运行时主要使用 Boehm–Demers–Weiser GC。其特征通常是：

- 非标准 CoreCLR 分代模型；
- 非移动、非压缩；
- 支持增量收集；
- 堆扩张后通常不会立即完整归还给操作系统。

### 7.1 Unity GC 通常不是分代 GC

不要假设“这一帧产生的临时对象只进入 Gen 0，所以回收很便宜”。Unity 中大量小型临时对象、装箱和闭包仍然会显著增加扫描和回收压力。

### 7.2 非压缩带来的影响

Unity GC 通常不移动托管对象，减少了对象搬迁成本，但也意味着：

- 托管堆更容易碎片化；
- 不同尺寸对象反复分配后，空闲块可能难以复用；
- 当前存活对象减少，不代表进程占用立即下降。

### 7.3 Incremental GC 不是分代 GC

增量 GC 将一次较大的标记工作拆到多个帧：

```text
帧 1：标记一部分
帧 2：继续标记
帧 3：继续标记
帧 4：完成标记和清扫
```

它主要用于削平单次停顿，不会减少垃圾创建本身的成本。如果分配速度超过清理速度，托管堆仍会持续扩张，最终仍可能发生明显卡顿。

---

## 8. Unity 中常见的托管分配来源

### 8.1 每帧创建集合

错误：

```csharp
void Update()
{
    var visibleEnemies = new List<Enemy>();
    FindVisibleEnemies(visibleEnemies);
}
```

改为复用：

```csharp
private readonly List<Enemy> visibleEnemies = new();

void Update()
{
    visibleEnemies.Clear();
    FindVisibleEnemies(visibleEnemies);
}
```

### 8.2 返回新数组或集合

应优先让调用方提供可复用容器：

```csharp
void FindEnemies(List<Enemy> results)
{
    results.Clear();
    // 填充 results
}
```

### 8.3 数组型 Unity API

某些 Unity 属性每次访问都会创建数组副本。不要在循环中反复读取：

```csharp
Vector3[] vertices = mesh.vertices;
for (int i = 0; i < vertices.Length; i++)
{
    Vector3 vertex = vertices[i];
}
```

如果当前 Unity API 提供 List 或 NonAlloc 版本，应优先复用容器，例如：

- `Physics.RaycastNonAlloc`；
- `Physics.OverlapSphereNonAlloc`；
- `GetComponents(List<T>)`；
- `Mesh.GetVertices(List<Vector3>)`；
- `Input.touchCount` 配合 `Input.GetTouch(i)`。

NonAlloc 缓冲区必须处理容量不足和结果截断问题。

### 8.4 字符串和 UI

不要在值没有变化时每帧更新文本：

```csharp
private int displayedScore = -1;

void UpdateScore(int score)
{
    if (displayedScore == score)
        return;

    displayedScore = score;
    scoreText.text = $"Score: {score}";
}
```

复杂文本可复用 `StringBuilder`，TextMeshPro 可使用当前版本提供的低分配格式化 API。

### 8.5 LINQ

LINQ 可能创建迭代器、委托、闭包、中间集合和排序缓存。在菜单或低频逻辑中可以正常使用，但在 `Update`、AI 查询、物理循环和高频 UI 刷新中应谨慎。

### 8.6 闭包和委托

捕获外部变量的 Lambda 通常会生成隐藏对象。不要在每帧反复创建捕获 Lambda；不捕获状态的比较器或回调可以缓存为静态委托。

### 8.7 装箱

值类型传给 `object` 参数时可能装箱：

```csharp
int value = 10;
object boxed = value;
```

应优先使用泛型 API，避免非泛型集合，并通过 Profiler 或反编译确认热点中的装箱。

### 8.8 `params` 参数

```csharp
void LogValues(params object[] values)
```

调用时可能创建 `object[]`，值类型参数还可能装箱。高频路径可以提供固定参数重载或采用预分配结构。

### 8.9 协程等待对象

固定时长的等待对象可以缓存：

```csharp
private readonly WaitForSeconds wait = new(0.5f);

IEnumerator Loop()
{
    while (true)
    {
        yield return wait;
    }
}
```

动态等待时长不能错误复用固定的等待对象。

---

## 9. Unity 非托管资源管理

### 9.1 NativeArray

```csharp
var data = new NativeArray<float>(1024, Allocator.TempJob);
try
{
    // 使用 data
}
finally
{
    if (data.IsCreated)
        data.Dispose();
}
```

分配器大致用途：

- `Allocator.Temp`：极短生命周期；
- `Allocator.TempJob`：短期 Job 使用，需在规定时间内释放；
- `Allocator.Persistent`：长期存活，必须显式释放。

如果 NativeArray 仍被 Job 使用，不能提前释放。可以把释放排在依赖之后：

```csharp
JobHandle handle = job.Schedule();
JobHandle disposeHandle = data.Dispose(handle);
```

### 9.2 UnityEngine.Object

Unity 对象通常同时包含 C# 托管包装和 C++ Native 对象。只写：

```csharp
enemy = null;
```

不会立即销毁 Native GameObject。应调用：

```csharp
Destroy(enemy);
enemy = null;
```

`Destroy` 通常延迟到当前 Update 循环之后、渲染之前执行。运行时不要随意使用 `DestroyImmediate()`。

### 9.3 运行时 Texture、Mesh 和 Material

运行时创建的资源要显式销毁：

```csharp
var texture = new Texture2D(width, height);
// 使用完成后
Destroy(texture);
texture = null;
```

访问 `renderer.material` 可能创建材质实例；如果只需要共享材质，可评估 `sharedMaterial`，但修改共享材质会影响所有引用者。

---

## 10. Addressables、AssetBundle 与 Resources

### 10.1 Addressables

每一次 Load 都应有对应的 Release：

```csharp
AsyncOperationHandle<GameObject> handle =
    Addressables.LoadAssetAsync<GameObject>("Enemy");

await handle.Task;
GameObject prefab = handle.Result;

Addressables.Release(handle);
```

实例化对象应使用匹配的释放方式：

```csharp
Addressables.ReleaseInstance(instance);
```

常见泄漏原因：

- 丢失 `AsyncOperationHandle`；
- Load 多次但只 Release 一次；
- 场景切换时没有释放场景级句柄；
- 静态字段或 `DontDestroyOnLoad` 对象长期持有资产。

### 10.2 AssetBundle

```csharp
bundle.Unload(true);
```

`true` 会同时卸载 Bundle 和从中加载的对象。`Unload(false)` 只卸载 Bundle 自身数据结构，已加载对象仍留在内存，生命周期管理更复杂。

卸载时还要处理 Bundle 依赖关系，不能提前卸载仍被其他资产依赖的 Bundle。

### 10.3 Resources.UnloadUnusedAssets

该 API 会检查当前未被场景、组件或静态字段等引用的 Native 资产并尝试卸载。它不是每帧调用的通用 GC，因为全局可达性检查可能很重。

适合的调用时机：

- 大场景切换；
- 返回主菜单；
- 章节转换；
- 加载界面；
- 已经清除引用并销毁对象之后。

推荐生命周期顺序：

```text
停止使用对象
→ Destroy 实例
→ 清理静态字段、事件和缓存引用
→ Addressables.Release / AssetBundle.Unload
→ 在非关键时机执行 Resources.UnloadUnusedAssets
```

---

## 11. 大对象与对象池策略

### 11.1 预分配和复用

```csharp
private byte[] frameData;

void Awake()
{
    frameData = new byte[4 * 1024 * 1024];
}

void Update()
{
    Capture(frameData);
}
```

### 11.2 ArrayPool

```csharp
using System.Buffers;

byte[] buffer = ArrayPool<byte>.Shared.Rent(requiredSize);
try
{
    Process(buffer.AsSpan(0, requiredSize));
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer, clearArray: false);
}
```

注意：

- `Rent` 返回的数组可能大于请求长度；
- 不得在 `Return` 后继续使用；
- 敏感数据应考虑清理；
- 池可能保留大数组，增加常驻内存。

### 11.3 分块处理

对于大型文件、网络包、解码和序列化，应考虑：

- 流式读取；
- 固定大小数据块；
- 环形缓冲区；
- 分批编码或解码；
- 限制并发任务数量。

### 11.4 对象池必须有上限

对象池不是越大越好。极端峰值产生的对象如果永久留在池中，会长期占据托管和 Native 内存。

对象池应明确：

- 预热数量；
- 最大容量；
- 超量销毁策略；
- 场景切换清理策略；
- 回池时的状态重置逻辑。

回池时还应停止协程、取消异步操作、清理事件监听并重置动画、粒子和临时集合。

---

## 12. 手动 GC 与增量 GC

### 12.1 不要在游戏循环中随意 GC.Collect

通常不要在 `Update`、战斗高峰、镜头切换中途调用：

```csharp
GC.Collect();
```

有限的合理场景是：

1. 刚完成大量临时分配；
2. 已经释放所有临时引用；
3. 当前处于加载画面或黑屏；
4. 接下来进入严格的实时阶段；
5. 已通过目标设备 Profiler 验证收益。

### 12.2 不要把 Incremental GC 当成垃圾优化

Incremental GC 只是把较大的停顿摊到多帧，不会消除分配、扫描和清扫的总成本。根本优化仍是减少垃圾产生。

### 12.3 谨慎禁用 GC

禁用期间垃圾无法回收，托管堆只会增长。它只适合生命周期边界明确、内存预算严格且关键阶段几乎零分配的特殊场景，不是通用优化方案。

---

## 13. Profiler 排查流程

### 13.1 在目标设备上测试

编辑器自身会产生额外分配，不能完全代表 Player。应优先使用 Development Build 连接目标设备分析。

### 13.2 检查 GC.Alloc

在 CPU Profiler 中查看：

- 哪一帧发生分配；
- 每次分配多少字节；
- 哪个线程产生分配；
- 分配调用栈来自哪里。

重点检查 Update、UI、日志、网络、JSON、物理查询、对象生成和寻路代码。

### 13.3 对照 GC 时间线

观察 GC 事件是否和卡顿帧重合，并检查卡顿前是否长期存在小额每帧分配。触发 GC 的帧不一定是根因，根因可能是之前持续积累的垃圾。

### 13.4 使用 Memory Profiler 快照

建议在以下时间点捕获并比较快照：

1. 进入场景前；
2. 进入场景后；
3. 离开场景后；
4. 重复进入和离开多次后。

重点观察：

- Managed Objects；
- Native Objects；
- Texture、Mesh、Material；
- GC Reserved 和 GC Used；
- AssetBundle；
- NativeArray 等 Native 分配；
- 对象的引用保活链。

### 13.5 区分问题类型

| 问题 | 常见表现 | 主要处理方式 |
|---|---|---|
| 托管垃圾过多 | GC.Alloc 持续出现，GC 与卡顿对应 | 减少临时对象、复用集合、避免装箱和闭包 |
| Native 资源泄漏 | GC.Alloc 不高，但 Texture/Mesh/Native Memory 上涨 | Destroy、Dispose、Release、Unload，清理引用 |
| 瞬时大内存峰值 | 加载、解压或上传期间峰值巨大 | 分块加载、限制并发、错峰卸载旧资源 |

---

## 14. 实践检查清单

### 高频代码

- [ ] `GC.Alloc` 是否接近 0 B/frame？
- [ ] 是否每帧创建 List、Dictionary 或数组？
- [ ] 是否在热点中使用 LINQ？
- [ ] 是否每帧拼接字符串或更新 UI 文本？
- [ ] 是否反复创建闭包、委托或 `params` 数组？
- [ ] 是否存在值类型装箱？
- [ ] 是否使用了可复用容器或 NonAlloc API？

### 非托管资源

- [ ] `IDisposable` 对象是否使用 `using` 或明确 Dispose？
- [ ] NativeArray 是否按分配器生命周期释放？
- [ ] 是否可能在 Job 完成前释放 NativeArray？
- [ ] 运行时创建的 Texture、Mesh、Material 是否 Destroy？
- [ ] Addressables Load 与 Release 是否一一配对？
- [ ] AssetBundle 依赖和卸载顺序是否正确？

### 大对象与池

- [ ] 大数组和缓冲区是否可以复用？
- [ ] 是否可以分块、流式或限制并发？
- [ ] 对象池是否设置最大容量？
- [ ] 回池时是否清理事件、协程和异步任务？
- [ ] 是否在目标设备上测量峰值内存？

### GC 调度

- [ ] 是否误以为 Incremental GC 能消除垃圾成本？
- [ ] 是否在关键帧调用 `GC.Collect()`？
- [ ] `Resources.UnloadUnusedAssets()` 是否只放在明确边界？
- [ ] 所有优化是否通过 Profiler 验证？

---

## 15. 总结

GC 判断垃圾的本质是：

> 从 GC Roots 出发不可达的托管对象，才是垃圾。

标准 .NET 通过 Gen 0、Gen 1、Gen 2 和 LOH 优化不同生命周期、不同大小的对象；Unity 常用的 Boehm GC 则通常是非分代、非压缩的，Incremental GC 只是把工作拆分到多个帧。

Unity 内存优化的核心不是更频繁地强制回收，而是：

1. 减少热点路径中的托管分配；
2. 复用集合、数组、缓冲区和高频对象；
3. 显式释放非托管和 Unity Native 资源；
4. 正确配对 Destroy、Dispose、Release 和 Unload；
5. 避免频繁创建大型数组、字符串和对象图；
6. 使用 CPU Profiler 和 Memory Profiler 建立证据链。

## 参考资料

- [.NET Garbage Collection Fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)
- [.NET Large Object Heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap)
- [Unity Garbage Collection Best Practices](https://docs.unity3d.com/2022.3/Documentation/Manual/performance-garbage-collection-best-practices.html)
- [Unity Incremental Garbage Collection](https://docs.unity3d.com/2022.3/Documentation/Manual/performance-incremental-garbage-collection.html)
- [Unity Managed Memory](https://docs.unity3d.com/Manual/performance-managed-memory.html)
- [Unity Tracking GC Allocations](https://docs.unity3d.com/Manual/performance-track-garbage-collection.html)
- [Unity Memory Profiler](https://docs.unity3d.com/Packages/com.unity.memoryprofiler@1.1/manual/memory-profiler-introduction.html)
