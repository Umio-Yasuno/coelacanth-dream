---
title: "AMDVLK GPUOpenドライバーが Navi22/Navy Flounder に対応"
date: 2021-03-20T21:36:33+09:00
draft: false
tags: [ "Navy_Flounder", "Navi22", "RDNA_2", "GPUOpen" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD が公式に提供しているオープンソース Vulkanドライバー、[AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK)、それを構成する [LLPC (LLVM-Based Pipeline Compiler)](https://github.com/GPUOpen-Drivers/llpc)、[PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal)、[XGL (Vulkan API Layer)](https://github.com/GPUOpen-Drivers/xgl) が、**Radeon RX 6700 XT** のベースとなる [Navi22](/tags/navi22)/[Navy Flounder](/tags/navy_flounder) に対応した。  


 * [2021-3-19 update · GPUOpen-Drivers/AMDVLK@f404771](https://github.com/GPUOpen-Drivers/AMDVLK/commit/f404771a8eb95455227c4f7f3827990b6d98f185)
    * [Update llpc from commit: 89e3cd653 · GPUOpen-Drivers/llpc@d69e5fb](https://github.com/GPUOpen-Drivers/llpc/commit/d69e5fb103b9c7394d4a098b23930ac5d5a99e6f)
    * [Update xgl from commit: 314ca488b4 · GPUOpen-Drivers/xgl@484d8e1](https://github.com/GPUOpen-Drivers/xgl/commit/484d8e1f46e0f4b3dcd16ca491253fbef41698a0)
    * [Update pal from commit: 928a2f9e0 · GPUOpen-Drivers/pal@4ea0bad](https://github.com/GPUOpen-Drivers/pal/commit/4ea0bad02244d155423be0a77d702c3a5a6e950f)

## Navi22/Navy Flounder 内部構成

*Navi22/Navy Flounder* の内部構成は以下から読み取れ、全体の SE (ShaderEngine) 数は 2基、SE あたりの SH (ShaderArray) は 2基、SH あたりの WGP (CU) 数は 5 (10) 基となっている。RB (RenderBackend) は SE あたり 4基。  
SE 1基分の規模は、**Radeon RX 6700 XT** よりも上位の **Radeon RX 6800/6900シリーズ** のベースとなる [Navi21](/tags/navi21)/[Sienna Cichlid](/tags/sienna_cichlid) と同じ。  
GPU 全体的には、SE数を *Navi21/Sienna Cichlid* からざっくり半分にしたのが *Navi22/Navy Flounder* となる。  
ただメモリバス幅は半分ではなく 3/4 の 192-bit で、L2キャッシュブロックもそれに合わせて 12基 (16-bit x 12ch分) 搭載している。  

 >                else if (AMDGPU_IS_NAVI22(pInfo->familyId, pInfo->eRevId))
 >                {
 >                    pInfo->gpuType                       = GpuType::Discrete;
 >                    pInfo->revision                      = AsicRevision::Navi22;
 >                    pInfo->gfxStepping                   = Abi::GfxIpSteppingNavi22;
 >                    pInfo->gfx9.numShaderEngines         = 2;
 >                    pInfo->gfx9.rbPlus                   = 1;
 >                    pInfo->gfx9.numSdpInterfaces         = 16;
 >                    pInfo->gfx9.maxNumCuPerSh            = 10;
 >                    pInfo->gfx9.maxNumRbPerSe            = 4;
 >                    pInfo->gfx9.numWavesPerSimd          = 16;
 >                    pInfo->gfx9.supportFp16Dot2          = 1;
 >                    pInfo->gfx9.gfx10.numGl2a            = 2;
 >                    pInfo->gfx9.gfx10.numGl2c            = 12;
 >                    pInfo->gfx9.gfx10.numWgpAboveSpi     = 5; // GPU__GC__NUM_WGP0_PER_SA
 >                    pInfo->gfx9.gfx10.numWgpBelowSpi     = 0; // GPU__GC__NUM_WGP1_PER_SA
 >                }
 >
 > {{< quote >}} [pal/gfx9Device.cpp at 4ea0bad02244d155423be0a77d702c3a5a6e950f · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/4ea0bad02244d155423be0a77d702c3a5a6e950f/src/core/hw/gfxip/gfx9/gfx9Device.cpp#L5008) {{< /quote >}}

{{< dia >}}
```
## Navi22/Navy Flounder Diagram

 +- ShaderEngine(00) -----------------+  +- ShaderEngine(01) -----------------+
 | +- ShaderArray(00) --------------+ |  | +- ShaderArray(00) --------------+ |
 | |  ==== ====  WGP(00) ==== ====  | |  | |  ==== ====  WGP(00) ==== ====  | |
 | |  ==== ====  WGP(01) ==== ====  | |  | |  ==== ====  WGP(01) ==== ====  | |
 | |  ==== ====  WGP(02) ==== ====  | |  | |  ==== ====  WGP(02) ==== ====  | |
 | |  ==== ====  WGP(03) ==== ====  | |  | |  ==== ====  WGP(03) ==== ====  | |
 | |  ==== ====  WGP(04) ==== ====  | |  | |  ==== ====  WGP(04) ==== ====  | |
 | |  [ RB+ ][ RB+ ]                | |  | |  [ RB+ ][ RB+ ]                | |
 | |        [- L1$ 128KB -]         | |  | |        [- L1$ 128KB -]         | |
 | |  [ Rasterizer/Primitive Unit ] | |  | |  [ Rasterizer/Primitive Unit ] | |
 | +--------------------------------+ |  | +--------------------------------+ |
 | +- ShaderArray(01) --------------+ |  | +- ShaderArray(01) --------------+ |
 | |  ==== ====  WGP(00) ==== ====  | |  | |  ==== ====  WGP(00) ==== ====  | |
 | |  ==== ====  WGP(01) ==== ====  | |  | |  ==== ====  WGP(01) ==== ====  | |
 | |  ==== ====  WGP(02) ==== ====  | |  | |  ==== ====  WGP(02) ==== ====  | |
 | |  ==== ====  WGP(03) ==== ====  | |  | |  ==== ====  WGP(03) ==== ====  | |
 | |  ==== ====  WGP(04) ==== ====  | |  | |  ==== ====  WGP(04) ==== ====  | |
 | |  [ RB+ ][ RB+ ]                | |  | |  [ RB+ ][ RB+ ]                | |
 | |        [- L1$ 128KB -]         | |  | |        [- L1$ 128KB -]         | |
 | |  [ Rasterizer/Primitive Unit ] | |  | |  [ Rasterizer/Primitive Unit ] | |
 | +--------------------------------+ |  | +--------------------------------+ |
 |      [- Geometry Processor -]      |  |      [- Geometry Processor -]      |
 +------------------------------------+  +------------------------------------+

    [L2$ 256K]    [L2$ 256K]    [L2$ 256K]    [L2$ 256K]
    [L2$ 256K]    [L2$ 256K]    [L2$ 256K]    [L2$ 256K]
    [L2$ 256K]    [L2$ 256K]    [L2$ 256K]    [L2$ 256K]
```
{{< /dia >}}

また、ワークアラウンド (一時的な回避策) の設定から、*Navi21/Sienna Cichlid* に存在した一部のハードウェア的な問題が修正されていると思われる。
(`waDisableFmaskNofetchOpOnFmaskCompressionDisable`, `waVgtFlushNggToLegacy`) [^wa]

[^wa]: [pal/gfx9SettingsLoader.cpp at 4ea0bad02244d155423be0a77d702c3a5a6e950f · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/4ea0bad02244d155423be0a77d702c3a5a6e950f/src/core/hw/gfxip/gfx9/gfx9SettingsLoader.cpp#L490)

