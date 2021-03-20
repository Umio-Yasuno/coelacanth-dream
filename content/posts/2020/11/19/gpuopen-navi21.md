---
title: "AMDVLK GPUOpenドライバーが Navi21/Sienna Cichlid に対応"
date: 2020-11-19T15:30:58+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2", "GPUOpen" ]
# keywords: [ "", ]
categories: [ "AMD", "GPU", "Hardware" ]
noindex: false
# summary: ""
toc: false
---

AMD が公式に提供しているオープンソースな Vulkan ドライバー [AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK)、それを構成する [LLPC (LLVM-Based Pipeline Compiler)](https://github.com/GPUOpen-Drivers/llpc)、[PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal)、[XGL (Vulkan API Layer)](https://github.com/GPUOpen-Drivers/xgl) が、**Radeon RX 6800シリーズ** のベースとなる [Navi21/Sienna Cichlid](/tags/sienna_cichlid) に対応した。  

## Navi21/Sienna Cichlid の構成 {#sienna_cichlid}

AMDGPU ASIC の構成情報等が記述されている [pal/ndDevice.cpp](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp) に *Navi21/Sienna Cichlid* の情報が追加された。  

 >        else if (AMDGPU_IS_NAVI21(m_nullIdLookup.familyId, m_nullIdLookup.eRevId))
 >        {
 >            pChipInfo->supportSpiPrefPriority  =     1;
 >            pChipInfo->doubleOffchipLdsBuffers =     1;
 >            pChipInfo->gbAddrConfig            = 0x345; // GB_ADDR_CONFIG_DEFAULT;
 >            pChipInfo->numShaderEngines        =     4; // GPU__GC__NUM_SE;
 >            pChipInfo->numShaderArrays         =     2; // GPU__GC__NUM_SA_PER_SE
 >            pChipInfo->maxNumRbPerSe           =     4; // GPU__GC__NUM_RB_PER_SE;
 >            pChipInfo->nativeWavefrontSize     =    32; // GPU__GC__SQ_WAVE_SIZE;
 >            pChipInfo->minWavefrontSize        =    32;
 >            pChipInfo->maxWavefrontSize        =    64;
 >            pChipInfo->numPhysicalVgprsPerSimd =  1024; // GPU__GC__NUM_GPRS;
 >            pChipInfo->maxNumCuPerSh           =    10; // GPU__GC__NUM_WGP_PER_SA * 2;
 >            pChipInfo->numTccBlocks            =    20; // GPU__GC__NUM_GL2C;
 >            pChipInfo->gsVgtTableDepth         =    32; // GPU__VGT__GS_TABLE_DEPTH;
 >            pChipInfo->gsPrimBufferDepth       =  1792; // GPU__GC__GSPRIM_BUFF_DEPTH;
 >            pChipInfo->maxGsWavesPerVgt        =    32; // GPU__GC__NUM_MAX_GS_THDS;
 >        }
 >
 > {{< quote >}} [pal/ndDevice.cpp at b1e752d402592628f61eb7e1aa2a802a205de798 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/b1e752d402592628f61eb7e1aa2a802a205de798/src/core/os/nullDevice/ndDevice.cpp#L990) {{< /quote >}}
 >
 >            else if (AMDGPU_IS_NAVI21(pInfo->familyId, pInfo->eRevId))
 >            {
 >                pInfo->gpuType                       = GpuType::Discrete;
 >                pInfo->revision                      = AsicRevision::Navi21;
 >                pInfo->gfxStepping                   = Abi::GfxIpSteppingNavi21;
 >                pInfo->gfx9.numShaderEngines         = 4;
 >                pInfo->gfx9.rbPlus                   = 1;
 >                pInfo->gfx9.numSdpInterfaces         = 16;
 >                pInfo->gfx9.maxNumCuPerSh            = 10;
 >                pInfo->gfx9.maxNumRbPerSe            = 4;
 >                pInfo->gfx9.numWavesPerSimd          = 16;
 >                pInfo->gfx9.supportFp16Dot2          = 1;
 >                pInfo->gfx9.gfx10.numWgpAboveSpi     = 5; // GPU__GC__NUM_WGP0_PER_SA
 >                pInfo->gfx9.gfx10.numWgpBelowSpi     = 0; // GPU__GC__NUM_WGP1_PER_SA
 >            }
 >
 > {{< quote >}} [pal/gfx9Device.cpp at 6f6b37dd68ffe37868b14bfc1be7cf029eecc361 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/6f6b37dd68ffe37868b14bfc1be7cf029eecc361/src/core/hw/gfxip/gfx9/gfx9Device.cpp#L4893) {{< /quote >}}

ShaderEngine (SE) 4基、SE あたりの ShaderArray (SA) 2基、SA あたりの CU 10基、  
SE あたりの RB (Render Backend) 数は 4基となり、RB+ に対応しているため、相当 ROP数は 128基となる。 ( 4 [SE] * 4 [RB] * 8 [pixel] )  
Waveあたりのスレッド数はネイティブで Wave32 (32スレッド)、Wave64 (64スレッド) にも対応する。資料がまだ AMD より公開されていないため不明だが、Wave64 の場合は *RDNA / GFX10* 世代同様 Wave32x2 に分解して実行するのではないかと思う。  
SIMDユニットあたりで保持される Wave数は 16エントリとなり、これは事前に出てきていた情報と一致する。  
{{< link >}} [AMD Sienna Cichlid をサポートするパッチが OpenGL、Vulkanドライバーに投稿される | Coelacanth's Dream](/posts/2020/06/09/amd-sienna-cichlid-oss-umd/#wavefront-16) {{< /link >}}

ただ不自然なのは TCC/L2キャッシュブロック (`numTccBlocks`) が 20基とされている点で、*Navi21/Sienna Cichlid* は GDDR6 256-bit (16ch) であるから 16基な十分なはずだ。  
誤字、間違いである可能性もあるため、やはり AMD による資料公開が待たれる。  

<br>
また、MALL (Memory Access Last Level) の対応も追加されている。  
マーケティング上では **Infinity Cache** と呼ばれるそれは、コード中においては MALL となるようだ。  

**RadeonSI (OpenGL)** ドライバーでは既に *NGGカリング/プリミティブシェーダー* を、*RDNA 2 / GFX10.3* 世代の dGPU ではデフォルトに有効とするようになっているが、**AMDVLK (Vulkan)** ドライバーでも同様となる。[^amdvlk-ngg]  
{{< link >}} [RadeonSIドライバー + RDNA 2 では NGGカリング/プリミティブシェーダー がデフォルトで有効に | Coelacanth's Dream](/posts/2020/10/17/gfx103-default-ngg-culling/) {{< /link >}}

[^amdvlk-ngg]: [Update xgl from commit: b7048b5 · GPUOpen-Drivers/xgl@fc11e79](https://github.com/GPUOpen-Drivers/xgl/commit/fc11e79aab63337702d8efc05e5433dec9efdf06#diff-88f1e3e98bd72293bf65c23d0d2d080b18ad57d11e752604d0ab514ba1e12320)
