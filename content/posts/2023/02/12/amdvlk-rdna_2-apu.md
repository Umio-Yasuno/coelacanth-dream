---
title: "RDNA 2 APU に対応した AMDVLK ドライバー v-2023.Q1.1 が公開"
date: 2023-02-12T02:25:19+09:00
draft: false
categories: [ "Software", "AMD", "APU" ]
tags: [ "GPUOpen", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

リリースされてから既に 2週間経っているが、2023/01/31 付で Steam Deck に採用されている *VanGogh APU (Ariel)* を除いた *RDNA 2* APU に対応した AMDVLK v-2023.Q1.1 が公開された。  
具体的な対象 APU は、*Rembrandt (Yellow Carp, Ryzen 6000, Ryzen 7035)*、*Raphael (Ryzen 7000 Desktop)*、*Mendocino (Ryzen 7020)* となる。  
*VanGogh APU* が含まれていない理由としては、*VanGogh APU* は Valve と AMD が提携して開発されたカスタム APU であり、そして Valve は他の AMD GPU 向け Vulkan ドライバー、Mesa3D RADV に採用されているコンパイラバックエンド ACO の開発を行っている。  
またAMDVLK ドライバーがソースコードか `rpm` パッケージ、`deb` パッケージでのみ提供されている。  
そうしたことから AMDVLK ドライバーが *VanGogh APU* に対応する意味は薄いと考えられているのかもしれない。  

 * [Release v-2023.Q1.1 · GPUOpen-Drivers/AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK/releases/tag/v-2023.Q1.1)
    * [Update pal from commit 48f044c6 · GPUOpen-Drivers/pal@0423623](https://github.com/GPUOpen-Drivers/pal/commit/042362399cdac1019fbc7f0ace8489aee2907883)

## Raphael, Mendocino の GPU 規模 {#spec}
*Raphael, Mendocino* の GPU 規模は共通しており、ShaderEngine (SE) 1基、SE あたりの ShaderArray (SA/SH) 1基、SA あたりの WGP 1基 (CU 2基)、SE あたりの RB (RenderBackend) 1基と、*RDNA 2 アーキテクチャ* としては最低限に近い規模となっている。  
GPU L2キャッシュブロックは 2基、ブロックあたりのサイズは 128KiB、合計 256KiB となる。  
*VanGogh APU* や *Rembrandt APU* と比較すると 1/4 に近い規模だ。  

*Vega/GFX9 アーキテクチャ* で最小規模の GPU を搭載した *Raven2 (Dali/Pollock) APU* は CU 3基、GPU L2キャッシュ 128KiB。  
それと比較すると、GPU L2キャッシュは増え、CU 数は減ったとも見られるが、アーキテクチャの更新や高クロック化 (+900~1200MHz) によって性能は大きく向上していると思われる。  

 >         +    else if (AMDGPU_IS_MENDOCINO(familyId, eRevId))
 >         +    {
 >         +        pChipInfo->supportSpiPrefPriority      =     1;
 >         +        pChipInfo->doubleOffchipLdsBuffers     =     1;
 >         +        pChipInfo->gbAddrConfig                =  0x42; // GB_ADDR_CONFIG_DEFAULT; ///????
 >         +        pChipInfo->numShaderEngines            =     1; // GPU__GC__NUM_SE;
 >         +        pChipInfo->numShaderArrays             =     1; // GPU__GC__NUM_SA_PER_SE
 >         +        pChipInfo->maxNumRbPerSe               =     1; // GPU__GC__NUM_RB_PER_SE;
 >         +        pChipInfo->nativeWavefrontSize         =    32; // GPU__GC__SQ_WAVE_SIZE;
 >         +
 >         +        pChipInfo->minWavefrontSize            =    32;
 >         +        pChipInfo->maxWavefrontSize            =    64;
 >         +
 >         +        pChipInfo->numPhysicalVgprsPerSimd     =  1024; // GPU__GC__NUM_GPRS;
 >         +        pChipInfo->maxNumCuPerSh               =     2; // GPU__GC__NUM_WGP_PER_SA * 2;
 >         +        pChipInfo->numTccBlocks                =     2; // GPU__GC__NUM_GL2C;
 >         +        pChipInfo->gsVgtTableDepth             =    32; // GPU__VGT__GS_TABLE_DEPTH;
 >         +        pChipInfo->gsPrimBufferDepth           =  1792; // GPU__GC__GSPRIM_BUFF_DEPTH;
 >         +        pChipInfo->maxGsWavesPerVgt            =    32; // GPU__GC__NUM_MAX_GS_THDS;
 >         +    }
 >
 > {{< quote >}} [Update pal from commit 48f044c6 · GPUOpen-Drivers/pal@0423623](https://github.com/GPUOpen-Drivers/pal/commit/042362399cdac1019fbc7f0ace8489aee2907883) {{< /quote >}}

| RDNA 2 APU | VanGogh | Rembrandt | Raphael, Mendocino |
| :--        | :--:    | :--:      | :--:               |
| SE         | 1       | 1         | 1                  |
| SA per SE  | 1       | 2         | 1                  |
| WGP (2 CU) per SA | 4| 3         | 1                  |
| RB per SA  | 4?      | 4         | 1                  |
| GPU L2cache| 1024KiB?| 2048KiB   | 256KiB             |
