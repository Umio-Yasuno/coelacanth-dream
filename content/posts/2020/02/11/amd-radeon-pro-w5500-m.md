---
title: "AMD、Radeon Pro W5500/Mを発表"
date: 2020-02-11T01:20:31+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

[先日](/posts/2020/01/31/drm-amdgpu-ids-update-20-01-31/) amdgpu.ids に追加されていたRadeon Pro W5500/Mだが、2020/02/10付けでAMDより公式発表された。  

[Introducing AMD Radeon™ Pro W5500 Workstation Graphics: Groundbreaking Technology for Modern Design and Engineering Professionals | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/introducing-amd-radeontm-pro-w5500-workstation-graphics)  

#### 簡易スペック

| | Radeon Pro W5500 | Radeon Pro W5500M |
| :--- | :--: | :--: |
| Compute Units | 22 | 22 |
| TFLOPS (FP32) | 5.35 | 4.79 |
| Maximum Power Consumption | 125W | 85W |
| GDDR6 | 8GB | 4GB |
| Memory Bandwidth | 224 GB/s | 224 GB/s |
| Memory Interface | 128-bit | 128-bit |
| Display Outputs | 4 | 4 |

Navi14ベースと思われ、構成は11WGP (22CU)、最大クロックはRadeon Pro W5500だと約1900MHz、W5500Mだと約1700MHz、  
メモリインターフェイスは128-bitで、GDDR6メモリの速度は14Gbps、  
ダイショットではNavi10とNavi14のI/O部が変わらないように見えたため、実装されることを密かに期待していたが、  
USB Type-Cコネクタは無く、最大映像出力は4、コネクタの種類はDisplayPortのみ、  
W5500はそれらをまとめ1つの8K 60Hzディスプレイに出力することも可能。  

