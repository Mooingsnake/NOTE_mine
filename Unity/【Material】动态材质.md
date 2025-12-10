# 材质
我们在制作材质的时候，会发现一个改动会同时应用到所有地方，如果遇到只想改色的时候会很被动，需要创建一大堆的材质文件。
# 方案1（材质实例化）
Unity 默认材质是 “共享实例”，修改属性会同步到所有引用处；但通过Instantiate复制材质为 “独立实例”，即可仅修改单个物体的材质属性，不影响其他物体。
## 代码
```
public class MaterialColorSetter : MonoBehaviour
{
    [Header("目标颜色")]
    public Color targetColor = Color.red;
    [Header("是否仅修改当前物体（不影响其他）")]
    public bool useInstance = true;

    private Renderer _renderer;

    void Start()
    {
        _renderer = GetComponent<Renderer>();
        if (_renderer == null) return;

        // 关键：复制材质为独立实例（断开共享）
        if (useInstance)
        {
            _renderer.material = new Material(_renderer.sharedMaterial); 
            // 等价写法：_renderer.material = Instantiate(_renderer.sharedMaterial);
        }

        // 修改颜色（仅当前实例生效）
        _renderer.material.SetColor("_Color", targetColor);
    }

    // 可选：编辑器下一键改色（无需运行）
    [ContextMenu("快速设置颜色")]
    void QuickSetColor()
    {
        if (_renderer == null) _renderer = GetComponent<Renderer>();
        if (useInstance) _renderer.material = new Material(_renderer.sharedMaterial);
        _renderer.material.SetColor("_Color", targetColor);
    }
}
```
<img width="569" height="222" alt="image" src="https://github.com/user-attachments/assets/03328eac-d952-4117-a641-56dd63db0859" />
## 注意
1. sharedMaterial：访问共享材质（修改会全局生效）；
2. material：访问 / 创建独立实例（修改仅当前物体生效）；
3. **缺点**：每个实例会占用额外内存，大量物体使用时需谨慎。

# 方案二 性能较优 —— 材质属性块（MaterialPropertyBlock）
Unity 提供MaterialPropertyBlock，可在不复制材质的前提下，为单个 Renderer 修改属性（底层共享材质，仅覆盖局部属性），内存占用极低，适合大量物体改色（如批量小怪、道具）。
```
using UnityEngine;

public class PropertyBlockColor : MonoBehaviour
{
    [Header("目标颜色")]
    public Color targetColor = Color.blue;
    private Renderer _renderer;
    private MaterialPropertyBlock _propBlock;

    void Start()
    {
        _renderer = GetComponent<Renderer>();
        _propBlock = new MaterialPropertyBlock();

        // 1. 获取当前Renderer的属性块
        _renderer.GetPropertyBlock(_propBlock);
        // 2. 修改颜色属性（_Color需与Shader中的属性名一致）
        _propBlock.SetColor("_Color", targetColor);
        // 3. 应用属性块到Renderer（仅当前物体生效）
        _renderer.SetPropertyBlock(_propBlock);
    }

    // 动态修改颜色（如交互时）
    public void ChangeColor(Color newColor)
    {
        _propBlock.SetColor("_Color", newColor);
        _renderer.SetPropertyBlock(_propBlock);
    }
}
```
## 性能注意
官网原文：
Note that this is not compatible with SRP Batcher. Using this in the Universal Render Pipeline (URP), High Definition Render Pipeline (HDRP) or a custom render pipeline based on the Scriptable Render Pipeline (SRP) will likely result in a drop in performance.
请注意，这与 SRP 批处理器不兼容。在通用渲染管线（URP）、高清渲染管线（HDRP）或基于可编程渲染管线（SRP）的自定义渲染管线中使用此功能，可能会导致性能下降。

### 为什么材质处理块（MaterialPropertyBlock）和SRP Batch不兼容？
**1. SRP 批处理是什么？**
SRP： Scriptable Render Pipeline 
SRP 批处理作用在「提交 Draw Call」这个核心环节，具体逻辑：
第一步（CPU 端）：渲染管线遍历所有可见物体，按「Shader + 材质参数 + 渲染状态」对物体分组；
第二步（关键优化）：对「Shader 相同、材质参数内存布局一致、渲染状态相同」的物体，将它们的绘制数据合并到同一个 GPU 缓冲区，最终只发送 1 次 Draw Call，让 GPU 批量渲染；
第三步（GPU 端）：GPU 一次性处理这批物体的顶点 / 像素计算，避免频繁切换绘制状态。

