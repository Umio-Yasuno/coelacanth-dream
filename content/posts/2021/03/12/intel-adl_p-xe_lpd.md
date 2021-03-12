---
title: "Linux Kernel に Intel Alder Lake-P をサポートするパッチが投稿される　―― ディスプレイエンジンに \"XE_LPD\""
date: 2021-03-12T07:46:30+09:00
draft: false
tags: [ "Alder_Lake", "XE_LPD" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

*Alder Lake-P* GPU をサポートするパッチが Linux Kernel (intel-gfx) メーリングリストに投稿された。  
その前の *XE_LPD* 関連のパッチにて予告めいて語られていた、GPUコア部は *Gen12 アーキテクチャ* を採用し、ディスプレイエンジンには *Display13/XE_LPD* を採用するハードウェアプラットフォームとは *Alder Lake-P* のことだった。  
{{< link >}} [Intel、Display13 改め "XE_LPD" をサポートするパッチを投稿　―― 今後は独立して各 IP が更新されるように | Coelacanth's Dream](/posts/2021/03/12/intel-xe_lpd-display13/) {{< /link >}}

 * [[Intel-gfx] [PATCH 00/56] Introduce Alder Lake-P](https://lists.freedesktop.org/archives/intel-gfx/2021-March/262168.html)

サポートするメモリについては *Alder Lake-S* と同じとされる。[^adl_p-memory]  
*Alder Lake-S/P* では Coreboot へのパッチとそれへのコメントから、DDR4/LPDDR4(X)/DDR5/LPDDR5 に対応することが分かっている。  

 >       +-------------+--------+---------+
 >       | Memory Type | Max Ch | Max DPC |
 >       +-------------+--------+---------+
 >       | DDR4        |      1 |       2 |
 >       | LPDDR4(X)   |      4 |       1 |
 >       | LPDDR5      |      4 |       1 |
 >       | DDR5        |      2 |       1 |
 >       +-------------+--------+---------+
 >
 >    These numbers are for a single memory controller. Since ADL has two memory controllers, there's twice as many channels in total. DPC stays the same.
 >
 >
 > {{< quote>}} <https://review.coreboot.org/c/coreboot/+/45192/7/src/soc/intel/alderlake/meminit.c#109> {{< /quote >}}

[^adl_p-memory]: [[Intel-gfx] [PATCH 51/56] drm/i915/adl_p: Update memory bandwidth parameters](https://lists.freedesktop.org/archives/intel-gfx/2021-March/262184.html)

## Alder Lake-P GPU

*Alder Lake-P* は GPUコア、メディアエンジンは *Tiger Lake, Rocket Lake, Alder Lake-S, DG1* 同様に *Gen12* を採用するが、ディスプレイエンジンはそれらより1つ進んだ *Display13/XE_LPD* となる。  

GPUコア、メディアエンジンと世代がずれることになっても、ディスプレイエンジンに1世代進んだものを採用したことについては、  
*Display13/XE_LPD* には、パワードメインの関係性を従来のものから変更したことでパワーゲーティングの適用をシンプルにしたことや、Watermark {{< comple >}} ディスプレイの解像度等に応じて決定されるメモリのタイミング {{< /comple >}} の数を従来の 32個から 256個へと大幅に増やしたこと等、電力効率を向上される改良が施されており、  
モバイル向けである *Alder Lake-P* に *Display13/XE_LPD* を採用することの効果が大きいと判断したのではないかと考えられる。  
{{< link  >}} [Linux Kernel に Intel Alder Lake-P をサポートするパッチが投稿される　―― ディスプレイエンジンに "XE_LPD" | Coelacanth's Dream](/posts/2021/03/12/intel-adl_p-xe_lpd/) {{< /link >}}