相変わらず11WGP (22CU)となっているが、Radeon Pro W5500はSKU名がNavi14 Pro-XLであり、今後上位のW5500X (Navi14 Pro-XT?/XTX?)とかが出ることになれば、  
それがNavi14のフルスペック、12WGP (24CU)となるかもしれない。  
そして、Radeon Pro W5500は以前からベンチマーク結果が確認されていたが、その際の CL_DEVICE_MAX_COMPUTE_UNITS は 12 となっていた。  
[AMD 7341:00 performance in CompuBench](https://compubench.com/device.jsp?benchmark=compu20d&os=Windows&api=cl&D=AMD+7341%3A00&testgroup=info)  
ベンチマークソフト側かドライバー側かはわからないが（恐らく後者）、実物が確実に使われるベンチマーク結果としても公式発表前の情報を100%信用することは出来ない、と言えるだろう。  

カードのインターフェイスはPCIe x16スロットだが、電気的にはx8まで。これはそもそものNavi14がPCIe Gen4 x8分までしか持たないため。  

ボード幅は1スロット、ボード長は241mm。  
TBP (Typical Board Power)は125W。  
それを1スロット、ブロアーファンというクーラーで大丈夫なのか、という気はするが、  
TBP 130Wで1スロットのRadeon Pro WX7100という製品もあるため、そこまでは問題ないと考えているのかもしれない。  
[Radeon™ Pro WX 7100 Graphics | GCN architecture | AMD](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-7100#product-specs)  

### vs NVIDIA Quadro&reg; P2200

比較対象にはNVIDIA Quadro&reg; P2200をあげており、SOLIDWORKS&reg; 2020のモデリングを実行させた時の、システムの平均消費電力がRadeon Pro W5500で56.86W、NVIDIA Quadro&reg; P2200で75.17Wと、  
Radeon Pro W5500の方が32.20%省電力だったとAMDは主張する。  

Radeon Pro W5500のTypical Board Powerは125W、NVIDIA Quadro&reg; P2200はNVIDIA公式の情報だと最大消費電力 75Wとなっており、有利な結果を抜粋したにしても、こうも大きく差が出るとデータシートの情報をどこまで信じていいやら。  
[NVIDIA Quadro P2200 | 株式会社 エルザ ジャパン](http://www.elsa-jp.co.jp/products/products-top/graphicsboard_pro/quadro/midrange_2/p2200/)  
[Datasheet Quadro P2200 - quadro-p2200-datasheet-letter-974207-r4-web.pdf](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/productspage/quadro/quadro-desktop/quadro-p2200-datasheet-letter-974207-r4-web.pdf)  
Radeon Pro W5500の方が省電力であることはともかく、TBPと差がありすぎるとAMDに損じゃないか、なんて心配をしてしまう。  
補助電源に1x 6-pinがあるため、処理によっては125W食うことは確かだろうけど。  

またSPECviewperf&reg; 13で計測したマルチタスク性能では、Radeon Pro W5500の方がNVIDIA Quadro&reg; P2200より最大10倍高かったとする。  
この *マルチタスク性能* はRadeonが有利にある傾向があり、AMDは新Radeon Proの発表の際にはほぼ毎回それを持ち出している。  
Radeonにはハードウェアスケジューラーが入っていることによる恩恵だろうか。  

#### Radeon Pro Software for Enterprise 20.Q1
同時にRadeon Pro Software for Enterprise 20.Q1も公開され、Linux版ではRadeon Pro W5700がようやく正式サポートされた。  
[Radeon Pro Software for Enterprise 20.Q1 for Linux Release Notes | AMD](https://www.amd.com/en/support/kb/release-notes/rn-pro-lin-20-q1)  
W5500も遅れずにサポートされている。  

<!--

| Navi14 Product | RX 5300M | <span style="color:#08E8DE">Pro 5300M</span> | RX 5500M | <span style="color:#08E8DE">Pro 5500M</span> | RX 5500 | RX 5500 XT | <span style="color:#08E8DE">Pro W5500</span>
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| WGPs (CUs) | 11 (22) | 10 (20) | 11 (22) | 12 (24) | 11 (22) | 11 (22) | 11 (22)
| SPs | 1408 | 1280 | 1408 | 1536 | 1408 | 1408 | 1408
| TMUs | 88 | 80 | 88 | 96 | 88 | 88 | 88
| ROPs | 32 | 32 | 32 | 32 | 32 | 32 | 32
||
| Game Clock (MHz) | 1181 | ? | 1448 | ? | 1670 | 1717 | ?
| Boost Clock (MHz) | 1445 | 1250 | 1645 | 1300 | 1845 | 1845 | 1900
||
| Max Memory Size | 3 GB | 4 GB | 4 GB | 8 GB | 4 GB | 8 GB | 8 GB
| Memory Interface (bit) | 96 | 128 | 128 | 128 | 128 | 128 | 128
| Memory Bandwidth (GB/s) | 168 | 192 | 224 | 192 | 224 | 224 | 224
||
| Peak Texture Fill-Rate (GT/s) | 127.2 | 100.0 | 144.76 | 114.4 | 162.36 | 162.36 | 167.2
| Peak Pixel Fill-Rate (GP/s) | 46.2 | 40 | 52.6 | 41.6 | 59.0 | 59.0 | 60.8
| FP16 (TFLOPS) | 8.14 | 6.4 | 9.26 | 8.0 | 10.4 | 10.4 | 10.7
| FP32 (TFLOPS) | 4.07 | 3.2 | 4.63 | 4.0 | 5.2 | 5.2 | 5.35
||
| TBP (W) | ~75? | 50 | 85 | 50 | 110 | 130 | 125 |

-->

#### Navi14 Spec/Product Table
PC、PCサイトモード推奨

<table>
<thead>
<tr>
<th align="left">Navi14 Product</th>
<th align="center">RX 5300M</th>
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
<td align="center">11 (22)</td>
<td align="center">10 (20)</td>
<td align="center">11 (22)</td>
<td align="center">12 (24)</td>
<td align="center" colspan="4">11 (22)</td>
</tr>

<tr>
<td align="left">SPs</td>
<td align="center">1408</td>
<td align="center">1280</td>
<td align="center">1408</td>
<td align="center">1536</td>
<td align="center" colspan="4">1408</td>
</tr>

<tr>
<td align="left">TMUs</td>
<td align="center">88</td>
<td align="center">80</td>
<td align="center">88</td>
<td align="center">96</td>
<td align="center" colspan="4">88</td>
</tr>

<tr>
<td align="left">ROPs</td>
<td align="center" colspan="8">32</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Game Clock (MHz)</td>
<td align="center">1181</td>
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
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Max Memory Size</td>
<td align="center">3 GB</td>
<td align="center" colspan="2">4 GB</td>
<td align="center">8 GB</td>
<td align="center" colspan="2">4 GB</td>
<td align="center" colspan="2">8 GB</td>
</tr>

<tr>
<td align="left">Memory Interface (bit)</td>
<td align="center">96</td>
<td align="center" colspan="7">128</td>
</tr>

<tr>
<td align="left">Memory Bandwidth (GB/s)</td>
<td align="center">168</td>
<td align="center">192</td>
<td align="center">224</td>
<td align="center">192</td>
<td align="center" colspan="4">224</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Peak Texture Fill-Rate (GT/s)</td>
<td align="center">127.2</td>
<td align="center">100.0</td>
<td align="center">144.76</td>
<td align="center">114.4</td>
<td align="center">162.36</td>
<td align="center">149.6</td>
<td align="center">162.36</td>
<td align="center">167.2</td>
</tr>

<tr>
<td align="left">Peak Pixel Fill-Rate (GP/s)</td>
<td align="center">46.2</td>
<td align="center">40</td>
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
<td align="center">6.4</td>
<td align="center">9.26</td>
<td align="center">8.0</td>
<td align="center">10.4</td>
<td align="center">9.58</td>
<td align="center">10.4</td>
<td align="center">10.7</td>
</tr>

<tr>
<td align="left">FP32 (TFLOPS)</td>
<td align="center">4.07</td>
<td align="center">3.2</td>
<td align="center">4.63</td>
<td align="center">4.0</td>
<td align="center">5.2</td>
<td align="center">4.79</td>
<td align="center">5.2</td>
<td align="center">5.35</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">TBP (W)</td>
<td align="center">~75?</td>
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

| <span style="color:darkorange">Navi14 SKU</span> | DID | RID | Product |
| :--- | :---: | :---: | :---: |
| ULA | 7340 | 40 | <span style="color:#08E8DE">Radeon Pro 5500M</span> |
| | | 41 | |
| | | 43 | <span style="color:#08E8DE">Radeon Pro 5300M</span> |
| | | 47 | |
| Gaming XTM | | C1 | <span style="color:coral">Radeon RX 5500M</span> |
| Gaming XLM | | C3 | <span style="color:coral">Radeon RX 5300M</span> |
| Gaming XTX | | C5 | <span style="color:coral">Radeon RX 5500 XT</span> |
| Gaming XT | | C7 | <span style="color:coral">Radeon RX 5500</span> |
| XL? | | CF | Radeon RX 5300? |
| WKS Pro-XL | 7341 | 00 | <span style="color:#08E8DE">Radeon Pro W5500</span> |
| | 7343 | 00 | |
| WKS Pro-XTM | 7347 | 00 | <span style="color:#08E8DE">Radeon Pro W5500M</span> |
| WKS Pro-XLM | 734F | 00 | <span style="color:#08E8DE">Radeon Pro W5300M</span> |

