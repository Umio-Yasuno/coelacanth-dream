---
title: "AMD、 Athlon Silver 3050U, Gold 3150Uを発表"
date: 2020-01-07T09:43:57+09:00
draft: false
tags: ["Radeon", "GCN", "Raven", "GFX9", "gfx909" ]
keywords: [ "Radeon", "Athlon", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

AMDより、7nm Ryzen 4000 U/H-seriesと共に14nm Athlon 3000U seriesが発表された。  
[AMD Announces World’s Highest Performance Desktop and Ultrathin Laptop Processors at CES 2020 - AMD](http://ir.amd.com/news-releases/news-release-details/amd-announces-worlds-highest-performance-desktop-and-ultrathin)  

Silver 3050U、Gold 3150Uの2つがあり、  
Silver 3050UはCPUが2-Core/2-Thread Base: 2.3GHz/Boost: 3.2GHz、GPUが2CU 1100MHz、  
Gold 3150Uは2C/4T Base: 2.6GHz/Boost: 3.3GHz、3CU 1000MHzというスペックだ。  
CPU/GPUと14nmであることから、ダイとしては[Raven2](/tags/raven2)だと思われるが、発表に使われたスライドの左下には、

 > AMD RENOIR/DALI PRODUCT LAUNCH PRESS DECK

とあり、[Dali](/tags/dali)の可能性もある。  
[AMD's Slide Deck - AMD Ryzen 4000 Mobile APUs: 7nm, 8-core on both 15W and 45W, Coming Q1 - AnandTech](https://www.anandtech.com/show/15324/amd-ryzen-4000-mobile-apus-7nm-8core-on-both-15w-and-45w-coming-q1/2)  
ただそうなると、Raven2からプロセスも設計も変更していないDaliの存在意義が謎なため、Raven2である可能性のが高いと個人的には見ている。  

{{% youtube id="T-xWH1vgxTM" title="AMD Athlon™ 3000 Series Mobile Processors – Real Performance Meets Modern Features" %}}

AMDはRyzenのセグメントにおいて、対抗であるIntelのCore iシリーズに合わせた 3/5/7 を取り入れたが、  
今回はAthlonにPentiumのセグメント、Silver/Goldを取り入れた形となる。  
PentiumではベースがSilverだとAtom系、GoldがCore系と少しややこしかったが、Athlon Silver/Goldではアーキテクチャ面において違いはないはずだ。  

### おまけ
Raven2を使っている（と思われる）モデルの性能を表にまとめてみたが、差別化にちょっとした苦労が見られる。  

<table>
<thead>
<tr>
<th align="left">Raven2 Product</th>
<th align="center">3050U</th>
<th align="center">300U</th>
<th align="center">3000G</th>
<th align="center">3150U</th>
<th align="center">3200U</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">CPU Cores / Threads</td>
<td align="center">2/2</td>
<td align="center" colspan="4">2/4</td>
</tr>

<tr>
<td align="left">CPU Base Clock (GHz)</td>
<td align="center">2.3</td>
<td align="center">2.4</td>
<td align="center">3.5</td>
<td align="center">2.6</td>
<td align="center">2.6</td>
</tr>

<tr>
<td align="left">CPU Boost Clock (GHz)</td>
<td align="center">3.2</td>
<td align="center">3.3</td>
<td align="center">?</td>
<td align="center">3.3</td>
<td align="center">3.5</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">GPU Cores</td>
<td align="center">2</td>
<td align="center" colspan="4">3</td>
</tr>

<tr>
<td align="left">GPU Clock (MHz)</td>
<td align="center">1100</td>
<td align="center">1000</td>
<td align="center">1100</td>
<td align="center">1000</td>
<td align="center">1200</td>
</tr>
</tbody>
</table>

