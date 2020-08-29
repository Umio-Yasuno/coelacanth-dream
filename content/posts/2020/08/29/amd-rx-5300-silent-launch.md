---
title: "AMD、Navi14ベース、メモリ 3GB の RX 5300 を製品に追加"
date: 2020-08-29T21:43:53+09:00
draft: false
tags: [ "Navi14", "gfx1011" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD は TSMC 7nm プロセスで製造される [Navi14](/tags/navi14) をベースとした、**Radeon RX 5300** を製品モデルにこっそり追加した。  
{{< link >}} [AMD Radeon™ RX 5300 Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5300#product-specs) {{< /link >}}

有効 WGP(CU)数は 11(22)基、Game Clock は 1448MHz、Boost Clock は 1645MHz と、コア部は既存のモバイル向けである **RX 5500M** と一致するが、  
メモリは 1チップ減らされ、メモリバス幅 96-bit、メモリ帯域は 168GB/s、メモリ容量 3GB となっている。  
TDP も **RX 5300** は 100W であり、補助電源 {{< comple >}} 恐らく 1x 6-pin {{< /comple >}} を必要とする。  

競合との比較では NVIDIA GTX 1650 の OCモデルをあげており、**RX 5300** はそれよりも高速としている。  
だが、NVIDIA GTX 1650 には GDDR5 と GDDR6 で異なるメモリ仕様が存在し、比較対象がどちらかは明らかでない。  

価格もまた不明だが、現時点の日本 Amazon では **RX 5500 XT 4GB** が安い方で &yen;20,000 前後であるため、それより少し低い程度になるのではないかと思う。個人的な予想価格は販売当初で &yen;18,500 〜 &yen;19,000 。  

## 補助電源無し Radeon は更新されないのか
補助電源を必要としない TDP 75W 以下の Radeon は 14nm で製造される *Polaris11、Polaris12* ベースの製品から更新されていない。具体的な製品には、**RX 560** 、**RX 550** があるが、それらが追加されたのは 2017年であり、もう 3年近く更新されていないことになる。  
そして、**RX 560** は [価格.com](https://kakaku.com/pc/videocard/) で確認した限りでは、現在どこも取り扱っておらず、最も新しい世代で、かつ補助電源不要のカードは **RX 550** のみだ。  
個人的には、*Navi14* ベースの製品で更新されることを期待していたが、**RX 5300** でそれは叶わなかった。  
そのこと自体は多少残念には思うが、そう悲観することでもないとも思う。  

[以前](/posts/2020/03/11/polaris-hard-worker/)も書いたが、*Navi14* の製造コストは **RX 570/580** 等のベースとなる *Polaris10* よりもコストが高いと考えられ、*Polaris11 /Polaris12* と比較すれば倍近くのコストだろう。  
それならば、消費電力を重視するようなターゲット帯は APU と **RX 550** でカバーし、*Navi14* ではクロックと消費電力を引き上げ、それより上の性能帯をカバーする方が良い。  
日本 Amazon での価格も、**RX 550 4GB** は 約 &yen;9,000 〜 &yen;10,000、**RX 5500 XT 4GB** は約 &yen;20,000 〜 &yen; 23,000 と、性能よりも製造コストに価格が沿っているように見える。  
言い換えると、性能で考えれば **RX 5300 /5500 XT** はお買い得ではないだろうか。  

下の表にあるように、*Navi14* は TDP が *Polaris11/12* の倍近いものとなっているが、クロックの向上とアーキテクチャの改良により、3倍以上の性能を達成している。  
様々な制約により、プロセス技術においてはコスト、消費電力が以前ほど下がらなくなったが、製品化における工夫により、コストパフォーマンスは向上している。  
メディア部の世代が進んだことにより使い勝手も良くなっている。主に Youtube で用いられる VP9コーディックの HWデコード対応は大きいだろう。  


| | RX 550 | RX 5300 | RX 5500 XT |
| :-- | :--: | :--: | :--: |
| GPU | *Polaris12* | *Navi14* | *Navi14* |
| CU | 8 | 22 | 22 |
| TMU | 32 | 88 | 88 |
| ROP | 16? | 32 | 32 |
||
| Peak Clock<br>(MHz) | 1183 | 1645 | 1845 |
| FP16 (FLOPS) | 1.2 |  9.26 | 10.38  |
| FP32 (FLOPS) | 1.2 | 4.63 | 5.19 |
||
| TDP | 50W  | 100W | 150W |

省電力、補助電源無し好きの人にとっては残念かもしれないが、そう悲観することではない。  
絶対的な消費電力は下がらなくなったが、相対的なワットパフォーマンス、コストパフォーマンスは向上している。  
どうしても省電力を求めたいのであれば、dGPU に代わり APU が性能をどんどん上げてきている。  
メモリ帯域と、それに深く関わるグラフィクス性能では劣るが、CU 6基の **Ryzen 3 PRO 4350G** でもピーク性能では **RX 550** を追い越している。グラフィクス性能も遠からず追い越すだろう。  
追い越すまで行かなくとも、APU の GPU性能で足りる用途も多い。  

省電力マシンの GPU は APU 内蔵のもので十分な時代となった。  
そうした道を往くべきなのだろう。  


<table>
<thead>
<tr>
<th align="left">Navi14 Product</th>
<th align="center">RX 5300M</th>
<th align="center">RX 5300</th>
<th align="center"><span style="color:#08E8DE">Pro 5300M</span></th>
<th align="center">RX 5500M</th>
<th align="center"><span style="color:#08E8DE">Pro 5500M</span></th>
<th align="center">RX 5500</th>
<th align="center"><span style="color:#08E8DE">Pro W5500M</span></th>
<th align="center">RX 5500 XT</th>
<th align="center"><span style="color:#08E8DE">Pro W5500</span></th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">WGPs (CUs)</td>
<td align="center" colspan="2">11 (22)</td>
<td align="center">10 (20)</td>
<td align="center">11 (22)</td>
<td align="center">12 (24)</td>
<td align="center" colspan="4">11 (22)</td>
</tr>

<tr>
<td align="left">SPs</td>
<td align="center" colspan="2">1408</td>
<td align="center">1280</td>
<td align="center">1408</td>
<td align="center">1536</td>
<td align="center" colspan="4">1408</td>
</tr>

<tr>
<td align="left">TMUs</td>
<td align="center" colspan="2">88</td>
<td align="center">80</td>
<td align="center">88</td>
<td align="center">96</td>
<td align="center" colspan="4">88</td>
</tr>

<tr>
<td align="left">ROPs</td>
<td align="center" colspan="9">32</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="9"></td>
</tr>

<tr>
<td align="left">Game Clock (MHz)</td>
<td align="center">1181</td>
<td align="center">1448</td>
<td align="center">?</td>
<td align="center">1448</td>
<td align="center">?</td>
<td align="center">1670</td>
<td align="center">?</td>
<td align="center">1717</td>
<td align="center">?</td>
</tr>

<tr>
<td align="left">Boost Clock (MHz)</td>
<td align="center">1445</td>
<td align="center">1645</td>
<td align="center">1250</td>
<td align="center">1645</td>
<td align="center">1300</td>
<td align="center">1845</td>
<td align="center">1700</td>
<td align="center">1845</td>
<td align="center">1900</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="9"></td>
</tr>

<tr>
<td align="left">Max Memory Size</td>
<td align="center" colspan="2">3 GB</td>
<td align="center" colspan="2">4 GB</td>
<td align="center">8 GB</td>
<td align="center" colspan="2">4 GB</td>
<td align="center" colspan="2">8 GB</td>
</tr>

<tr>
<td align="left">Memory Interface (bit)</td>
<td align="center" colspan="2">96</td>
<td align="center" colspan="7">128</td>
</tr>

<tr>
<td align="left">Memory Bandwidth (GB/s)</td>
<td align="center" colspan="2">168</td>
<td align="center">192</td>
<td align="center">224</td>
<td align="center">192</td>
<td align="center" colspan="4">224</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="9"></td>
</tr>

<tr>
<td align="left">Peak Texture Fill-Rate (GT/s)</td>
<td align="center">127.2</td>
<td align="center">144.8</td>
<td align="center">100.0</td>
<td align="center">144.8</td>
<td align="center">114.4</td>
<td align="center">162.4</td>
<td align="center">149.6</td>
<td align="center">162.4</td>
<td align="center">167.2</td>
</tr>

<tr>
<td align="left">Peak Pixel Fill-Rate (GP/s)</td>
<td align="center">46.2</td>
<td align="center">52.6</td>
<td align="center">40.0</td>
<td align="center">52.6</td>
<td align="center">41.6</td>
<td align="center">59.0</td>
<td align="center">54.4</td>
<td align="center">59.0</td>
<td align="center">60.8</td>
</tr>

<tr>
<td align="left">FP16 (TFLOPS)</td>
<td align="center">8.14</td>
<td align="center">9.26</td>
<td align="center">6.40</td>
<td align="center">9.26</td>
<td align="center">7.98</td>
<td align="center">10.38</td>
<td align="center">9.58</td>
<td align="center">10.38</td>
<td align="center">10.70</td>
</tr>

<tr>
<td align="left">FP32 (TFLOPS)</td>
<td align="center">4.07</td>
<td align="center">4.63</td>
<td align="center">3.20</td>
<td align="center">4.63</td>
<td align="center">3.99</td>
<td align="center">5.19</td>
<td align="center">4.79</td>
<td align="center">5.19</td>
<td align="center">5.35</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="9"></td>
</tr>

<tr>
<td align="left">TBP (W)</td>
<td align="center">65</td>
<td align="center">100</td>
<td align="center">50</td>
<td align="center">85</td>
<td align="center">50</td>
<td align="center">110</td>
<td align="center">85</td>
<td align="center">130</td>
<td align="center">125</td>
</tr>
</tbody>
</table>

