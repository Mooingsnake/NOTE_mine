在 Unity Profiler 中区分 CPU 瓶颈和 GPU 瓶颈，关键在于**查看谁在等待谁（Wait Time）**。

---

### 如何在 Unity Profiler 中看区别

最直观的方法是观察 Profiler 的 **Timeline** 模块或 CPU Usage 中的**等待标记（Marker）**：

**1. CPU 瓶颈（CPU Bound）**
* **关键现象**：CPU 的 Main Thread（主线程）或 Render Thread（渲染线程）耗时极长，而 GPU 线程在等待。
* **核心 Marker**：
  * **`Gfx.WaitForCommands`**：出现在 Render Thread 上，表示渲染线程和 GPU 都空闲，**正趴着等待主线程提交指令**。
  * **`Camera.Render` / `RenderPipeline.Render`** 耗时高：说明 CPU 在收集、剔除对象以及打包 Draw Call 阶段花费了大量时间。

**2. GPU 瓶颈（GPU Bound）**
* **关键现象**：CPU 已经迅速把代码和渲染命令处理完了，但必须停下来等 GPU 把上一帧画面渲染完才能继续。
* **核心 Marker**：
  * **`Gfx.WaitForPresentOnGfxThread`** 或 **`Gfx.PresentFrame`**：出现在 Main Thread 上且耗时极高，说明 CPU 在**等待 GPU 完成渲染并呈现（Present）**。
  * 在 Profiler 顶部开启 **GPU Usage**，能直接看到 GPU Frame Time（如 30ms+）远高于 CPU 的实际工作时间。

> **排查注意**：关闭 **VSync** 和帧率限制（`Application.targetFrameRate = -1`），否则 `WaitForTargetFPS` 会干扰判断。

---

### 常见瓶颈原因及“为什么是 CPU 或 GPU”

性能瓶颈归根结底取决于**工作发生在硬件的哪一部分**：

| 瓶颈类型 | 常见原因 | 为什么导致的是 CPU 瓶颈（而非 GPU） |
| :--- | :--- | :--- |
| **CPU 瓶颈** | **Draw Call / Batch 数量过多** | **CPU 的职责是“打包命令”**。每一个 Draw Call，CPU 都需要设置渲染状态、切换材质并向 API 提交指令。对象太多时，CPU 会卡在命令打包上，而 GPU 收到命令后可能眨眼就画完了。 |
| **CPU 瓶颈** | **复杂的脚本逻辑 / GC 垃圾回收** | **通用计算由 CPU 执行**。如 `Update()` 中的密集循环、寻路算法、频繁使用 `Instantiate` 触发 GC 攒出的卡顿，全部运行在 CPU 核心上，与 GPU 无关。 |
| **CPU 瓶颈** | **CPU 蒙皮与物理模拟** | **碰撞与骨骼计算**。`UpdateManager`、`Physics.Simulate`、以及非 GPU 驱动的动画蒙皮（Skinned Mesh）都是 CPU 在逐顶点计算位置。 |
| **GPU 瓶颈** | **填充率不足 / Overdraw（重绘过多）** | **像素计算属于 GPU（Pixel/Fragment Shader）**。半透明特效或层层叠加的 UI 会导致同一个屏幕像素点被反复重绘（Overdraw），极大地消耗 GPU 的像素填充率（Fillrate）和显存带宽。 |
| **GPU 瓶颈** | **Shader 计算过于复杂** | **GPU 执行 Fragment/Vertex 代码**。使用了大量光照、PBR、复杂的数学运算或全屏后处理（Post-Processing，如 Bloom、SSAO），GPU 的算术逻辑单元（ALU）负载满载。 |
| **GPU 瓶颈** | **顶点/面片数量过高（高模）** | **GPU 几何阶段**。几何体顶点过多时，GPU 的 Vertex Shader 和 Rasterizer（光栅化器）处理不过来。*注意：若使用了 CPU Dynamic Batching，顶点多也会拉高 CPU*。 |

简单的判断逻辑是：**“算逻辑、发命令、算物理”** 归 CPU；**“算像素、算光影、画图形”** 归 GPU。
