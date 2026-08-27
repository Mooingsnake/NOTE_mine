# Unity 中 UI 与粒子系统的遮挡关系及底层原理

---

## 1. 传统 3D 摄像机模式的遮挡困境

* **3D 渲染与 UI 渲染规则冲突**：
* **传统 3D 渲染**：依据 **Z 轴深度（Depth）** 判断遮挡关系，离摄像机近的物体遮挡离摄像机远的物体。
* **UGUI 渲染**：依据 **Hierarchy 节点树的顺序（Sibling Index）** 逐层叠加渲染（类似于 Photoshop 的图层叠放）。


* **失效场景**：
在需要实现“UI 底图 < 粒子特效 < 关闭按钮/文字”这类**夹心结构**时，3D 粒子系统由于只有统一的 3D 空间深度，无法同时做到“在某些 UI 之上”且“在另一些 UI 之下”。

---

## 2. 主流解决方案： ParticleEffectForUGUI (UIParticle)

开源插件 [mob-sakai/ParticleEffectForUGUI](https://github.com/mob-sakai/ParticleEffectForUGUI) 是解决 UI 与粒子系统混合渲染的最佳方案。

### 核心底层原理

* **API Mesh 烘焙 (Mesh Baking)**：
在每一帧，插件调用 Unity 原生接口 `ParticleSystem.BakeMesh`，抓取该瞬间粒子的所有顶点位置，将动态粒子“冻结”并导出为标准的 3D Mesh 数据。
* **接管 CanvasRenderer 绘制**：
烘焙出的 Mesh 不再经过 `ParticleSystemRenderer` 进行 3D 渲染，而是直接送入 UGUI 的 `CanvasRenderer` 组件。使粒子在物理属性上被转变为标准的 UI 图元。
* **基于 Sibling Index 排序与 Mask 兼容**：
* 粒子转化为 UI 节点后，其渲染顺序彻底受 Hierarchy 树中的节点上下顺序控制。
* 配合 UI 材质 Shader，通过 Stencil（模板测试）兼容 `Mask`，利用 `UnityGet2DClipping` 和 `_ClipRect` 逐像素裁剪兼容 `RectMask2D`。



---

## 3. 高级性能优化机制

### 零内存分配 (Zero Allocation / 0 GC)

由于粒子每帧位置都在变化，若逐帧 `new Mesh()` 会产生大量垃圾内存并触发 GC 卡顿。

* **实现思想**：
在对象初始化时提前申请预分配的 Mesh 缓存区对象（“空盒子”）。每帧刷新时调用 `_bakeMesh.Clear()` 清空数据，重新填充顶点，全程复用同一个 Mesh 对象。
* **核心代码示意**：
```csharp
public class UIParticleRenderer : MaskableGraphic
{
    private Mesh _bakeMesh; // 生命周期内仅实例化一次

    protected override void Awake()
    {
        base.Awake();
        if (_bakeMesh == null)
        {
            _bakeMesh = new Mesh();
            _bakeMesh.markDynamic(); // 标记为动态网格以提升 GPU 传输效率
        }
    }

    public void UpdateMesh(ParticleSystem ps)
    {
        if (ps == null) return;

        // 1. 清空数据而不销毁对象（0 GC 开销）
        _bakeMesh.Clear();

        // 2. 烘焙顶点直接写入已有缓存
        ps.BakeMesh(_bakeMesh, useTargetCamera: false);

        // 3. 提交给 UGUI Canvas 渲染
        canvasRenderer.SetMesh(_bakeMesh);
    }
}

```



### 网格共享 (Mesh Sharing)

对于界面上大量重复出现的粒子特效（如金币飞散、连续爆破）：

* **原理**：只由一个 **Primary（主）** 实例进行 `BakeMesh` 形状计算，其余 **Replica（副本）** 实例直接复用该 Mesh 数据。
* **为什么金币旋转时依然能共享？**：
Mesh 记录的是局部坐标系（Local Space）下的相对形状。金币的旋转与位移仅作用于各自节点的 `Transform` 矩阵（全局变换），GPU 可以在渲染时利用同一份 Mesh 结合不同的 Transform 直接绘制出不同角度与位置的金币。

---

## 4. 移动端与 PC 端显示不一致的排查指南

在 PC 端显示正常但在手机端显示不全或消失时，可按以下方面排查：

* **视口包围盒裁剪（Bake View Size / AABB）**：
移动端 GPU 的裁剪逻辑比 PC 更严格。若粒子的旋转或飞散范围超出了烘焙视口，会被手机 GPU 整体剔除（Cull）。
* *解决方式*：在 `UIParticle` 组件中勾选 **Use Custom View** 并调大 **Custom View Size**（如设为 `1000`）；或在 Project Settings 中调大 **Default View Size For Baking**。


* **Shader 通道丢弃（UV 数据限制）**：
移动端 UGUI 顶点数据通常只保留 `UV.xy` 通道（`zw` 会被丢弃）。若粒子使用了 Custom Vertex Streams 或贴图动画，手机端可能因数据丢失导致显示异常。
* *解决方式*：避免使用 Standard 3D 粒子 Shader，替换为 UI 专用 Shader（如 `UI/Additive` 或 `UI/Default`）。


* **移动端 65535 顶点限制**：
移动端 GPU 对单个网格的顶点数上限为 65535。若开启了 Trail 模块或发射数量过多会导致顶点溢出而被强制裁切。
* *解决方式*：适当降低 ParticleSystem 的 **Max Particles** 或 Trail 模块的 `Ribbon Count`。


* **坐标缩放模式（Position Mode）**：
不同屏幕分辨率下，`Position Mode` 设置为 `Relative` 可能会导致发射点偏移出屏外。
* *解决方式*：将 `Position Mode` 调整为 `Absolute` 并结合 `Auto Scaling Mode = UIParticle` 重新测试。
