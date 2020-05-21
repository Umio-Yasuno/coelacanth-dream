---
title: "AMD、TDP 6W の AMD 3020e、Athlon Silver 3050e の仕様をひっそりと公開"
date: 2020-05-20T16:35:17+09:00
draft: false
tags: [ "Raven2", "Dali", "Pollock" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

AMD は公式サイトにて **AMD 3020e** 、**Athlon Silver 3050e** の仕様をひっそりと公開した。  
{{< link >}}[AMD 3020e | AMD](https://www.amd.com/en/products/apu/amd-3020e) / [AMD Athlon™ Silver 3050e | AMD](https://www.amd.com/en/product/9896){{< /link >}}
どちらもベンチマーク結果から名前だけは出ており[^1][^2]、 **AMD 3020e** は少し違う **A4-3020E** という名前で Lenovo の製品仕様からも確認されていた。  
{{< link >}}[Lenovo製品の仕様より A4-3020E が確認される | Coelacanth's Dream](/posts/2020/03/04/amd-athlon-3020e/){{< /link >}}

[^1]: <https://twitter.com/TUM_APISAK/status/1223226008519823360>
[^2]: <https://twitter.com/TUM_APISAK/status/1231125733604392960>

どちらも TDP 6W と低電力仕様のプロセッサとなっており、今後登場する Intel のモバイル向け 10nm Atomプロセッサ *Jasper Lake* への対抗製品と考えられる。  

## AMD 3020e
基本的な仕様は既に確認されていたものと変わらず、  
CPUは 2-Core/2-Thread、Base Clock 1.2GHz、Boost Clock 2.6GHz、  
GPUは CU数 3基、Clock 1000MHz。  
あやふやだった、メモリはデュアルチャネル、対応メモリクロックは 2400MHz、そして TDP 6W という仕様がはっきりとした。  

それに加えて、*DisplayPort 1.4 with DSC* に対応するとしている。  
**AMD 3020e** はその仕様と製造プロセス 14nm ということから、ベースとなるダイは [Raven2](/tags/raven2)、ひいてはその別リビジョンである [Dali](/tags/dali) 、[Pollock](/tags/pollock) と思われる。  
しかし、これまでに *Raven2* が *DisplayPort 1.4 with DSC* に対応している、という話は出てきていないと記憶している。  
*Raven2* と 別リビジョン *Dali & Pollock* の違いだが、ディスプレイコントローラの仕様が異なるかもしれない、程度しかわかっておらず、別リビジョンを設け、それに新たなコードネームを与えた意義がはっきりしていなかった。  
もしかすると、*DisplayPort 1.4 with DSC* 対応の有無こそが、*Raven2* と *Dali & Pollock* を分けるものなのかもしれない。  

ただ **AMD 3020e** だけというのは不自然であるため、誤記である可能性も十分に考えられるが。  

また、公式サイトでは **AMD 3020e** となっているが、Lenovo の製品仕様から、**A4-3020E** が正式な製品名と思われる。  

## Athlon Silver 3050e
CPU仕様は 2-Core/4-Thread、Base Clock {{< comple >}}は記載されていないがベンチマークの結果から 1.4GHz と思われる{{< /comple >}}[^2]、Boost Clock は 2.8GHz、  
GPU の仕様と TDP は **AMD 3020e** と同様。  
しかし、**Athlon Silver 3050e** には *DisplayPort 1.4 with DSC* の記載が見当たらない。  

## Raven2ベースプロセッサ比較
**AMD 3020e** と **Athlon Silver 3050e** の登場で、*Raven2* ベースのプロセッサは計10製品となったが、  
*Raven2* は最大でも CPU 2-Core/4-Thread、GPU 3CU と小規模であるため、基本 CPUクロックを調整することで、性能と TDP による製品の差別化が為されている。  
が、その CPUクロック差が 0.2 GHz 程度しかないため、同コア同スレッド数の時、どれだけ実性能に差が出てくるかは気になるところだ。  

なお、**Ryzen Embedded R1102G** のみ、PCIeGen3 4-Lane、メモリ シングルチャネルという仕様になっている。  


<!--
| Raven2 | Core/Thread | Base/Boost Clock | GPU CU | GPU Clock | Default TDP |
| :-- | :--: | :--: | :--: | :--: | :--: |
| Athlon 300U | 2/4 | 2.4/3.3 GHz | 3 | 1000 MHz | 15 W |
| Ryzen 3 3200U | 2/4 | 2.6/3.5 GHz | 3 | 1200 MHz | 15 W |
| Athlon 3000G | 2/4 | 3.5 GHz | 3 | 1100 MHz | 35 W |
| Athlon 300GE | 2/4 | 3.4 GHz | 3 | 1100 MHz | 35 W |
| AMD 3020e | 2/2 | 1.2/2.6 GHz | 3 | 1000 MHz | 6 W |
| Athlon Silver 3050e | 2/4 | 1.4/2.8 GHz | 3 | 1000 MHz | 6 W |
| Athlon Silver 3050U | 2/2 | 2.3/3.2 GHz | 2 | 1100 MHz | 15 W |
| Athlon Gold 3150U | 2/4 | 2.4/3.3 GHz | 3 | 1000 MHz | 15 W |
-->

<table>
<thead>
<tr>
<th align="left">Raven2</th>
<th align="center">Core/Thread</th>
<th align="center">Base/Boost Clock</th>
<th align="center">GPU CU</th>
<th align="center">GPU Clock</th>
<th align="center">Default TDP</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">Athlon 300U</td>
<td align="center" rowspan="2">2/4</td>
<td align="center">2.4/3.3 GHz</td>
<td align="center" rowspan="2">3</td>
<td align="center">1000 MHz</td>
<td align="center" rowspan="4">15 W</td>
</tr>

<tr>
<td align="left">Ryzen 3 3200U</td>
<td align="center">2.6/3.5 GHz</td>
<td align="center">1200 MHz</td>
</tr>

<tr>
<td align="left">Athlon Silver 3050U</td>
<td align="center">2/2</td>
<td align="center">2.3/3.2 GHz</td>
<td align="center">2</td>
<td align="center">1100 MHz</td>
</tr>

<tr>
<td align="left">Athlon Gold 3150U</td>
<td align="center" rowspan="3">2/4</td>
<td align="center">2.4/3.3 GHz</td>
<td align="center" rowspan="7">3</td>
<td align="center">1000 MHz</td>
</tr>

<tr>
<td align="left">Athlon 3000G</td>
<td align="center">3.5 GHz</td>
<td align="center" rowspan="2">1100 MHz</td>
<td align="center" rowspan="2">35 W</td>
</tr>

<tr>
<td align="left">Athlon 300GE</td>
<td align="center">3.4 GHz</td>
</tr>

<tr>
<td align="left">AMD 3020e</td>
<td align="center">2/2</td>
<td align="center">1.2/2.6 GHz</td>
<td align="center" rowspan="4">1000 MHz</td>
<td align="center" rowspan="2">6 W</td>
</tr>

<tr>
<td align="left">Athlon Silver 3050e</td>
<td align="center">2/4</td>
<td align="center">1.4/2.8 GHz</td>
</tr>

<tr>
<td align="left">Ryzen Embedded R1305G</td>
<td align="center">2/4</td>
<td align="center">1.5/2.8 GHz</td>
<td align="center">8-10 W</td>
</tr>

<tr>
<td align="left">Ryzen Embedded R1102G</td>
<td align="center">2/2</td>
<td align="center">1.2/2.6 GHz</td>
<td align="center">6 W</td>
</tr>

</tbody>
</table>
