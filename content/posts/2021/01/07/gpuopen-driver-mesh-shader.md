---
title: "GPUOpenドライバーアップデート ―― Task/Mesh Shader に対応"
date: 2021-01-07T13:29:58+09:00
draft: false
tags: [ "RDNA_2", "GPUOpen", "NGG" ]
# keywords: [ "", ]
categories: [ "AMD", "GPU", "Software" ]
noindex: false
# summary: ""
toc: false
---

[GPUOpenドライバー](https://github.com/GPUOpen-Drivers) とそれを構成する各種ソフトウェア (LLPC, XGL, PAL) がアップデートされた。  
AMDVLKドライバーでは、**RADV** ドライバーとの切り換えが可能となった。[^amdvlk]  
他には、[Sienna Cichlid/Navi21](/tags/sienna_cichlid) への最適化、各種問題の修正、ヘッダーのアップデート、そして Task/Mesh Shader の対応が行なわれている。  

[^amdvlk]: [Update README.md: Usage of AMD switchable graphics layer · GPUOpen-Drivers/AMDVLK@52a9f57](https://github.com/GPUOpen-Drivers/AMDVLK/commit/52a9f5750fee299162db35f6ff71bbc8a4f3b003)

 * [Update llpc from commit: 2e5cf60e5 · GPUOpen-Drivers/llpc@44a22a1](https://github.com/GPUOpen-Drivers/llpc/commit/44a22a1e3af35c3209c149c871897fac2b3d6e17)
 * [Update pal from commit: 5cece43a1 · GPUOpen-Drivers/pal@26cb05f](https://github.com/GPUOpen-Drivers/pal/commit/26cb05f899cc587f9398399a3381ee22ab41f4c2)
 * [Update xgl from commit: 32c048be33 · GPUOpen-Drivers/xgl@86f61a3](https://github.com/GPUOpen-Drivers/xgl/commit/86f61a31988a626371131a1633547b0d0ebfcfcb)

以前、[ACOバックエンド](/tags/aco) を開発する、Valve の Timur Kristóf 氏が、XDC2020 の発表にて、Mesh Shader は *NGG* で実行できる *可能性* がある、と述べていた。  

*NGG/Primitve Shader* はハードウェア側に新設されたシェーダーステージであり、それまでのバーテックスシェーダー (Vertex Shader, VS)、ジオメトリシェーダー (Geometry Shader, GS) が統合されており、それらを置き換えるものとなる。  
{{< link >}} [ACOバックエンドでも NGG をサポートする動き | Coelacanth's Dream](/posts/2020/10/04/aco-ngg-gfx10/) {{< /link >}}
ハードウェア側のシェーダーステージは *NGG* が有効な場合、LSHS (Local Shader + Hull Shader) 、*NGG* 、Pixel Shader の 3段と短くなる。  
また、*NGG* では早期カリング機能として、 *NGG Culling* が存在する。  

そう多くを読み取れてはいないが、今回のアップデートで追加された部分から、Mesh Shader をハードウェア側では ジオメトリシェーダー部で処理しているものと思われ、  
上述したように、*NGG* には ジオメトリシェーダー部も統合されている。  

 >       #if (PAL_CLIENT_INTERFACE_MAJOR_VERSION >= 574)
 >               if (HasMeshShader())
 >               {
 >                   // HasMeshShader(): PipelineMesh
 >                   // API Shader -> Hardware Stage
 >                   // PS -> PS
 >                   // MS -> GS
 >       
 >                   CalcDynamicStageInfo(graphicsInfo.ms, &pStageInfos->gs);
 >               }
 >               else
 >       #endif
 >
 > {{< quote >}} [pal/gfx9GraphicsPipeline.cpp at 26cb05f899cc587f9398399a3381ee22ab41f4c2 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/26cb05f899cc587f9398399a3381ee22ab41f4c2/src/core/hw/gfxip/gfx9/gfx9GraphicsPipeline.cpp#L608-L619) {{< /quote >}}

ただ、判定に *NGG* の存在や GPU の世代が使われていないため、*RDNA 2/GFX10.3* 以外、*Vega/GFX9* 、*RDNA/GFX10* でも Mesh Shader をサポートする可能性が浮かぶが、実際の所は不明。  
Mesa3D (RadeonSI, RADV) ドライバーで Task/Mesh Shader の本格的な実装が始まれば、断片として、また少し詳細が明らかにされるかもしれない。  

 >        // Mesh shaders are not supported on Gfx6/7/8.
 >        PAL_ASSERT((metadata.pipeline.hasEntry.meshScratchMemorySize == 0) ||
 >                   (metadata.pipeline.meshScratchMemorySize == 0));
 >
 > {{< quote >}} [pal/gfx6GraphicsPipeline.cpp at 1e61f9ab6eb2041c73d7e7bd8f2021fa05a4bdd8 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/1e61f9ab6eb2041c73d7e7bd8f2021fa05a4bdd8/src/core/hw/gfxip/gfx6/gfx6GraphicsPipeline.cpp) {{< /quote >}}
 >
 >        // Register address for passing the 32-bit GPU virtual address of a buffer storing the shader-emulated task+mesh
 >        // pipeline stats query.
 >        uint16  taskPipeStatsBufRegAddr;
 >
 > {{< quote >}} [pal/gfx9Chip.h at 1e61f9ab6eb2041c73d7e7bd8f2021fa05a4bdd8 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/1e61f9ab6eb2041c73d7e7bd8f2021fa05a4bdd8/src/core/hw/gfxip/gfx9/gfx9Chip.h) {{< /quote >}}
 >
 >         MeshPipeStatsBuf  = 0x10000014, ///< 32-bit GPU virtual address of a buffer storing the shader-emulated mesh
 >                                        ///  pipeline stats query.
 >
 > {{< quote >}} [pal/gfx9Chip.h at 1e61f9ab6eb2041c73d7e7bd8f2021fa05a4bdd8 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/1e61f9ab6eb2041c73d7e7bd8f2021fa05a4bdd8/src/core/hw/gfxip/gfx9/gfx9Chip.h) {{< /quote >}}


{{< ref >}}

 * [Using Mesh Shaders for Professional Graphics | NVIDIA Developer Blog](https://developer.nvidia.com/blog/using-mesh-shaders-for-professional-graphics/)
 * [PowerPoint Presentation - AMD_RDNA2_DirectX12_Ultimate_SamplerFeedbackMeshShaders.pdf](https://gpuopen.com/wp-content/uploads/slides/AMD_RDNA2_DirectX12_Ultimate_SamplerFeedbackMeshShaders.pdf)

{{< /ref >}}
