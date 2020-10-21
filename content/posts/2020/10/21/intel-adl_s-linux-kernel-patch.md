---
title: "Linux Kenrel に Intel Alder Lake-S GPU をサポートするパッチが投稿される"
date: 2020-10-21T23:08:08+09:00
draft: false
tags: [ "Alder_Lake", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kenrel (intel-gfx) に、*Alder Lake-S* の GPU部をサポートする 18個のパッチが投稿された。  
{{< link >}} [[Intel-gfx] [PATCH 00/18] Introduce Alderlake-S](https://lists.freedesktop.org/archives/intel-gfx/2020-October/251057.html) {{< /link >}}

GPUアーキテクチャの世代はこれまでの情報通り [Gen12](/tags/gen12) となる。  
*Rocket Lake* は *Comet Lake PCH* と *Tiger Lake LP PCH* の両方に対応するように記述されているが、[^rkl]  
*Alder Lake-S* は *Alder Point PCH (600 Series)*[^adp-600] のみに対応する。[^adp]  

[^rkl]: [Intel、Rocket Lake 一連の Linux Kernelパッチを投稿 | Coelacanth's Dream](/posts/2020/05/01/intel-send-kernel-patch-rocketlake/)
[^adp-600]: [util/ifdtool: Include ADL dynamic check as per Gen12 SPI flash guide (I0faf0f0f) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/45808)
[^adp]: [[Intel-gfx] [PATCH 03/18] drm/i915/adl_s: Add PCH support](https://lists.freedesktop.org/archives/intel-gfx/2020-October/251060.html)

ディスプレイ出力において、*Alder Lake-S* はコンボPHYを 5基持ち、1 eDP、2 HDMI、2 DP の出力構成に対応する。[^adls-display]  

また、Coreboot での開発者のコメントから、*Alder Lake-S* はメモリコントローラーを 2基持ち、DDR4/LPDDR4(X)/LPDDR5/DDR5 に対応するとされる。[^adl-mem]  

[^adls-display]: [[Intel-gfx] [PATCH 08/18] drm/i915/adl_s: Setup display outputs and HTI support for ADL-S](https://lists.freedesktop.org/archives/intel-gfx/2020-October/251066.html)
[^adl-mem]: [Intel Alder Lake-S の GPU は Gen12アーキテクチャ | Coelacanth's Dream](/posts/2020/09/14/intel-adls-gen12-gpu/#adls-gt1_adlp-gt2)

PCI ID には `0x4680, 0x4681, 0x4682, 0x4683, 0x4690, 0x4691, 0x4692, 0x4693, 0x4698, 0x4699` の 10個が追加されているが、それぞれどのバリアントになるかは不明。  
*Alder Lake-S* の GPU は最大で GT1 (32EU) になるとされる。  

| Intel Gen12 | GT0.5 | GT1 | GT2 | GT2 (DG1) |
| :-- | :--: | :--: | :--: | :--: |
| Dual-Sub Slices | 1 | 2 | 6 | 6 |
| &ensp;EUs | 16 | 32 | 96 | 96 |
| &ensp;&ensp;SPs | 128 | 256 | 768 | 768 |
| GPU L3$ | 1536KB? | 1536KB? | 3072KB | 16384KB[^dg1-l3] |
| | RKL | TGL /RKL<br>/ADL-S | TGL /ADL-P? | DG1 |

[^dg1-l3]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)
