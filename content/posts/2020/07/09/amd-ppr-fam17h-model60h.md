---
title: "AMD、「PPR for Family 17h Model 60h, Revision A1 Processor」を公開"
date: 2020-07-09T22:35:22+09:00
draft: false
tags: [ "Renoir", "Zen_2" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

AMD は 2020/07/08付で **Processor ProgrammingReference (PPR)for AMD Family 17h Model 60h, Revision A1 Processors** を公開した。  
{{< link >}}[Tech Docs | AMD](https://www.amd.com/en/support/tech-docs?keyword=family+17h+model+60h){{< /link >}}

`Family 17h Model 60h` は、TSMC 7nmプロセスで製造される、CPU は *Zen 2 アーキテクチャ* 最大 8-Core/16-Thread、GPU は *Vega アーキテクチャ (gfx908)* 最大 8CU、最大 2-Render Backend(8-ROP相当)の APU、*Renoir* を指す。  

`Model 60-67h` プロセッサ、*Renoir* は高いエネルギー効率と優れた性能を実現し、ラップトップ、デスクトップ環境のニーズを満たすため開発された、第9世代APUとしている。  
特長として、内部バスに *AMD Infinity Fabric (Scalable Data Fabric/SDF)* を採用しており、それによって最小限のレイテンシ、最大化された帯域幅の利用効率を可能とし、システム全体の性能を向上させている。  
また、*Renoir* は CPU、GPU、I/O、マルチメディア、メモリインターフェイスが統合された SoC であり、外部にチップセットを必要としないため、BoM(部品点数)を削減している。  

搭載されるパッケージには、*FP6* と *AM4* の 2種類存在し、*FP6* はモバイル向けのメインストリームクラス、*AM4* はデスクトップ向けとなる。  
TDP は、モバイル向けのソリューションで {{< comple >}}Ryzen Mobile 4000シリーズでは cTDP でも 10Wが下限とされているが {{< /comple >}} 4.5W-55W が利用可能とされている。  

内部GPUの Device ID には `0x1636` が使われるとしている。  

{{< pindex >}}

 * [謎と推測](#guess)
   * [x86\_Model から見えること](#rn-x86_model)
   * [広い TDP 範囲とその下限](#rn-tdp-range)

{{< /pindex >}}

## 謎と推測 {#guess}

今回公開された資料で気になるのは、x86\_Model の範囲を `Model 60-67h` としている点、対応TDP の範囲を 4.5W-55W としている点だ。  
そして、ここから先は個人の推測、妄想が多分に含まれることを留意していただきたい。  

### x86\_Model から見えること {#rn-x86_model}
x86_Model が `Model 60-6Fh` ではなく、`Model 60-67h` と若干中途半端になっていることには心当たりがあり、恐らく *Raven* と *Picasso* と同じパターンと思われる。  

まず、*Raven* の x86_Model は `Model 11h` となっており、その後継であり、内部構成はほとんど変わらない *Picasso* は `Model 18h` となっている。  
`Model 11-18h` と `Model 60-67h` の間隔は一致し、そして *Raven* に対する *Picasso* のような関係を *Renoir* に持つ APU には、つい最近名を現した *Lucianne* がいる。{{< comple >}}何か言われそうだけど、あれから特に新情報も無いため、この名で統一する。{{< /comple >}}  
{{< link >}}[AMD Renoir の兄弟？ 現れるは Lucianne | Coelacanth's Dream](/posts/2020/06/20/amd-lucianne-apu/){{< /link >}}

要は、やはり内部構成、アーキテクチャは *Renoir* と変わらず、プロセスを最適化または進めた APU(*Lucianne*) がロードマップ上に存在し、それが `Model 67h` とされるのではないかということを主張したい。  

TSMC 7nmプロセス(*N7*) と物理設計に互換性を持つプロセスには、TSMC の第2世代 7nmプロセス(*N7P*)、6nmプロセス(*N6*) がある。  
*N7* と比較して、*N7P* は同じ消費電力なら最大で 7% 性能向上、同じクロックなら 10% 省電力。  
*N6* は性能、消費電力に関しては不明で、ただロジック密度は 1.8倍と謳われている。  

### 広い TDP 範囲とその下限 {#rn-tdp-range}
`Model 60-67h` プロセッサは TDP 4.5-55W の範囲に対応すると記載されており、TDP 4.5W というのはかなり低く、優秀であるように思える。  
*Raven2* シリコンダイをベースとする `Model 20-2Fh` プロセッサ {{< comple >}}Dali /Pollock{{< /comple >}} も TDP は 4.5W から対応しており、これは *Renoir* と *Raven2* のダイサイズが大体同じであることから(約155mm<sup>2</sup>)、まあ可能なのかもしれないという気はする。  
しかしコストという点で両者は大きく異なり、プロセスから、*Renoir (7nm)* は *Raven2 (14nm)* の倍近いコストであると思われる。[^7nm-cost]  

[^7nm-cost]: [【後藤弘茂のWeekly海外ニュース】AMDアーキテクチャの変化の原因となった7nmプロセスの特性 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1199176.html)

モバイル向けの TDP 10W 以下のラインは、今 *Raven2* ベースの *Dali* が頑張って埋めているところであり、TSMC 7nmで製造される `Model 60-67h` がそのラインに来るとして一体いつになるのかという気になるし、コストと必要とされる性能を考えても無駄が多く、しばらく、だいぶ長い間は *Raven2* で十分ではないかとも思う。  

AMD は ISSCC 2020 にて、*Zen 2 アーキテクチャ* の CCX構成に、*Raven2* と同様であろう 2-Core、L3 4MB の構成を発表しており、[^isscc-2020-zen2]  
将来 TDP 10W 以下のラインを *Raven2* から置き換えるのであれば、その構成を採用した低コストなプロセッサが適しているように思う。  

[^isscc-2020-zen2]: [Zen 2: The AMD 7nm Energy-Efficient High-Performance x86-64 Microproc…](https://www.slideshare.net/AMD/zen-2-the-amd-7nm-energyefficient-highperformance-x8664-microprocessor-core),<br> <https://image.slidesharecdn.com/isscc2020zen2finalfordistribution-20200224-200224225255/95/zen-2-the-amd-7nm-energyefficient-highperformance-x8664-microprocessor-core-9-638.jpg?cb=1582584849>
 
結局の所、今回の `Model 60-67h` プロセッサが TDP 4.5W から対応可能というのは、意図が読み切れないところがある。  

| Family17h Processor | CPU Arch | x86\_Model |
| :-- | :--: | :--: |
| Summit Ridge<br> /Naples | Zen | 1h |
| Pinaccle Ridge<br> /Colfax | Zen+ | 8h |
| Raven Ridge | Zen/+ | 11h |
| Picasso | Zen+ | 18h |
| Raven2<br> /Dali | Zen | 18h |
| Dali(for Chromebook, some)<br>/Pollock | Zen | 20h |
| Matisse | Zen 2 | 71h |
| Castle Peak<br> /Rome | Zen 2 | 31h |
| Renoir | Zen 2 | 60h
| Lucianne? | Zen 2 | 67h? |

{{< ref >}}

 * [7nm Technology - Taiwan Semiconductor Manufacturing Company Limited](https://www.tsmc.com/english/dedicatedFoundry/technology/7nm.htm)
 * [TSMC Announces Performance-Enhanced 7nm & 5nm Process Technologies](https://www.anandtech.com/show/14687/tsmc-announces-performanceenhanced-7nm-5nm-process-technologies)
 * [PCテクノロジートレンド 2020 - プロセス編 (1) プロセス編 - TSMCとSamsung | マイナビニュース](https://news.mynavi.jp/article/20200101-949108/)

{{< /ref >}}
