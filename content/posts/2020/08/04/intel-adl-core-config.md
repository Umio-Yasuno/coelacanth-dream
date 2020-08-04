---
title: "Coreboot へのパッチから Intel Alder Lake-S/P のコア構成が判明か"
date: 2020-08-04T17:54:45+09:00
draft: false
tags: [ "Alder_Lake", ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU" ]
noindex: false
---

先の記事で、Coreboot へのパッチから *Alder Lake-S/P* の GPUバリアントが明らかになったと書いたが、  
{{< link >}} [Coreboot に Alder Lake のデバイスIDが追加　―― ADL-S の GPU は最大で GT1、ADL-P は GT2 か | Coelacanth's Dream](/posts/2020/08/04/adl-coreboot/) {{< /link >}}
また新たなパッチにて、今度は *Alder Lake-S/P* のハイブリットコア構成らしきものが判明した。[^adl-sku-list]  
*Alder Lake-P* もまたハイブリッドコア構成を取ると思われる。  

[^adl-sku-list]: <https://review.coreboot.org/c/coreboot/+/44163/1/src/soc/intel/alderlake/bootblock/report_platform.c>

以下が *Alder Lake-S/P* の MCH(Memory Controller Hub) のテーブル部分を抜き出したものであり、*Alder Lake-S* または *Alder Lake-P* の後に続く文字列は `( {Core} + {Atom} + {GPU} )` を表しているものと思われる。  
デスクトップ向けの *Alder Lake-S* は左の数字が大きい傾向に、モバイル向けまたは組み込み向けと思われる *Alder Lake-P* は中の数字が大きい傾向にあることからそう考えられる。  
[先の記事](/posts/2020/08/04/adl-coreboot/)で書いたように、GPU は *Alder Lake-S* では最大 **GT1** 、*Alder Lake-P* は最大 **GT2** となっており、右の数字はそれに沿っている。  



 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_1, "Alderlake-S (8+8+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_2, "Alderlake-S (8+6+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_3, "Alderlake-S (8+4+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_4, "Alderlake-S (8+2+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_5, "Alderlake-S (8+0+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_6, "Alderlake-S (6+8+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_7, "Alderlake-S (6+6+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_8, "Alderlake-S (6+4+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_9, "Alderlake-S (6+2+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_10, "Alderlake-S (6+0+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_11, "Alderlake-S (4+0+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_12, "Alderlake-S (2+0+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_13, "Alderlake-S (8+6+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_14, "Alderlake-S (8+6+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_S_ID_15, "Alderlake-S (8+8+1)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_1, "Alderlake-P (2+8+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_2, "Alderlake-P (2+4+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_3, "Alderlake-P (6+8+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_4, "Alderlake-P (6+4+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_5, "Alderlake-P (4+8+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_6, "Alderlake-P (2+4+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_7, "Alderlake-P (2+8+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_8, "Alderlake-P (2+0+2)" },
 >       { PCI_DEVICE_ID_INTEL_ADL_P_ID_9, "Alderlake-P (2+0+2)" },
 >
 > 引用元: <cite><https://review.coreboot.org/c/coreboot/+/44163/1/src/soc/intel/alderlake/bootblock/report_platform.c></cite>

<!--

| Alder Lake | Core | Atom | GPU |
| :-- | :--: | :--: | :--: |
| Alder Lake-S | 8 | 8 | 1 |
| | 8 | 6 | 1 |
| | 8 | 4 | 1 |
| | 8 | 2 | 1 |
| | 8 | 0 | 1 |
| | 6 | 8 | 1 |
| | 6 | 6 | 1 |
| | 6 | 4 | 1 |
| | 6 | 2 | 1 |
| | 6 | 0 | 1 |
| | 4 | 0 | 1 |
| | 2 | 0 | 1 |
| Alder Lake-P | 2 | 8 | 2 |
| | 2 | 4 | 2 |
| | 6 | 8 | 2 |
| | 6 | 4 | 2 |
| | 4 | 8 | 2 |
| | 2 | 4 | 2 |
| | 2 | 8 | 2 |
| | 2 | 0 | 2 |

-->

*Alder Lake-S/P* のハイブリッドコア構成らしきものを、少しでもわかりやすくしようとしたのが以下の表だが、なおも見にくい。  

<table>
<thead>
<tr>
<th align="left">Alder Lake</th>
<th align="center">Core?<br>(big)</th>
<th align="center">Atom?<br>(small)</th>
<th align="center">GPU</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left" rowspan="12">Alder Lake-S</td>
<td align="center" rowspan="5">8</td>
<td align="center">8</td>
<td align="center" rowspan="12">GT1</td>
</tr>

<tr>
<td align="center">6</td>

</tr>

<tr>
<td align="center">4</td>

</tr>

<tr>
<td align="center">2</td>

</tr>

<tr>
<td align="center">0</td>

</tr>

<tr>
<td align="center" rowspan="5">6</td>
<td align="center">8</td>

</tr>

<tr>
<td align="center">6</td>

</tr>

<tr>
<td align="center">4</td>

</tr>

<tr>
<td align="center">2</td>

</tr>

<tr>
<td align="center">0</td>

</tr>

<tr>
<td align="center">4</td>
<td align="center">0</td>

</tr>

<tr>
<td align="center">2</td>
<td align="center">0</td>

</tr>

<tr>
<td align="left" rowspan="6">Alder Lake-P</td>
<td align="center" rowspan="3">2</td>
<td align="center">8</td>
<td align="center" rowspan="6">GT2</td>
</tr>

<tr>
<td align="center">4</td>
</tr>

<tr>
<td align="center">0</td>
</tr>

<tr>
<td align="center">4</td>
<td align="center">8</td>
</tr>

<tr>
<td align="center" rowspan="2">6</td>
<td align="center">8</td>
</tr>

<tr>
<td align="center">4</td>
</tr>

</tbody>
</table>

ここから読み取れるのは、*Alder Lake-S/P* の GPU部はそれぞれ固定されており、GPU において下位のバリアントが存在しない。ただ、最大で **GT2** を持つ *Alder Lake-P* に **GT1** が存在しないのは少し不自然なように思う。最もコアの少ない `2(big) + 0(small)` のような CPU構成でも GPU はフル構成を取るだろうか？  
*Alder Lake* のコアは *Core (big)* 、*Atom (small)* 共に `2 /4 /6 /8` のどれかから選ばれる。  
*Alder Lake-S* の *Core (big)* が 2-Core または 4-Core である構成は *Atom (small)* 系コアを持たない。ローエンド/エントリー向けだろうか。  
反面、*Alder Lake-P* は *Core (big)* が 4-Core または 6-Core であっても、*Atom (small)* 系コアを最大数であろう 8-Core 持つ構成が存在する。*Tiger Lake-U/Y* は最大で 4-Core であるため、*Atom (small)* 系コアを抜きに *Core (big)* 系コアの数だけで見ても勝っている。  

クライアント向けの *Alder Lake* は 2021年の後半に市場へ投入されると、Intel は第2四半期の業績と同時に発表している。  
{{< link >}} [Intel Corporation - Intel Reports Second Quarter 2020 Financial Results](https://www.intc.com/investor-relations/investor-education-and-news/investor-news/press-release-details/2020/Intel-Reports-Second-Quarter-2020-Financial-Results/default.aspx) {{< /link >}}
何だか遠い話のようにも思えるが、*Alder Lake* サポートに向けた OSS へのパッチは既に投稿され始めている。  
*Alder Lake* の姿を思い描き、期待を内に膨らませたい人は以下のリンクを定期的にチェックすることをお勧めしたい。  
{{< link >}} [topic:"ADL_UPSTREAM" (status:open OR status:merged) · Gerrit Code Review](https://review.coreboot.org/q/topic:%22ADL_UPSTREAM%22+(status:open%20OR%20status:merged)) {{< /link >}}
{{< link >}} [The Intel-gfx Archives](https://lists.freedesktop.org/archives/intel-gfx/) {{< /link >}}
