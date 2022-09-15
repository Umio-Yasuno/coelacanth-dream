---
title: "AMDVLK GPUOpen ドライバが Vulkan レイトレーシングに対応"
date: 2022-09-15T08:50:19+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GPUOpen", "RDNA_2" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

オープンソースで開発されている Vulkan ドライバ、AMDVLK GPUOpen が v-2022.Q3.4 リリースで *Navi2x/RDNA 2* における HWレイトレーシング機能に対応した。  
AMD GPU 向けの Vulkan ドライバには現在、Mesa3D RADV、AMDVLK GPUOpen、プロプライエタリな AMDGPU-PRO の 3種類があるが、今回ですべてが Vulkan レイトレーシングに対応したこととなる。  

 * [Release v-2022.Q3.4 · GPUOpen-Drivers/AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK/releases/tag/v-2022.Q3.4)

リリースに合わせて、レイトレーシングにおける BVH (Bounding volume hierarchy) のビルドやソート処理を HLSL シェーダーで実装した GPU Ray Tracing Library (GPURT) も公開されている。  
GPURT は Vulkan レイトレーシングの他に DXR (DirectX 12) にも対応している。  
そのため、GPURT は PAL (Platform Abstraction Library) に依存したライブラリとなるが、他の AMD GPU 向けのプロプライエタリなドライバでも GPURT を採用していることが考えられる。  

 * [GPUOpen-Drivers/gpurt: GPU Ray Tracing Library](https://github.com/GPUOpen-Drivers/gpurt)

Vulkan と DirectX の両方でレイトレーシングに対応すると同時に高速化を目的として、共通のライブラリ、シェーダー実装を用いる方策は Intel GPU でも採られている。  
Intel GPU では、OpenCLカーネルと独自フォーマットのメタカーネルによる GRL (Graphics Library for Ray-tracing) を Windows ドライバと Mesa3D Anvil (ANV) ドライバで採用している。  

{{< ref >}}
 * [レイトレーシング対応が進む Intel Vulkan ドライバー、Windows ドライバーと共有のライブラリを導入 | Coelacanth's Dream](/posts/2022/08/06/intel-anv-grl/)
{{< /ref >}}