**2. MaterialPropertyBlock 为什么和 SRP 批处理不兼容？**
核心结论：MPB 会破坏 SRP 批处理依赖的 “材质参数一致性”，导致原本能合并的 Draw Call 被迫拆分—— 但这不是 “MPB 本身性能差”，而是 “使用 MPB 时若不注意，会失去 SRP 批处理的优化收益”。
1. SRP 批处理的核心前提：材质参数的 “内存布局一致”
SRP 批处理能合并 Draw Call 的关键条件是：
```
  多个物体必须使用同一个 Shader + 材质参数的 CBuffer 内存布局完全一致 + 渲染状态（如混合模式、深度测试）相同。
```
这里的「CBuffer」是 Shader 中存储材质参数的内存块（对应 Shader 里的CBUFFER_START(UnityPerMaterial)），URP/HDRP 会将每个材质的参数打包到 CBuffer 中，SRP 批处理通过对比 CBuffer 的 “哈希值” 来判断是否能合并：
如果两个物体的材质 CBuffer 哈希值相同 → 合并 Draw Call；
如果哈希值不同 → 拆分 Draw Call。

2. MPB 破坏批处理的底层逻辑
MaterialPropertyBlock 的本质是「为单个 Renderer 覆盖材质的部分参数」，这个操作会直接改变该物体的 CBuffer 哈希值，具体分两步：
步骤 1：MPB 的参数覆盖机制
MPB 不会修改原始材质的 CBuffer，而是在渲染该物体时，临时替换该 Renderer 对应的 CBuffer 中部分参数值（比如仅改_Color）。
步骤 2：CBuffer 哈希值差异化
即使两个物体用的是同一个基础材质、同一个 Shader，只要其中一个用了 MPB 修改了任意参数（比如颜色），它的 CBuffer 哈希值就会和未修改的物体不同 → SRP 批处理无法将它们归为一组，只能拆分 Draw Call。

3. 关键补充：不是 “完全不兼容”，而是 “滥用会失效”
官网的表述是「MPB 与 SRP 批处理不兼容」，更准确的解读是：
如果给一批原本能批处理的物体，每个都用 MPB 修改不同的参数（比如 100 个物体各改不同颜色）→ 每个物体的 CBuffer 哈希值都不同 → SRP 批处理完全失效，100 个物体产生 100 个 Draw Call；
如果给一批物体用 MPB 修改相同的参数（比如 100 个物体都改红色）→ 它们的 CBuffer 哈希值仍一致 → SRP 批处理依然生效，100 个物体仍合并为 1 个 Draw Call。

# 方案三 GPU Instancing（实例化）
**1. 核心原理**
在 Shader 中标记「实例化属性」（如_Color），让 GPU 在批量渲染时，为每个实例读取不同的参数；
CPU 只需提交 1 次 Draw Call，GPU 自动为每个实例应用不同参数，既保留批处理，又支持个性化。
GPU Instancing和SRP不兼容
**2. 实现示例（URP Shader）**
```
Shader "Custom/InstancedColor"
{
    Properties
    {
        _BaseColor ("Base Color", Color) = (1,1,1,1)
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" "EnableInstancing"="True" }
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing // 开启实例化
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct Attributes {
                float4 positionOS : POSITION;
                UNITY_VERTEX_INPUT_INSTANCE_ID // 实例ID
            };

            struct Varyings {
                float4 positionHCS : SV_POSITION;
                UNITY_VERTEX_OUTPUT_STEREO
            };

            // 标记为实例化属性（关键）
            UNITY_INSTANCING_BUFFER_START(Props)
                UNITY_DEFINE_INSTANCED_PROP(float4, _BaseColor)
            UNITY_INSTANCING_BUFFER_END(Props)

            Varyings vert(Attributes input)
            {
                Varyings output;
                UNITY_SETUP_INSTANCE_ID(input); // 绑定实例ID
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

                output.positionHCS = TransformObjectToHClip(input.positionOS.xyz);
                return output;
            }

            half4 frag(Varyings input) : SV_Target
            {
                // 读取当前实例的颜色
                float4 color = UNITY_ACCESS_INSTANCED_PROP(Props, _BaseColor);
                return color;
            }
            ENDHLSL
        }
    }
}
```
### 官网原文
The performance benefits of GPU instancing depend on the platform and the GPU. For each draw call, Unity has to collect, combine, and upload properties from various memory locations, so the performance overhead might outweigh the benefits. The performance benefits are better on mobile platforms than on desktop platforms.
GPU 实例化的性能优势取决于平台和 GPU。对于每个绘制调用，Unity 必须从不同的内存位置收集、合并并上传属性，因此性能开销可能会超过其带来的好处。移动平台上的性能优势要优于桌面平台。
