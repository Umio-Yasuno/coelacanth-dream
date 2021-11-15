---
title: "Alder Lake-S と同じ GPU IP となる Raptor Lake-S"
date: 2021-11-15T22:20:07+09:00
draft: false
tags: [ "Alder_Lake", "Raptor_Lake", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

intel-gfx メーリングリストに、*Raptor Lake-S (RPL-S)* のサポートを追加する最初のパッチが投稿された。  

 * [[Intel-gfx] [PATCH 0/3] Introduce Raptor Lake S](https://lists.freedesktop.org/archives/intel-gfx/2021-November/283099.html)
    * [[Intel-gfx] [PATCH 1/3] drm/i915/rpl-s: Add PCI IDS](https://lists.freedesktop.org/archives/intel-gfx/2021-November/283100.html)
    * [[Intel-gfx] [PATCH 2/3] drm/i915/rpl-s: Add PCH ID](https://lists.freedesktop.org/archives/intel-gfx/2021-November/283097.html)
    * [[Intel-gfx] [PATCH 3/3] drm/i915/rpl-s: Enable guc submission by default](https://lists.freedesktop.org/archives/intel-gfx/2021-November/283098.html)

*{{< xe class="lp" >}}/Gen12LP* から Intel GPU では Graphics (Render), Display, Media に分かれて IPバージョンが更新されるようになり、モバイル向けの *Alder Lake-P* は Graphics (Render) IP は Gen12 だが、Display IP は Gen13 ([XE_LPD](/tags/xe_lpd))、Media IP は Gen12.2 とされている。  
{{< link >}} [Intel、Display13 改め 「XE_LPD」 をサポートするパッチを投稿　―― 今後は独立して各 IP が更新されるように | Coelacanth's Dream](/posts/2021/03/12/intel-xe_lpd-display13/) {{< /link >}}
*DG2-G10/G11* では、Graphics (Render) IP に *{{< xe class="hpg" >}}/Gen12.55* 、Display IP に Gen13 ([XE_LPD](/tags/xe_lpd)) を採用している。  

今回のパッチのコメントでは、*Raptor Lake-S* は Graphics (Render), Display, Media ともに *Gen12* となり、ドライバー上では *Alder Lake-S* と同じ IPバージョンとなることが語られている。  
Display IP が、*Alder Lake-S* では Gen12、*Alder Lake-P* では Gen13 ([XE_LPD](/tags/xe_lpd))と分かれていたが、*Raptor Lake-S* では *Alder Lake-S* の GPU IP 構成をそのまま引き継ぎ、IP の更新はしないことになる。  

現時点で *Raptor Lake-S* の GPU部において *Alder Lake-S* と異なるのは、GuC (Graphics micro (µ) Controller) submission がデフォルトで有効化される点のみとなる。  

 > 		Raptor Lake S(RPL-S) is a version 12
 > 		Display, Media and Render. For all i915
 > 		purposes it is the same as Alder Lake S (ADL-S).
 > 		The series introduces it as a subplatform
 > 		of ADL-S. The one difference is the GuC
 > 		submission which is default on RPL-S but
 > 		was not the case with ADL-S.
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 0/3] Introduce Raptor Lake S](https://lists.freedesktop.org/archives/intel-gfx/2021-November/283099.html) {{< /quote >}}

GuC はドライバー部で担っている一部機能をオフロードする目的で開発されており、Gen9 から導入されている。  
Gen9 では GuC は、メディアエンコードのビットレート制御やヘッダのパースのオフロード可能な HuC (The HEVC /H.265 micro (µ) Controller) の認証を担っていた。  
*Alder Lake-P* の世代からは、GPUコンテキストのスケジューリング、クロックとパワーゲーティングの適用を選択する機能も実装され、またデフォルトで有効化されるようになった。  
*Alder Lake-S* では HuC の認証のみで、デフォルトでは無効化されている状態だったため、*Raptor Lake* では各 GPU IP こそ更新されないが、マイクロコントローラ部では更新が行われ、*Alder Lake-P* と同世代となる。  

| Intel GPU | Graphics IP ver | Display IP ver | Media IP ver |
| :-- | :--: | :--: | :--: |
| Tiger Lake<br>/Rocket Lake? | Gen12 | Gen12 | Gen12 |
| DG1 | Gen12.1 | Gen12 | Gen12 |
| Alder Lake-S<br>/Raptor Lake-S | Gen12 | Gen12 | Gen12.2[^gen12_2] |
| Alder Lake-P | Gen12 | Gen13<br>([XE_LPD](/tags/xe_lpd)) | Gen12.2[^gen12_2] |
| {{< xe class="hp" >}} | Gen12.5 | - | Gen12.5 |
| DG2-G10/G11 | Gen12.55 | Gen13<br>([XE_LPD](/tags/xe_lpd)) | Gen12.55 |

[^gen12_2]: [Add tests to detect and check MFX runtime. by uartie · Pull Request #306 · intel/vaapi-fits](https://github.com/intel/vaapi-fits/pull/306)

{{< ref >}}
 * [GuC](https://01.org/linuxgraphics/gfx-docs/drm/ch04s04.html)
 * [I915 GuC Submission/DRM Scheduler Section — The Linux Kernel documentation](https://www.kernel.org/doc/html/v5.15/gpu/rfc/i915_scheduler.html)
{{< /ref >}}
