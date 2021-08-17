---
title: "Yellow Carp APU では DisplayPort 2.0 をサポートか"
date: 2021-08-17T22:11:12+09:00
draft: false
tags: [ "Yellow_Carp", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

Linux Kernel における AMD GPU ドライバーに向け、DisplayPort 2.0 SST (Single-Stream Transport)、そして UHBR10 (Ultra High Bit Rate, 10 Gbit/s) に対応するパッチが投稿された。  
DisplayPort 2.0 ではエンコード方式が、従来の 8b/10b (データ効率 80%) から 128b/130b (データ効率 96.71%) 変更されており、データ効率が向上している。  
レーンあたりのビットレートには UHBR10 (10 Gbit/s)、UHBR13.5 (13.5 Gbit/s)、UHBR20 (20 Gbit/s) の 3つがサポートされている。  

 * [[PATCH 0/6] Add DP 2.0 SST Support](https://lists.freedesktop.org/archives/amd-gfx/2021-August/067720.html)

今回の一連のパッチでは、ディスプレイエンジンIP **DCN 3.1** の内部ブロックのサポートを追加するものも含まれており、**DCN 3.1** にて DP 2.0 UHBR10 をサポートすると考えられる。  
そして **DCN 3.1** は、*Yellow Carp (Rembrandt) APU* で採用されることが以前のパッチで判明している。  
{{< link >}} [RDNA 2 APU 「Yellow Carp」 をサポートするパッチが Linux Kernel に投稿される　―― DCN3.1 / VanGogh より大きいキャッシュ | Coelacanth's Dream](/posts/2021/06/03/yellow_carp-apu-linux-kernel/#dcn3_1) {{< /link >}}
*RDNA 2 アーキテクチャ* を採用する dGPU {{< comple >}} Sienna Cichlid, Navy Flounder, Dimgrey Cavefish, Beige Goby {{< /comple >}} と *VanGogh APU* はディスプレイエンジンに **DCN 3.0x** を採用しており、既に製品として登場しているものは DP 1.4 with DSC までの対応となっている。[^rx-6900-xt]  
そのため、DP 2.0 UHBR10 をサポートしている可能性を持つ AMD GPU/APU は *Yellow Carp (Rembrandt) APU* からとなる。  

[^rx-6900-xt]: [AMD Radeon™ RX 6900 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6900-xt#product-specs)

*Yellow Carp (Rembrandt)* は *VanGogh* に続く *RDNA 2 APU* となり、人感センサーのサポートに関する他のパッチから次世代モバイルプラットフォームとして登場すると目される。  
{{< link >}} [AMD の次世代モバイルプラットフォームでは人感センサーをサポート | Coelacanth's Dream](/posts/2021/06/27/amd-hpd-next-gen-platform/) {{< /link >}}
**DCN 3.1** では新たなクロックステートが追加され、省電力面も従来から強化されている。  
また人感センサーについても、既に近い技術、機能を *Tiger Lake* の世代に投入している Intel の用途から、省電力への活用が期待される。  

アーキテクチャの進化以外にも、APU/SoC として各種機能にも改良が加えられている *Yellow Carp (Rembradt)* だが、一方でマルチメディアエンジンから AV1 を含む一部コーディックのサポートが削られる情報が存在する。  
{{< link >}} [他 RDNA 2 GPU とは対応コーディックが異なる Yellow Carp APU と Beige Goby GPU | Coelacanth's Dream](/posts/2021/07/14/yc-bg-vcn/) {{< /link >}}
さらには最近になって、*Barcelo* 、*Mendocino* という新たな APU のコードネームが OSS 上に登場したこともあり、機能と登場時期については読み取りにくいところがある。  
最も、AMD APU/GPU をサポートするパッチが投稿されてから実際に登場するまでは、ものによっては大きく期間が空くこともあり、予想したところで仕方がないし、興味の対象にもなりにくい。  

| AMD APU | CPU Arch | GPU Arch |
| :-- | :--: | :--: |
| Renoir | Zen 2 | Vega (gfx90c) |
| Lucienne | Zen 2 | Vega (gfx90c) |
| Cezanne | Zen 3 | Vega (gfx90c) |
| Barcelo | Zen 3 ? | Vega (gfx90c) |
| *Mendocino* | ? | ? |
| Yellow Carp (Rembrandt) | ? | RDNA 2 (gfx1035) |
| VenGogh | Zen 2 | RDNA 2 (gfx1033) |
| Cyan Skilfish | ? | RDNA (gfx1013) |

{{< ref >}}
 * [Chromium OS Docs - glossary.md](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/glossary.md)
{{< /ref >}}
