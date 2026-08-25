# UGUI 事件分发与 ScrollRect 性能优化

## 一、EventSystem 与 Raycaster 的事件分发机制

UGUI 的事件响应遵循以下底层链路：

> **Input Module 驱动 → 多 Raycaster 射线检测与排序 → 事件冒泡分发**

### 1. 触发与射线检测阶段

#### 1.1 Input Module 轮询

`StandaloneInputModule`（或 `InputSystemUIInputModule`）在 `Update` / `Process` 中捕获输入设备（鼠标、触摸、手柄）的状态。

#### 1.2 搜集 Raycaster

`EventSystem` 维护一个已注册的 `BaseRaycaster` 列表，其中包括：

- `GraphicRaycaster`
- `PhysicsRaycaster`
- `Physics2DRaycaster`

#### 1.3 多源射线检测

Input Module 调用各个 Raycaster 的 `Raycast()` 方法，搜集目标对象。

##### GraphicRaycaster

`GraphicRaycaster` 会遍历 Canvas 下所有满足以下条件的 `Graphic` 组件：

- `raycastTarget = true`
- 未被遮挡
- 点击点位于 Rect 范围内

同时，通过 `RectTransformUtility` 判定点击点是否落入 Rect 范围，并检测 Alpha 裁剪或 Mask。

##### PhysicsRaycaster

`PhysicsRaycaster` 利用 3D 物理系统发射射线，通过 `Physics.Raycast` 检测带有 Collider 的 3D 物体。

##### Physics2DRaycaster

`Physics2DRaycaster` 利用 2D 物理系统进行射线检测，检测带有 Collider 的 2D 物体。

---

### 2. 排序阶段：Raycast Result Sorting

搜集到所有命中的 `RaycastResult` 后，`EventSystem` 调用全局排序算法，决定哪个对象优先响应事件。

排序优先级如下：

#### 2.1 Sorting Criteria（排序类型）

优先比较 Canvas 的以下属性：

- Depth
- Sorting Layer
- Sorting Order

#### 2.2 Sort Order / Depth

- 对于 2D UI，先比较 Canvas 的 `sortingOrder`。
- 同一 Canvas 内比较 Hierarchy 渲染顺序：越靠后的节点渲染在前，优先级越高。
- 对于 3D 物体或 Canvas，比较距离摄像机的 Depth / Distance：越近优先级越高。

#### 2.3 Sorting Layer

比较 Sorting Layer，渲染层级靠前的对象优先。

#### 2.4 World Position / Plane Distance

在 3D 空间内比较对象与摄像机之间的绝对距离。

---

### 3. 事件冒泡与分发阶段：Event Bubbling

确定最上层的 Target GameObject 后，Input Module 利用 `ExecuteEvents` 类分发事件。

#### 3.1 PointerDown / PointerClick

从命中节点开始：

1. 检查当前节点是否实现对应事件接口，例如 `IPointerClickHandler`。
2. 如果当前节点未实现对应接口，事件将沿着 Hierarchy 向父节点层层冒泡传递。
3. 直到找到实现该接口的节点并执行，或到达根节点后终止。

#### 3.2 PointerDrag（拖拽事件）

当发生拖拽时：

1. 首先寻找最上层实现了 `IDragHandler` 的节点。
2. 如果当前节点未实现 `IDragHandler`，则沿父级向上寻找。
3. 这使得嵌套在 `ScrollRect` 内的按钮在滑动时，能够将 Drag 事件正确交给父级 `ScrollRect` 处理。

---

## 二、大量动态 Item 的 ScrollRect 性能优化

### 1. 原生 Layout Group 的性能痛点

`HorizontalLayoutGroup`、`VerticalLayoutGroup`、`GridLayoutGroup` 等 Layout Group 的核心痛点是：

只要子节点的尺寸、状态或数量发生变化，就可能触发 `SetLayoutDirty()`，进而引发整条层级链条的 Layout Rebuild 和 Canvas Mesh Rebuild。

