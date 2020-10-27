---
title: "Intel Alder Lake-S GPU には Rocket Lake 同様 GT0.5 バリアントが存在"
date: 2020-10-27T09:01:27+09:00
draft: false
tags: [ "Alder_Lake", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

先日 Linux Kernel (intel-gfx) に初期のサポートを追加するパッチが投稿された *Alder Lake-S* だが、  
{{< link >}} [Linux Kenrel に Intel Alder Lake-S GPU をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2020/10/21/intel-adl_s-linux-kernel-patch/) {{< /link >}}
ユーザーモードのオープンソース GPU ドライバー、Mesa3D にも PCI ID (DeviceID) を追加する初期的なパッチが投稿されている。  

そして、そのパッチ内で *Alder Lake-S* GPU に GT0.5 バリアントが存在することがわかった。  

 >       CHIPSET(0x4680, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4681, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4682, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4683, adl_gt05, "ADL-S GT0.5", "Intel(R) Graphics")
 >       CHIPSET(0x4690, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4691, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4692, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4693, adl_gt05, "ADL-S GT0.5", "Intel(R) Graphics")
 >       CHIPSET(0x4698, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >       CHIPSET(0x4699, adl_gt1, "ADL-S GT1", "Intel(R) Graphics")
 >
 > {{< quote >}} [intel/dev: Add device info for ADL-S (!7322) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7322/diffs) {{< /quote >}}

*Gen12アーキテクチャ* での GT0.5 バリアントは *Rocket Lake* にも存在し、実行ユニットである EU の規模は最大でも 16基という、*Gen12アーキテクチャ* では最小のバリアントとなる。  
*Alder Lake-S* には GT1 バリアントも存在し、こちらは最大 EU 32基となる。  

モバイル向けと見られる *Alder Lake-P* には最大で GT2 バリアントが存在するが、デスクトップ向けとされる *Alder Lake-S* は GT1 が最大となる。  
*Alder Lake-S* では DDR5メモリサポートによるメモリ帯域向上といった、GPU 性能の向上にも繋がる要素があるが、デスクトップ向けとして *Rocket Lake* から GPU の規模は増やされないと考えられる。  
2021年中にはエンスージアスト向けのディスクリート GPU **DG2 / {{< xe class="hpg" >}}** が投入されるため、統合 GPU を強化する必要がそうない、というのもあるだろう。  

*Rocket Lake* は 2021Q1 に市場へ投入される予定と発表されており[^rkl-2021q1]、*Alder Lake* は 2021年の後半 (2021Q2 - 2021Q3) に投入される予定にある。[^adl-2021q2-q3]  
ただ、これでもあまりにも間がないため、2021年後半に投入される *Alder Lake* はモバイル向け (*Alder Lake-P*) であり、デスクトップ向け (*Alder Lake-S*) はもう少し後になる、といった話もある。  

[^rkl-2021q1]: [Intel’s Commitment to Gaming, and a Sneak Peek at Intel Technology to Come | by Intel Author | Intel Tech | Oct, 2020 | Medium](https://medium.com/intel-tech/intels-commitment-to-gaming-and-a-sneak-peek-at-intel-technology-to-come-83677833be7f)
[^adl-2021q2-q3]: [Intel Reports Second-Quarter 2020 Financial Results :: Intel Corporation (INTC)](https://www.intc.com/news-events/press-releases/detail/1402/intel-reports-second-quarter-2020-financial-results)

| Intel Gen12 | GT0.5 | GT1 | GT2 | GT2 (DG1) |
| :-- | :--: | :--: | :--: | :--: |
| Dual-Sub Slices | 1 | 2 | 6 | 6 |
| &ensp;EUs | 16 | 32 | 96 | 96 |
| &ensp;&ensp;SPs | 128 | 256 | 768 | 768 |
| GPU L3$ Banks | 4 | 4 | 8 | 8 |
| GPU L3$ Size | 1920KB? | 1920KB | 3072KB | 16384KB[^dg1-l3] |
| | RKL / ADL-S | TGL /RKL<br>/ADL-S | TGL /ADL-P | DG1 |

[^dg1-l3]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)
