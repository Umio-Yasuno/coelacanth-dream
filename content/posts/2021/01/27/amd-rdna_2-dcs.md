---
title: "AMD RDNA 2 GPU は 「Duty Cycle Scaling」 をサポート"
date: 2021-01-27T13:55:36+09:00
draft: false
tags: [ "RDNA_2", "Navy_Flounder", "Dimgrey_Cavefish", "Sienna_Cichlid" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel (amd-gfx) に、*RDNA 2/GFX10.3* GPU でサポートしているとする `Gfx Duty Cycle Scaling (DCS)` を有効化するパッチが投稿された。  

 * [[PATCH] drm/amd/pm: Enable gfx DCS feature](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058852.html)
 * [[PATCH v2] drm/amd/pm: Enable gfx DCS feature](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058873.html)

## Duty Cycle Scaling(DCS) {#dcs}

パッチのコメントを読むに、DCS はより厳格な電力制限を行うための機能と取れる。  

DCS は *RDNA 2/GFX10.3* 世代の GPU に実装されているが、その有効化は消費電力制限が小さい SKU、[Sienna Cichlid/Navi21](/tags/sienna_cichlid) 以外の *RDNA 2* dGPU である [Navy Flounder/Navi22](/tags/navy_flounder) と [Dimgrey Cavefish/Navi23?](/tags/dimgrey_cavefish) に適用される。  

DCS の機能は、高負荷な処理中、電流/電力/温度が設定された制限を超える時に働き、一時的に GPUコアの電源を切ったり入れたりすることでそれ以上超えることを抑え、電流/電力/温度を枠内に再度収めようとする。  

一時的に GPUコアの電源を切る機能には他にもう 1つ、*Raven APU* からサポートされている `GFXOFF` があるが[^gfxoff]、`DCS` とは機能する状況が真逆で、`GFXOFF` は負荷がほとんど無いアイドル状態時、さらに消費電力を減らすための機能だが、`DCS` は負荷が高いビジー状態時、制限を超えさせないための機能となる。  

[^gfxoff]: [[PATCH 00/20] drm/amdgpu: gfx off support](https://lists.freedesktop.org/archives/amd-gfx/2018-April/021499.html)

`DCS` には 2種類あるとされ、`Async DCS` と `Frame-aligned.DCS` となる。  
この 2つは電力プロファイルのタイプによって使い分けられ、3D fullscreenモードと VRモード時には `Frame-aligned.DCS` が適用される。  
だが現時点では `Async DCS` のみをサポートするとして、3D fullscreenモードか VRモード時は無効化するようになっている。  
確かデフォルトでは 3D fullscreenモードとなるはずであり、`DCS` を有効化するには Kenelパラメータから `DCS` のフラグを立てた上で、プロファイルを別のものに切り替える必要がある。  

電力プロファイルにはその 2つ以外に、Power Savingモード、Videoモード、Computeモード、Customモードがある。  
それと、今回ソースコードを眺めていて気付いたが、*Sienna Cichlid* ではそれらに加えて、W3Dモードが追加されていた。  
詳細については追加したこと以上の言及が無く、どういったモードなのかや W3D の意味等は不明。  

 >        // Workload bits
 >        #define WORKLOAD_PPLIB_DEFAULT_BIT        0 
 >        #define WORKLOAD_PPLIB_FULL_SCREEN_3D_BIT 1 
 >        #define WORKLOAD_PPLIB_POWER_SAVING_BIT   2 
 >        #define WORKLOAD_PPLIB_VIDEO_BIT          3 
 >        #define WORKLOAD_PPLIB_VR_BIT             4 
 >        #define WORKLOAD_PPLIB_COMPUTE_BIT        5 
 >        #define WORKLOAD_PPLIB_CUSTOM_BIT         6 
 >        #define WORKLOAD_PPLIB_W3D_BIT            7 
 >        #define WORKLOAD_PPLIB_COUNT              8 
 >
 > {{< quote >}} <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/pm/inc/smu11_driver_if_sienna_cichlid.h?h=amd-staging-drm-next&id=ae5ac5eae6abfad3bba8ac74ad7510b8bb2836dd> {{< /quote >}}

`DCS` の使用モデルには CPU/APU + GPU の構成を採る高性能なノートPCを想定しているものと思われる。  
発熱、消費電力に厳しい制限のあるノートPCでは、処理に応じて CPU/APU、GPU に割り振る消費電力枠を最適化することで性能をより引き出すことができ、全体での電力管理が重要視される。  
AMD は **Ryzen 4000シリーズ (Renoir APU)** 発表時に Radeon dGPU と組み合わせることで使用可能な **AMD SmartShift** [^smartshift] を発表している。  
それに続いたのか、NVIDIA は CPU側のベンダーを問わずに使用可能な **NVIDIA Dynamic Boost** を[^dynamic-boost]、Intel も *Tiger Lake* と **Iris Xe MAX** の組み合わせで使用可能な **Dynamic Power Share** [^dynamic-power-share]を発表している。**NVIDIA Dynamic Boost 2.0** では VRAM も含めて電力管理を行うようになった。  

`DCS` はより厳格に GPU の制限を守ることで、そうしたノートPCにおける性能最適化を推し進める機能なのだと考えられる。  

[^smartshift]: [SmartShift Technology | AMD](https://www.amd.com/en/technologies/smartshift)
[^dynamic-boost]: [NVIDIA Details Dynamic Boost Tech & Advanced Optimus (G-Sync & Optimus At Last)](https://www.anandtech.com/show/15692/nvidia-details-dynamic-boost-tech-and-advanced-optimus)
[^dynamic-power-share]: [Innovation Extends with Intel Iris Xe MAX Graphics and Deep Link | Intel Newsroom](https://newsroom.intel.com/news/iris-xe-max-discrete-graphics-deep-link/)
