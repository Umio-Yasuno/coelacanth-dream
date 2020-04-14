---
title: "AMD、コアあたりの性能を高めたEPYC 7Fx2シリーズを発表"
date: 2020-04-14T22:27:01+09:00
draft: true
tags: [ "Zen2", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
---

AMDは2020/04/14付けで、ベースクロックを上げ、L3cacheを多くあてることでコアあたりの性能を高めた EPYC 7Fx2シリーズを発表した。  

[New 2nd Gen AMD EPYC™ Processors Redefine Performance for Database, Commercial HPC and Hyperconverged Workloads | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/new-2nd-gen-amd-epyctm-processors-redefine-performance-database)

## EPYC 7Fx2シリーズ

追加されたSKUは、8-Core *EPYC 7F32* 、16-Core *EPYC 7F52* 、24-Core *EPYC 7F72* 。  
全体的に、コア数を抑え、L3cacheは多くした(=CCDチップレットが多い)仕様。それまでの同コア数のSKUは、8-Coreで 2-CCD、16-Coreで2-CCDか4-CCD、24-Coreで4-CCDが基本だった。  
L3cacheから最低でも、8-Core *EPYC 7F32* は4-CCD、24-Core *EPYC 7F72* は6-CCD、  
そして *EPYC 7F52* は中でも豪勢な仕様であり、L3cache 256MBということから、 *EPYC Rome* パッケージが搭載可能な8-CCDすべてを、それぞれ2-Coreずつ有効することで 16-Coreとしている。  
CCD内は2-CCX構成であるため、CCXが対称という中で最低までコアを無効化したSKUとなった。  

| | EPYC 7F32 | EPYC 7F52 | EPYC 7F72 |
| :--- | :---: | :---: | :---: |
| Core/Thread | 8/16 | 16/32 | 24/48 |
| Total L3cache | 128MB | 256MB | 192MB |
| Base Clock | 3.7 GHz | 3.5 GHz | 3.2 GHz |
| Boost Clock | 3.9 GHZ | 3.9 GHZ | 3.7 GHz |
| TDP | 180W | 240W | 240W |
| CCD | 4? | 8 | 6?  |

## データベース /商用HPC /高密度な仮想マシンに最適なパフォーマンス
### チップレットアーキテクチャにおけるメモリ帯域
前にも少し書いたが、AMDの *Matisse (Ryzen)* 、*Castle Peak (Threadripper)* 、*Rome (EPYC)* のチップレットアーキテクチャでは、CCDチップレットの数で実質的なメモリ帯域が決まる。[^1]  

[^1]: [Zen2のメモリ帯域 | Coelacanth's Dream](/posts/2019/11/10/zen2-mbw/)

インターフェイスとしては8chあり、その分のメインメモリ容量を搭載することは可能だが、I/Oダイ(sIOD)内部にてDataFabricとメモリコントローラが 32-Byte/Cycle、メモリコントローラとDRANチャネルが16-B/Cで接続されているのに対し、[^2]  
CPUダイ(CCD)とsIOD間の接続が、Write: 16-B/C、Read: 32-B/C となっているため[^3]、メモリ帯域はCCDの数が反映される。  
それにより、*EPYC 7232P* [^6] 、*EPYC 7252* [^7] 、*EPYC 7272* [^8] 、*EPYC 7282* [^9] といった2-CCDのSKUは、4チャネル 2667MHz DIMMに最適化しているとされ、ソケットあたりのメモリ帯域が 85.3 GB/sになっている。  

[^2]: [Zen2のメモリ帯域 | Coelacanth's Dream](/posts/2019/11/10/zen2-mbw/)<br><https://blog.coelacanth-dream.com/image/2019/11/10/pinnacle-diagram.webp>
[^3]: [拡大画像 photo03l | AMD 第2世代EPYCプロセッサの実態を紐解く - 構造・性能・普及状況【Deep Dive】 (1) 2nd Gen EPYC - Internal Architecture DeepDive | マイナビニュース](https://news.mynavi.jp/photo/article/20190819-879251/images/photo03l.jpg)

[^6]: [2nd Gen AMD EPYC™ 7232P | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7232p#product-specs)
[^7]: [2nd Gen AMD EPYC™ 7252 | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7252#product-specs)
[^8]: [2nd Gen AMD EPYC™ 7272 | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7272#product-specs)
[^9]: [2nd Gen AMD EPYC™ 7282 | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7282#product-specs)

これは消費電力を減らすための工夫とのこと。[^4]

[^4]: [AMD Ryzen 3900X and 3700X (Zen2) Review (Page 3 [Out of the Box Performance: CINEBENCH, wPrime, and AIDA64]) | TweakTown](https://www.tweaktown.com/reviews/9051/amd-ryzen-3900x-3700x-zen2-review/index3.html)

AMDは今回発表したモデルを、データベース、商用HPC、高密度な仮想マシン(ハイパーコンバージドインフラストラクチャー)[^5]で最適なパフォーマンスを発揮するプロセッサとしており、  
コア数に対してCCDを多く使用しているのは、それらの用途においてメモリ性能を最適化するため、というのが理由の1つだろう。  

[^5]: [ハイパーコンバージドインフラストラクチャー（HCI）とは : 富士通](https://www.fujitsu.com/jp/products/computing/virtual/tech/term/hci/)

それ以外にもL3cacheが多いことでコアあたりの性能が高くなり、クロックの向上でその作用をさらに強め、SKUの差別化を図れる、というのはチップレットアーキテクチャがまさに柔軟であることを示すのではないかと思う。
