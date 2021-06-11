---
title: "現時点で 2バージョン存在する AMD GPU の レイトレーシングHW IP"
date: 2021-06-11T15:55:01+09:00
draft: false
tags: [ "RDNA_2", "gfx1013", "GPUOpen" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD がオープンソースで開発し、公式的に提供している Vulkanドライバー、[AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK) の v-2021.Q2.5 がリリースされ、それを構成する [LLPC (LLVM-Based Pipeline Compiler)](https://github.com/GPUOpen-Drivers/llpc)、[PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal)、[XGL (Vulkan API Layer)](https://github.com/GPUOpen-Drivers/xgl) が共にアップデートされたが、  
その中で AMD GPU が搭載するレイトレーシングハードウェアの IPバージョンが 2種類存在することが新たに明かされた。  
レイトレーシングHW は AMD GPU の CU内に、*Ray Accelerator* という名で搭載されている。[^ra]  

[^ra]: [DirectX ® Ray tracing 1.1 - AMD_RDNA2_DirectX12_Ultimate_Raytracing1_1.pdf](https://gpuopen.com/wp-content/uploads/slides/AMD_RDNA2_DirectX12_Ultimate_Raytracing1_1.pdf)

 * [Release v-2021.Q2.5 · GPUOpen-Drivers/AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK/releases/tag/v-2021.Q2.5)
    * [Update pal from commit: a735a4dfb · GPUOpen-Drivers/pal@02ac99b](https://github.com/GPUOpen-Drivers/pal/commit/02ac99ba650afb3aebff3eb8006862ce93d31968)

## RT IP 1.0, 1.1 {#rt-ip-1_0-1_1}

今回 [PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal) のアップデートで追加された内容によれば、レイトレーシングHW の IP には現時点で、最初の実装となる `RtIp1_0` と、そこにトライアングルの重心座標 (`triangle barycentrics`) を処理する機能を追加した `RtIp1_1` が存在するとされる。  

 > 		/// Supported RTIP version enumeration
 > 		enum class RayTracingIpLevel : uint32
 > 		{
 > 		    _None = 0,
 > 		#ifndef None
 > 		    None = _None,      ///< The device does not have an RayTracing Ip Level
 > 		#endif
 > 		
 > 		    RtIp1_0 = 0x1,     ///< First Implementation of HW RT
 > 		    RtIp1_1 = 0x2,     ///< Added computation of triangle barycentrics into HW
 > 		};
 >
 > {{< quote >}} [pal/palDevice.h at 02ac99ba650afb3aebff3eb8006862ce93d31968 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/02ac99ba650afb3aebff3eb8006862ce93d31968/inc/core/palDevice.h#L754) {{< /quote >}}

そして、他の部分に追加されたコードでは、*RDNA 2/GFX10.3* とそれ以降の HWレイトレーシング IP が `RtIp1_1` だとしている。  (`IsGfx103Plus()` は処理的には *RDNA/GFX10.1* 以降の場合に true を返すようになっている[^gfx103p-func])  

[^gfx103p-func]: [pal/device.h at 7f6e731aa5193a518356bdfddc75571d032a3928 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/7f6e731aa5193a518356bdfddc75571d032a3928/src/core/device.h#L2555)

このことは ["RDNA 2" Instruction Set Architecture: Reference Guide](https://developer.amd.com/wp-content/resources/RDNA2_Shader_ISA_November2020.pdf) の 「8.2.10 Ray Tracing」セクションでも触れられており、そこでも レイトレーシングHW は重心座標の計算処理を直接サポートするよう設計されていると説明している。  

 > 		        if (IsGfx103Plus(pInfo->gfxLevel))
 > 		        {
 > 		            pInfo->gfx9.rayTracingIp = RayTracingIpLevel::RtIp1_1;
 > 		        }
 >
 > {{< quote >}} [pal/gfx9Device.cpp at 02ac99ba650afb3aebff3eb8006862ce93d31968 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/02ac99ba650afb3aebff3eb8006862ce93d31968/src/core/hw/gfxip/gfx9/gfx9Device.cpp#L5200) {{< /quote >}}

そして、*RDNA 2/GFX10.3* とそれ以降のレイトレーシングHW が `RtIp1_1` だとされていることからは、*RDNA 2/GFX10.3* 以前は `RtIp1_0` を搭載している可能性が示される。  
先日、LLVM に追加された [GPUID: gfx1013](/tags/gfx1013) は、*Navi10 (gfx1010)* をベースにしながら *RDNA 2/GFX10.3* でサポートされているレイトレーシング命令をサポートし、さらには APU という奇妙なものだったが、  
{{< link >}} [GPUID: gfx1013 は APU で、レイトレーシング命令をサポートしている? | Coelacanth's Dream](/posts/2021/06/06/gfx1013-apu-rt/) {{< /link >}}
今回 PAL に追加された内容と合わせて、 *gfx1013* で実装されているレイトレーシングHW *RDNA 2/GFX10.3* とは異なると考えられる。  
そもそも今回追加された部分は、*gfx1013* に関する LLVM の変更に合わせたか、AMD のソフトウェアエンジニアが *gfx1013* のサポートに向けて動き始めたことによるものなのかもしれない。  

レイトレーシングHW がトライアングルの重心座標を直接する計算する機能の有無がどれだけ影響するかは、正直自分には推し量れないし、IPバージョンの違いが分かってもまだ *gfx1013* の正体にはまだ遠いが、わずかでも明らかにされたこと、それを知ることができたことは嬉しい。  

{{< ref >}}
 * ["RDNA 2" Instruction Set Architecture: Reference Guide - RDNA2_Shader_ISA_November2020.pdf](https://developer.amd.com/wp-content/resources/RDNA2_Shader_ISA_November2020.pdf)
 * [DirectX ® Ray tracing 1.1 - AMD_RDNA2_DirectX12_Ultimate_Raytracing1_1.pdf](https://gpuopen.com/wp-content/uploads/slides/AMD_RDNA2_DirectX12_Ultimate_Raytracing1_1.pdf)
{{< /ref >}}
