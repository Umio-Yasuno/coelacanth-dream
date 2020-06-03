---
title: "AMD、RX 5600シリーズ、RX 5700Mを発表"
date: 2020-01-07T09:53:25+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "Radeon RX 5600" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

AMDよりRadeon RX 5600シリーズ、RX 5700Mが正式発表された。  
[AMD Unveils Four New Desktop and Mobile GPUs, including AMD Radeon™ RX 5600 Series: Ultimate 1080p Gaming, Breath-Taking Visuals and Game-Changing Software Features - AMD](http://ir.amd.com/news-releases/news-release-details/amd-unveils-four-new-desktop-and-mobile-gpus-including-amd)  

RX 5600 XTは 2020-01-21 に各社ボードパートナーより販売が、  
RX 5600/M、RX 5700MはOEMより第一四半期（4月〜6月）の販売が予定されている。  
内、Dellから 2020-04 にゲーミングラップトップが（RX 5600M/RX 5700M ?）、数週間内にゲーミングデスクトップがAlienwareブランドで出る予定だ。  

RX 5600シリーズのトランジスタ数が10.3Bと、RX 5700シリーズと同数なことから、使われているダイは[Navi10](/tags/navi10)で確定だろう。  

RX 5600 XTのコア周りはRX 5700と同一であり、差別化のためクロックがだいぶ控えめで、  
各社のオリジナルモデルを見てもBoost Clockが、高くて1620MHzとRX 5700のGame Clockを超えないくらいに調整されている。  
メモリもRX 5600シリーズ、RX 5700Mでは12Gbpsのチップが使われ、RX 5600シリーズでは256-bitから減らされ192-bitとなる。  
そのことでメモリコントローラーと接続されているL2キャッシュも3MBに減っているはずだ。ROPが減っていないのは、たぶんRDNAアーキテクチャの恩恵。  
[Navi14のちょっとしたまとめ ](/posts/2019/12/06/navi14-a-little-matome/)  

スペックの数字だけだとRX 5600シリーズは他の同世代Naviから見劣りするが、見方によってはコストパフォーマンス、ワットパフォーマンスにおいてかなり優秀となるだろう。  

Navi10ベース、そしてGTX1660Ti対抗でありながら今の今まで出てこなかったのは、この優秀によって需要を食い合い、RX 5700シリーズの売れ行きが悪くなるのを嫌った結果なのかもしれない。  

<table>
<thead>
<tr>
<th align="left">Navi10 Product</th>
<th align="center"><span style="color:tomato">RX 5600</span></th>
<th align="center"><span style="color:tomato">RX 5600 XT</span></th>
<th align="center"><span style="color:tomato">RX 5600M</span></th>
<th align="center"><span style="color:tomato">RX 5700M</span></th>
<th align="center"><span style="color:skyblue">W5700</span></th>
<th align="center"><span style="color:tomato">RX 5700</span></th>
<th align="center"><span style="color:tomato">RX 5700 XT</span></th>
<th align="center"><span style="color:skyblue">W5700X</span></th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">WGPs</td>
<td align="center">16</td>
<td align="center" colspan="5">18</td>
<td align="center" colspan="2">20</td>
</tr>

<tr>
<td align="left">CUs</td>
<td align="center">32</td>
<td align="center" colspan="5">36</td>
<td align="center" colspan="2">40</td>
</tr>

<tr>
<td align="left">SPs</td>
<td align="center">2048</td>
<td align="center" colspan="5">2304</td>
<td align="center" colspan="2">2560</td>
</tr>

<tr>
<td align="left">TMUs</td>
<td align="center">128</td>
<td align="center" colspan="5">144</td>
<td align="center" colspan="2">160</td>
</tr>

<tr>
<td align="left">ROPs</td>
<td align="center" colspan="8">64</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Game Clock (MHz)</td>
<td align="center" colspan="2">1375</td>
<td align="center">1190</td>
<td align="center">1620</td>
<td align="center">?</td>
<td align="center">1625</td>
<td align="center">1755</td>
<td align="center">?</td>
</tr>

<tr>
<td align="left">Boost Clock (MHz)</td>
<td align="center" colspan="2">1560</td>
<td align="center">1265</td>
<td align="center">1720</td>
<td align="center">1929</td>
<td align="center">1725</td>
<td align="center">1905</td>
<td align="center">1855</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Max Memory Size (GB)</td>
<td align="center" colspan="3">6</td>
<td align="center" colspan="4">8</td>
<td align="center">16</td>
</tr>

<tr>
<td align="left">Memory Interface (bit)</td>
<td align="center" colspan="3">192</td>
<td align="center" colspan="5">256</td>
</tr>

<tr>
<td align="left">Memory Speed (Gbps)</td>
<td align="center" colspan="4">12</td>
<td align="center" colspan="4">14</td>
</tr>

<tr>
<td align="left">Memory Bandwidth (GB/s)</td>
<td align="center" colspan="3">288</td>
<td align="center">384</td>
<td align="center" colspan="4">448</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Peak Texture Fill-Rate (GT/s)</td>
<td align="center">199.68</td>
<td align="center">224.64</td>
<td align="center">182.16</td>
<td align="center">247.70</td>
<td align="center">277.78</td>
<td align="center">248.4</td>
<td align="center">304.80</td>
<td align="center">296.8</td>
</tr>

<tr>
<td align="left">Peak Pixel Fill-Rate (GP/s)</td>
<td align="center" colspan="2">99.84</td>
<td align="center">80.96</td>
<td align="center">110.10</td>
<td align="center">123.46</td>
<td align="center">110.4</td>
<td align="center">121.90</td>
<td align="center">118.72</td>
</tr>

<tr>
<td align="left">FP16 (TFLOPS)</td>
<td align="center">12.78</td>
<td align="center">14.38</td>
<td align="center">11.66</td>
<td align="center">15.85</td>
<td align="center">17.78</td>
<td align="center">15.9</td>
<td align="center">20.28</td>
<td align="center">11.0</td>
</tr>

<tr>
<td align="left">FP32 (TFLOPS)</td>
<td align="center">6.39</td>
<td align="center">7.19</td>
<td align="center">5.83</td>
<td align="center">7.93</td>
<td align="center">8.89</td>
<td align="center">7.95</td>
<td align="center">10.14</td>
<td align="center">9.5</td>
</tr>

<tr>
<td align="left">TBP (W)</td>
<td align="center" colspan="2">150</td>
<td align="center">?</td>
<td align="center">?</td>
<td align="center">190<br>205(+USB)</td>
<td align="center">180</td>
<td align="center">225</td>
<td align="center">?</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="8"></td>
</tr>

<tr>
<td align="left">Launch Price</td>
<td align="center">?</td>
<td align="center">$279</td>
<td align="center">?</td>
<td align="center">?</td>
<td align="center">?</td>
<td align="center">$349</td>
<td align="center">$399</td>
<td align="center">?</td>
</tr>

</tbody>
</table>

#### 答え合わせ
推測が当たったと言えるのはTBP、RX 5600の16WGPくらい？  
ただ後者はRX 5600 XTのスペックが判明してからのもので、あまり喜べない。  
価格は$10の違いだけで惜しかった。  
1080p144fpsというのも方向は合ってたけどそこまでピッタリではない。  

#### 感想
メモリ以外はあまり削っていないのは嬉しい驚きだった。<span class="hide">（コア部分のスペックを削っても原価的には変わらないけど）</span>  
メモリをケチらないのがAMDの良い所だったのに、という声も見かけるが、それを言われていたPolarisにはもう後継のRX 5500 XTがいる。  
メモリバス幅 128-bitではあるがピーク帯域はGDDR5 256-bitと大した差がない上、マルチメディア周りではRX 5500 XTに分がある。  
価格はと言うと……、RX 5500 XTがもう\2,000ほど下がれば、さすがにRX 590/580を求める層が流れていくと思いたい。  

<hr>
<code>RX 5600関連</code>

 * [Radeon RX 5600推測 ](/posts/2019/12/18/radeon-rx-5600-guess/)
 * [RX 5600情報アップデート & 推測 Part2](/posts/2019/12/22/rx-5600-info-update-guess/)
 * [RX 5600 XTのスペックが判明? ](/posts/2019/12/29/rx-5600-xt-cld/)