除对象池外，可以采用以下方式避免频繁 Rebuild。

---

### 2. 废弃原生 Layout Group，采用“虚拟化复用列表”（Virtual List）

#### 原理

只生成视口（Viewport）内可见的少量 Item，例如 10 个。滑动时动态更新这 10 个 Item 的数据内容，并在 Item 离开视口时，将其移动到列表另一端循环利用。

#### 收益

- 无需挂载 Layout Group 组件，完全关闭自动布局计算。
- Item 的坐标更新由逻辑代码直接设置 `anchoredPosition`。
- 不再触发 UGUI 复杂的布局树计算。

#### 推荐开源方案

- [LoopScrollRect](https://github.com/qiankanglai/LoopScrollRect)
- [FancyScrollView](https://github.com/izumiya/FancyScrollView)

---

### 3. 自定义布局控制：绝对坐标赋值

如果必须自行实现，可以移除 `VerticalLayoutGroup` / `GridLayoutGroup`，用数学计算代替组件布局。

#### 实现方式

1. 预先计算所有 Item 的目标 `anchoredPosition`。
2. 例如，使用以下公式计算 Y 坐标：

   ```text
   Y = -index × itemHeight
   ```

3. 滑动时，仅对可见区域内的 Item 更新坐标。

这样可以彻底断开 Layout Rebuild 链条。

---

### 4. 隔离 Canvas：Canvas Isolation

#### 原理

将 `ScrollRect` 及其动态生成的 Item 整体放在一个独立的 Child Canvas 节点下。

#### 收益

虽然 ScrollRect 滑动时 Item 的移动不可避免地会触发其所在 Canvas 的 Mesh Rebuild，但 Canvas 隔离后，不会引发外部主 Canvas（例如 HUD、背景、导航栏）的重建，可以将 Rebuild 限制在局部范围内。

---

### 5. 优化 Item 内部的 Rebuild 触发源

#### 5.1 禁用非必要的 Raycast Target

Item 内部绝大多数 `Image` 和 `Text` 并不需要交互，可以关闭其 `raycastTarget`，从而显著降低 `GraphicRaycaster` 的遍历开销。

#### 5.2 避免使用 Content Size Fitter

Item 内部若包含动态文本，不要使用 `ContentSizeFitter` 让容器自适应大小。可以改用：

- 固定高度；或
- 由 C# 代码显式设置尺寸。

#### 5.3 使用 Alpha 控制代替 SetActive

动态隐藏或显示 Item 内部的微小元素时，可以使用 `CanvasGroup.alpha = 0 / 1` 代替 `GameObject.SetActive(false / true)`。

后者会强制触发 Hierarchy 变更并引发 Rebuild，而修改 CanvasGroup 的 Alpha 不会产生 Rebuild 开销。

#### 5.4 优化 Text 变化

对于频繁变动的数值，例如伤害数字、倒计时等，可以采用以下方式：

- 使用 TextMeshPro 的 `SetText(charArray)`，避免字符串拼接产生 GC。
- 使用 `CanvasRenderer.SetColor` 实现颜色渐变，避免触发顶点重写。

---

## 三、颜色修改、字符串拼接与 TextMeshPro 低 GC 更新原理

### 1. 为什么改变 Text 颜色（修改 `graphic.color`）会触发 Rebuild？

在 UGUI 中，`Image` 和 `Text` 都继承自 `Graphic` 类。UI 渲染本质上是在 CPU 侧拼装几何网格（Vertex Data），然后提交给 GPU 绘制。

#### 1.1 修改 `graphic.color` 的底层过程

当直接修改 `graphic.color = Color.red` 时，UGUI 底层会执行以下逻辑：

1. **触发标记**：调用 `Graphic.SetVerticesDirty()`，将当前 UI 对象标记为 Graphic Rebuild Dirty。
2. **加入重建队列**：`CanvasUpdateRegistry` 会将该对象放入待重建列表。
3. **网格重写（Rebuild）**：在当前帧渲染前的 `Canvas.SendWillRenderCanvases()` 阶段，UGUI 会重新调用 Text 或 Image 的 `OnPopulateMesh()` 方法。
4. **顶点重新填充**：根据新的颜色值，重新生成全部顶点的 Color 属性。UI 的每个字符由 4 个顶点组成的两个三角形构成，修改颜色意味着重新写入这 4 个顶点的 `Color32` 数据，同时重新计算 UV / Pos，并重新把整个大 Mesh 上传到 GPU 显存。

如果在一个拥有数百个 UI 元素的 Canvas 里，每帧都修改某几个 Text 的 `color`，就会反复引发 `OnPopulateMesh()` 计算，进而引发整个 Canvas 的 Mesh 重建，造成严重的 CPU 峰值。

### 2. 为什么 `CanvasRenderer.SetColor` 不触发 Rebuild？

`CanvasRenderer` 是 C# 层 Graphic 组件与 Unity C++ 原生渲染引擎之间的桥梁。

当调用 `CanvasRenderer.SetColor(color)` 时：

- 它不会修改 CPU 端的顶点网格数据。
- 它不会调用 `SetVerticesDirty()`。
- 它会直接将颜色值传递给底层的 C++ 渲染节点。
- 在 GPU 渲染绘制（DrawCall）前，通过 Shader 的 Per-Instance 或 Uniform Color 属性，与已有的顶点颜色进行乘法混合。

#### 结论

网格几何数据没有发生变化，因此不会产生 Rebuild 成本，适合用于以下 UI 动画：

- 颜色渐变
- 闪烁
- 淡入淡出

### 3. 为什么字符串拼接会导致 GC（Garbage Collection）？

这与 C# 中 `System.String` 的底层存储机制息息相关。

#### 3.1 `string` 是不可变（Immutable）的引用类型

在 C# 中，`string` 是存放在托管堆（Managed Heap）上的引用类型。最关键的特性是不可变性：一旦一个字符串对象被创建，它包含的字符序列就不能再被修改。

#### 3.2 字符串拼接在堆上的真实动作

下面是一段典型的 UI 更新代码：

```csharp
int gold = 100;

// 假设每帧或高频执行
myText.text = "Gold: " + gold.ToString();
```

当这行代码执行时，底层会进行一连串的堆内存分配：

1. **`gold.ToString()`**：将整型转为字符串，在托管堆上分配内存，生成一个新的 `string` 对象，例如 `"100"`。
2. **`"Gold: " + ...`**：C# 编译器将其转化为 `string.Concat("Gold: ", "100")`。
3. **分配新内存**：`string.Concat` 会在托管堆上申请一块全新的内存，大小足以容纳 `"Gold: 100"`，并把字符拷贝过去。
4. **旧对象失效**：上一步产生的临时字符串 `"100"` 在使用完毕后失去所有引用，变成垃圾对象。

即使写成以下形式：

```csharp
string s = "A" + "B" + "C";
```

也可能在堆上申请新的内存空间并丢弃旧空间。这些失去引用的临时字符串会持续占用托管堆，当堆内存达到临界值时，就会触发 GC（垃圾回收），导致游戏画面卡顿，即 GC Spike。

### 4. TextMeshPro 的 `SetText(charArray)` 如何消除 GC？

常规的 `Text.text = "123"` 必须传入一个 `string` 对象。如果写成：

```csharp
tmpText.text = myScore.ToString();
```

那么 `ToString()` 仍然会产生字符串堆分配。

TextMeshPro 提供了格式化重载，可以直接传入数值：

```csharp
// TMP 的高性能赋值重载（低 GC 方案）
tmpText.SetText("Score: {0}", myScore);
```

#### TMP 的低 GC 逻辑

1. **内部字符缓冲区（Internal Char Buffer）**：TMP 内部维护预先分配的字符缓冲区，避免频繁扩容。
2. **无需先转为字符串**：`SetText("Score: {0}", myScore)` 会利用内部格式化逻辑，将整数 `myScore` 的每一位数字直接填充到内部字符缓冲区中。
3. **避免字符串拼接**：整个过程可以避开显式的 `ToString()` 和字符串拼接，从而减少托管堆上的临时字符串分配。

#### 示例对比

```csharp
// 容易产生临时字符串分配
myText.text = "Score: " + myScore.ToString();

// 推荐：使用 TMP 的格式化 SetText
tmpText.SetText("Score: {0}", myScore);
```

---

## 四、UGUI Canvas Rebatching 与网格重建流程

### 1. 什么是 Canvas Rebatching？

**Rebatching（网格重批 / 合批）**是指 Canvas 收集其下 UI 元素的顶点、材质和纹理数据，重新构建并合并 Mesh，以生成尽量少的 DrawCall 的过程。

简单来说，UGUI 会尝试将满足合批条件的 UI 元素组织到同一批次中，从而减少 CPU 向 GPU 提交渲染命令的次数。

### 2. Rebatching 的完整执行流程

#### 2.1 标记 Dirty（变脏）

当某个 UI 元素发生属性变更，例如以下情况：

- 位置发生变化
- 大小发生变化
- 颜色发生变化
- 文本内容发生变化
- 层级或渲染状态发生变化

对应的 `CanvasRenderer` 会被标记为 Dirty，等待后续重建或重新组织批次。

#### 2.2 收集 Geometry

Canvas 会深度遍历所有未被禁用的 UI 元素，并收集各 UI 元素的以下数据：

- 顶点数据：`Position`、`UV`、`Color`
- `Material` 引用
- `Texture` 引用

#### 2.3 计算 Overlap 与 Depth

Canvas 按照 Hierarchy 顺序处理 UI 元素，并判断相邻或重叠的 UI 元素是否满足合批条件：

1. 判断 UI 元素的 Material 和 Texture 是否一致。
2. 判断 UI 元素之间的深度关系是否允许合批。
3. 如果材质和纹理一致，且深度没有被截断，则合并到同一个 Batch。
4. 如果材质或纹理不一致，或者发生遮挡，则打断合批并增加 DrawCall。

#### 2.4 生成 Mesh 缓存

完成 Geometry 收集和批次计算后，重新生成一块较大的 Mesh，并将其缓存下来。

#### 2.5 提交 GPU

实际渲染时，Canvas 直接使用缓存的大 Mesh 发起 DrawCall，将数据提交给 GPU 绘制。

### 3. Rebatching 与 Rebuild 的关系

在实际分析性能时，需要区分以下几个概念：

- **Layout Rebuild**：重新计算布局尺寸、位置和层级关系。
- **Graphic Rebuild**：重新生成 UI Graphic 的顶点数据、材质或纹理相关数据。
- **Canvas Rebatching**：重新组织 Canvas 下 UI 元素的批次和渲染顺序。
- **DrawCall 提交**：将最终组织好的渲染批次提交给 GPU。

这些过程彼此相关，但并不是完全相同的操作。某个属性变化可能只影响其中一个阶段，也可能沿着依赖链触发多个阶段。

---

## 五、UGUI 网格重建（Rebuild）的触发条件

UGUI 的 Rebuild 主要分为 **Layout Rebuild（布局重算）** 和 **Graphic Rebuild（网格与渲染重算）** 两个阶段。

### 1. 布局重算：Layout Rebuild

以下情况可能触发 Layout Rebuild：

#### 1.1 容器及布局组件变动

在带有以下组件的物体下添加或删除子节点，或者修改子节点尺寸：

- `HorizontalLayoutGroup`
- `VerticalLayoutGroup`
- `GridLayoutGroup`
- `ContentSizeFitter`

这些变更可能导致布局系统重新计算整个相关层级。

#### 1.2 RectTransform 驱动变更

修改控制布局的以下属性时，也可能触发布局重算：

- 锚点（Anchors）
- 轴心（Pivot）
- 尺寸（Size）
- `sizeDelta`

### 2. 网格重算：Graphic Rebuild

以下情况可能触发 Graphic Rebuild：

#### 2.1 位移与形变

修改以下属性时，可能触发 UI 网格或渲染数据更新：

- `RectTransform.anchoredPosition`
- `RectTransform.localScale`
- `RectTransform.localRotation`
- `RectTransform.sizeDelta`

这类变化常见于拖拽、滚动和动态移动 UI。

#### 2.2 外观与内容变更

以下变更通常会影响 Graphic 的网格或渲染状态：

- 修改 Image 的 Sprite
- 修改 Image 的 Type，例如从 `Sliced` 改为 `Simple`
- 修改 Image 的 `Fill Amount`
- 修改 Text / TextMeshPro 的文本字符串
- 修改字体大小
- 修改对齐方式
- 修改 UI 组件的 Color

其中，使用 `graphic.color` 修改颜色会触发 Mesh 重写；使用 `CanvasRenderer.SetColor` 或修改 `CanvasGroup.alpha` 则不会触发 Rebuild。

#### 2.3 层级与状态变更

以下操作会影响 UI 的层级、启用状态或渲染顺序：

- `GameObject.SetActive(true / false)`
- 修改 Hierarchy 结构
- `Transform.SetParent`
- `Transform.SetSiblingIndex`
- 启用或禁用 UI 组件，例如 `Image.enabled`

这些变更可能导致相关 Canvas 重新组织 Geometry、渲染顺序或批次。

### 3. 动态 UI 的隔离建议

频繁移动的动态 UI，例如血条、伤害数字等，务必考虑放入独立的 Child Canvas 中。

这样可以隔离 Rebuild 影响范围，避免局部 UI 的频繁变化导致主 UI Canvas 的整网格重构。

---

## 六、移动端 UI 纹理压缩：ASTC / ETC2 最佳实践

在移动端 UI 性能优化中，纹理压缩的核心目标是在**画质瑕疵（Artifacts）**与**内存 / 带宽消耗（bpp，Bits Per Pixel）**之间找到平衡。

- iOS 端已全面普及 ASTC 格式。
- Android 端以 ASTC 为主流，但为了兼容少量老旧设备，仍可能保留 ETC2 作为保底方案。

### 1. Block-based 压缩原理

ASTC 和 ETC2 都属于基于固定块（Block-based）的纹理压缩格式。纹理会被划分为固定大小的像素块，每个块使用有限的数据表达颜色和 Alpha 信息。

压缩率由 **Block Size（块大小）** 决定：

- 块越小，单位像素占用的 Bit 数（bpp）越高，画质越好，但内存占用越大。
- 块越大，单位像素占用的 Bit 数（bpp）越低，内存占用越小，但画质损失越明显。

### 2. 常见压缩格式与 Block Size 对照

| 压缩格式 | Block Size | bpp（Bits Per Pixel） | 相对内存占用（相比 RGBA32） | 画质特征与适用场景 |
| :--- | :--- | :--- | :--- | :--- |
| **ASTC 4x4** | $4 \times 4$ | 8.00 bpp | **25%** | **最高画质**：无明显瑕疵，适合极其精细的图标、全屏 CG、核心角色立绘。 |
| **ASTC 5x5** | $5 \times 5$ | 5.12 bpp | **16%** | **高品质推荐**：平衡性较好，适合通用 UI 图集、细微渐变和光晕图片。 |
| **ASTC 6x6** | $6 \times 6$ | 3.56 bpp | **11.1%** | **标准 UI 推荐**：画质良好，适合常规背景图、九宫格按钮和不含极细渐变的通用图集。 |
| **ASTC 8x8** | $8 \times 8$ | 2.00 bpp | **6.25%** | **高压缩率**：可能出现色块或模糊，适合大面积纯色、低细节背景、噪点贴图和暗纹。 |
| **ETC2 RGBA8** | $4 \times 4$ | 8.00 bpp | **25%** | **Android 保底**：带 Alpha 通道，Alpha 渐变区容易出现网格化噪点。 |
| **ETC2 RGB8** | $4 \times 4$ | 4.00 bpp | **12.5%** | **Android 保底**：不带 Alpha 通道，适合完全不透明的 UI 背景图。 |

### 3. Sprite Atlas 的尺寸、Padding 与 Alpha 限制

#### 3.1 尺寸对齐要求

由于压缩是以固定像素块为单位进行的，图集尺寸会受到 Block Size 的影响：

- ETC2 常见块尺寸为 $4 \times 4$。
- ASTC 常见块尺寸包括 $4 \times 4$ 到 $10 \times 10$。
- 图集长宽需要按照压缩块进行处理，通常要求为 Block Size 的整数倍。
- 为了兼容性，通常建议将图集尺寸设置为 2 的幂次方（POT）。

如果尺寸不满足要求，边缘可能会被填充补齐，或造成压缩异常。

#### 3.2 Alpha 透明过渡区的色块与脏边

当图集中包含 UI 光晕、阴影、微弱半透明按钮等 Alpha 渐变内容时，压缩算法需要在同一个压缩块中同时表达 RGB 颜色和 Alpha 梯度，容易出现以下问题：

- 失真
- 网格状噪点
- 半透明边缘脏边
- 透明区域渗色

#### 3.3 Padding 与 Bleeding

针对高质量、精细 Alpha 边缘 UI，打图集时需要适当增大 Padding，通常可以设置为 4 或 8，以减少纹理采样时的 Bleeding（拉伸渗色）。

也可以采用专门的 Alpha 分离通道策略，降低透明边缘压缩对颜色信息的影响。

#### 3.4 ETC2 Alpha 模式

ETC2 包含以下 Alpha 模式：

- **RGB + 1-bit Alpha（Punchthrough）**：只支持纯透或不透。
- **RGBA 8-bit**：支持渐变透明。

如果图集中包含渐变 UI，必须强制整张图集使用 `ETC2 RGBA8`，这会导致图集体积直接翻倍，即从 4 bpp 变成 8 bpp。

### 4. 内存与性能上的隐形限制

#### 4.1 CPU 读取导致的解压开销

GPU 可以直接读取显存中的 ASTC / ETC2 压缩数据。但如果 CPU 端需要读取像素，例如使用 `GetPixel` 校验点击区域，则必须勾选 `Read/Write Enabled`。

此时 RAM 中会保留一份未压缩的 RGBA32 副本，导致内存占用直接增加 4～8 倍。

#### 4.2 Alpha 通道的内存性价比衰减

ASTC 的 Bitrate（bpp）取决于 Block Size，例如：

- $4 \times 4 = 8\text{ bpp}$
- $6 \times 6 = 3.56\text{ bpp}$

如果为了追求高清晰度的 Alpha 曲线而使用更小的 Block Size，例如 ASTC 4x4，内存消耗会逼近非压缩格式，可能削弱纹理压缩的收益。

#### 4.3 通道分离带来的额外采样与批次成本

在老旧设备或极度追求透明度质量的场景中，有时会将图片拆分为 RGB 纹理与 Alpha 纹理两个 ETC1 / ETC2 压缩图。

这种方案可能带来以下成本：

- Shader 需要进行两次 Texture 采样。
- 纹理管理和材质配置更加复杂。
- 如果图集拆分不当，可能阻断 UGUI 的内置默认 Batch 逻辑。
- GPU 采样开销和 DrawCall 可能增加。

---

## 七、移动端 UI 纹理最佳实践配置策略

针对不同类型的 UI 资产，建议在 Unity 的 Texture Import Settings 中进行分类配置。

### 1. 核心精细 UI 与通用图集（S 级 / A 级）

**适用对象：**

- 主界面精细图标
- 带有细微 Alpha 渐变的 UI，例如按钮外发光、半透明阴影和渐变遮罩
- 高精角色立绘

**配置建议：**

- **iOS（ASTC）**：`ASTC 5x5`；画质极其敏感的立绘可以使用 `ASTC 4x4`。
- **Android（ASTC / ETC2）**：优先 `ASTC 5x5`；降级至 ETC2 时使用 `ETC2 RGBA8`。
- 如果 Alpha 渐变出现严重网格噪点，可以考虑拆分 Alpha 通道，或对部分核心图集放宽体积限制。

### 2. 常规 UI 图集与通用组件（B 级）

**适用对象：**

- 九宫格框
- 通用边框
- 常规按钮
- 不含复杂渐变的常规 UI 图集

**配置建议：**

- **iOS（ASTC）**：`ASTC 6x6`。
- **Android（ASTC / ETC2）**：优先 `ASTC 6x6`；保底使用 `ETC2 RGBA8`。

### 3. 大面积背景图与暗纹（C 级）

**适用对象：**

- 全屏底图
- 暗色遮罩
- 大面积不透明 UI 背景

**无 Alpha 背景：**

- **iOS**：`ASTC 8x8`
- **Android**：`ETC2 RGB8`，使用 4.00 bpp，不带 Alpha，节约一半内存。

**有 Alpha 背景：**

- **iOS**：`ASTC 8x8` 或 `ASTC 6x6`
- **Android**：`ASTC 8x8` / `ETC2 RGBA8`

### 4. 字体纹理与动态生成纹理

**适用对象：**

- TextMeshPro 字体图集（Font Atlas）
- 系统动态生成字体

**配置建议：**

- 严禁使用 ASTC / ETC2 有损压缩。
- 字体纹理边缘对 Alpha 极度敏感，压缩可能导致文字边缘模糊或破损。
- 必须使用单通道无损格式，例如 `Alpha 8` / `R8`，或使用无损 `RGBA32`。

---

## 八、移动端 UI 纹理落地踩坑防范清单

### 1. 尺寸对齐 Block Size

打 Sprite Atlas 时，图集长宽需要按照 Block Size 进行对齐。例如使用 ASTC 6x6 时，尺寸最好是 6 的倍数。

行业通用做法是直接设置为 2 的幂次方（POT），例如：

- `1024x1024`
- `2048x2048`

这样可以减少引擎补齐 Padding 导致的边缘失真。

### 2. 关闭不必要的 Mipmap

2D UI 纹理在正交相机下通常与屏幕像素 1:1 映射。勾选 `Generate Mipmaps` 会额外浪费约 33% 的内存，并可能导致 UI 在非整数倍缩放时变糊。

除非 UI 纹理确实需要 Mipmap，否则建议取消勾选。

### 3. 关闭 Read/Write Enabled

除非需要在 C# 代码中读取 UI 像素，例如自定义不规则按钮点击区域检测，否则不应勾选 `Read/Write Enabled`。

勾选后，内存中会额外保留一份 RGBA32 的 Uncompressed CPU 副本，使显存与内存占用增加。

### 4. 处理 Alpha 渐变黑边

ASTC / ETC2 压缩半透明边缘时，如果颜色通道在 Alpha 接近 0 的区域包含脏颜色，压缩后容易出现脏色黑边。

建议在打图集工具（例如 TexturePacker）中开启以下选项：

- `Bleeding`
- `Extrude`

通过相邻像素填充透明区域，降低纹理采样和压缩造成的边缘污染。
