---
title: "AMDVLK GPUOpenドライバーが Beige Goby/Navi24 に対応"
date: 2022-01-27T07:54:32+09:00
draft: false
tags: [ "GPUOpen", "RDNA_2", "Beige_Goby" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD が公式に提供しているオープンソース Vulkanドライバー、[AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK)、それを構成する各種ソフトウェア ([LLPC (LLVM-Based Pipeline Compiler)](https://github.com/GPUOpen-Drivers/llpc), [PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal), [XGL (Vulkan API Layer)](https://github.com/GPUOpen-Drivers/xgl)) が、*Beige Goby/Navi24 (gfx10)* に対応した。  

 * [Update pal from commit: f4c1b21c · GPUOpen-Drivers/pal@0a0a4ae](https://github.com/GPUOpen-Drivers/pal/commit/0a0a4ae4ab062d31fcd68863c35952967a8060ee)
 * [Update llpc from commit: 9c33c180 · GPUOpen-Drivers/llpc@7c2f5f7](https://github.com/GPUOpen-Drivers/llpc/commit/7c2f5f7e28f62f82524baa6234d967eeef9e77a9)
 * [Update xgl from commit: b30d9ce9 · GPUOpen-Drivers/xgl@d01032b](https://github.com/GPUOpen-Drivers/xgl/commit/d01032b9f8f7f3b7ce54a3078e9aae149386b716)

## Beige Goby/Navi24 spec {#beige_goby}
*Beige Goby/Navi24* の GPUコア部は、ShaderEngine (SE) 1基、SE あたりの ShaderArray (SA,SH) 2基、SA あたりの WGP (CU) 4基 (8基)、SE あたりの RenderBackend (RB) 4基という構成になっている。  
スペックに関するもう 1つのコードブロックでは SE あたりの RB数は 2基となっているが、AMD 公式サイトでは ROP 32基 (RB+ 4基) となっているため、下引用部が正しいと思われる。[^wrong-spec-?]  
メモリサイドキャッシュには、L2キャッシュブロック 1MiB (ブロック 4基)、L3キャッシュ/MALL 16MiB を持つ。メモリチャネルは 4ch、メモリタイプは GDDR6 に対応している (GDDR6 64-bit [16-bit x 4ch])。  
余談となるが、AMDGPU に関するソフトウェア中では L3キャッシュを MALL と呼んでいるものの、MALL が何の略称かは分かれており、LLVM では *Memory Attached Last Level* [^llvm-mall]、Linux Kernel における AMDGPUドライバーでは *Memory Access at Last Level* だとしている。[^amdgpu-mall]  

[^llvm-mall]: [llvm-project/AMDGPUUsage.rst at 6d28dffb6bf4c97848290b9aee3c19025470e54a · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/6d28dffb6bf4c97848290b9aee3c19025470e54a/llvm/docs/AMDGPUUsage.rst#memory-model-gfx10)
[^amdgpu-mall]: [linux/dc-glossary.rst at 8d0749b4f83bf4768ceae45ee6a79e6e7eddfc2a · torvalds/linux](https://github.com/torvalds/linux/blob/8d0749b4f83bf4768ceae45ee6a79e6e7eddfc2a/Documentation/gpu/amdgpu/display/dc-glossary.rst)
[^wrong-spec-?]: [pal/gfx9Device.cpp at 0a0a4ae4ab062d31fcd68863c35952967a8060ee · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/0a0a4ae4ab062d31fcd68863c35952967a8060ee/src/core/hw/gfxip/gfx9/gfx9Device.cpp#L5097-L5115)

 > 		    else if (AMDGPU_IS_NAVI24(m_nullIdLookup.familyId, m_nullIdLookup.eRevId))
 > 		    {
 > 		        pChipInfo->supportSpiPrefPriority  =     1;
 > 		        pChipInfo->doubleOffchipLdsBuffers =     1;
 > 		        pChipInfo->gbAddrConfig            = 0x242; // GB_ADDR_CONFIG_DEFAULT;
 > 		        pChipInfo->numShaderEngines        =     1; // GPU__GC__NUM_SE;
 > 		        pChipInfo->numShaderArrays         =     2; // GPU__GC__NUM_SA_PER_SE
 > 		        pChipInfo->maxNumRbPerSe           =     4; // GPU__GC__NUM_RB_PER_SE;
 > 		        pChipInfo->nativeWavefrontSize     =    32; // GPU__GC__SQ_WAVE_SIZE;
 > 		        pChipInfo->minWavefrontSize        =    32;
 > 		        pChipInfo->maxWavefrontSize        =    64;
 > 		        pChipInfo->numPhysicalVgprsPerSimd =  1024; // GPU__GC__NUM_GPRS;
 > 		        pChipInfo->maxNumCuPerSh           =     8; // GPU__GC__NUM_WGP_PER_SA * 2;
 > 		        pChipInfo->numTccBlocks            =     4; // GPU__GC__NUM_GL2C;
 > 		        pChipInfo->gsVgtTableDepth         =    32; // GPU__VGT__GS_TABLE_DEPTH;
 > 		        pChipInfo->gsPrimBufferDepth       =  1792; // GPU__GC__GSPRIM_BUFF_DEPTH;
 > 		        pChipInfo->maxGsWavesPerVgt        =    32; // GPU__GC__NUM_MAX_GS_THDS;
 > 		    }
 >
 > {{< quote >}} [pal/ndDevice.cpp at 0a0a4ae4ab062d31fcd68863c35952967a8060ee · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/0a0a4ae4ab062d31fcd68863c35952967a8060ee/src/core/os/nullDevice/ndDevice.cpp#L1050-L1067) {{< /quote >}}

*NGG (Next Generation Geometry)* に対応する RDNA系アーキテクチャ GPU では、特殊なオンチップバッファ、Parameter Cache (PC) を持つ。PC はメモリキャッシュと異なり、ピクセルシェーダ (Pixel Shader, PS) への入力バッファとして用いられる。  
{{< link >}} [RADVドライバーに NGGカリングが実装 | Coelacanth's Dream](/posts/2021/07/26/radv-nggc/#pc) {{< /link >}}
PC の規模について *RDNA 2 dGPU* という枠で見たとき、*Beige Goby* は他 GPU よりも少ない 512ラインとなっている。  
その他 *RDNA 2 dGPU*、*Sienna Cichlid/Navi21, Navy Flounder/Navi22, Dimgrey Cavefish/Navi23* それぞれで全体的な規模は異なるが、PC は同じ 1024ラインという構成を採っており、PC は GPUコア部の規模とは別に調節されているものと思われる。  
また、*RDNA 2 APU* である *VanGogh (Aerith), Yellow Carp (Rembrandt)* で PC は、*Beige Goby* よりも小さい 256ラインとなっている。  

 > 		      case CHIP_RAVEN:
 > 		      case CHIP_RAVEN2:
 > 		      case CHIP_RENOIR:
 > 		      case CHIP_NAVI10:
 > 		      case CHIP_NAVI12:
 > 		      case CHIP_SIENNA_CICHLID:
 > 		      case CHIP_NAVY_FLOUNDER:
 > 		      case CHIP_DIMGREY_CAVEFISH:
 > 		         pc_lines = 1024;
 > 		         break;
 > 		      case CHIP_NAVI14:
 > 		      case CHIP_BEIGE_GOBY:
 > 		         pc_lines = 512;
 > 		         break;
 > 		      case CHIP_VANGOGH:
 > 		      case CHIP_YELLOW_CARP:
 > 		         pc_lines = 256;
 > 		         break;
 >
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 29f264f25804eeea962f21c29c39050c3fc1663d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/29f264f25804eeea962f21c29c39050c3fc1663d/src/amd/common/ac_gpu_info.c#L1022-1039) {{< /quote >}}

| RDNA 2 | Sienna Cichlid<br>(Navi21) | Navy Flounder<br>(Navi22) | Dimgrey Cavefish<br>(Navi23) | Beige Goby<br>(Navi24) |
| :-- | :--: | :--: | :--: | :--: |
| Shader Engine | 4 | 2 | 2 | 1 |
| WGP (CU) | 40 (80) | 20 (40) | 16 (32) | 8 (16) |
| RB+ | 16 | 8 | 8 | 4 |
| Memory Bus | 256-bit | 192-bit | 128-bit | 64-bit |
| L2 Cache | 4 MiB | 3 MiB | 2 MiB | 1 MiB |
| L3 Cache | 128 MiB | 96 MiB | 32 MiB | 16 MiB |
| Transistor | 26.8 B | 17.2 B | 11.1 B | 5.4 B |
| Process | 7 nm | 7nm | 7 nm | 6 nm |
| Die size | 520 mm2 | 336 mm2 | 237 mm2 | 107 mm2 |

規模、ターゲット性能が近い *Navi14* と *Beige Goby/Navi24* を比較すると、SE 1基というのは共通するが、*Beige Goby* では 総 WGP (CU) 数は 8基減り、メモリバス幅と L2キャッシュも半減した。しかし L3キャッシュ 16 MiB が追加されている。  
RB数も減ったが、処理可能なピクセル数を増やした RB+ となったため、相当 ROP数は変わらない。  
それと *Navi14* ではハードウェアのバグで *NGG* が無効化されていたが、*Beige Goby* ではそれが無いため有効可能になっていることは触れておきたい。  

| | Navi14 |  Beige Goby<br>(Navi24) |
| :-- | :--: | :--: |
| Shader Engine | 1 | 1 |
| WGP (CU) | 12 (24) | 8 (16) |
| RB (ROP) | 8 (32) | 4 (32) |
| Memory Bus | 128-bit | 64-bit |
| L2 Cache | 2 MiB | 1 MiB |
| L3 Cache | N/A | 16 MiB |
| Transistor | 6.4 B | 5.4 B |
| Process | 7 nm | 6 nm |
| Die size | 158 mm2 | 107 mm2 |
| NGG | Disabled<br>(for HW bug) | Enabled |

{{< ref >}}
 * [AMD Radeon RX 6500 XT Graphics Card | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6500-xt#product-specs)
 * [2021 Hot Chips RDNA2 Pomianowski Final 20210820.pdf](https://hc33.hotchips.org/assets/program/conference/day2/2021%20Hot%20Chips%20RDNA2%20Pomianowski%20Final%2020210820.pdf)
{{< /ref >}}
