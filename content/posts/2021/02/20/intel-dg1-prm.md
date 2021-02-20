---
title: "Intel、DG1/Iris Xe Max PRM を公開"
date: 2021-02-20T15:20:22+09:00
draft: true
tags: [ "DG1", "Gen12" ]
# keywords: [ "", ]
categories: [ "Intel", "Hardware", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Intel は GPUのオープンソースドライバーに関係したリソースを公開する [Intel® Graphics for Linux\* | 01.org](https://01.org/linuxgraphics) にて、*Intel® Iris® XeMAX Graphics Open SourceProgrammer's Reference Manual* を新たに公開した。  

 * [2020 Discrete GPU formerly named "DG1" | 01.org](https://01.org/node/37109)

## DG1 スペック

SKU とそのスペック一覧は *Volume 4: Configurations* に記載されている。  

 * [Intel® Iris® Xe MAX Graphics Open Source Programmer's Reference Manual For the 2020 Discrete GPU formerly named "DG1" Volume 4: Configurations](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-dg1-vol04-configurations.pdf)

個人的に気になっていた Pixel Rate は 24 pixel/clock (24-ROP相当) とあった。  
資料に記載されているのは *DG1* のスペックのみで、*Tiger Lake* は無いため、その Pixel Rate が *DG1* 固有なのか、*Gen12 GT2* で共通しているのかは不明。  

### DG1/Gen12LPアーキテクチャでの変更点

Intel の最近の GPUアーキテクチャは大体、Slice と呼ぶ塊の中に ジオメトリエンジンやラスタライザ、ピクセルバックエンド、複数の Sub-Slice が収められ、  
Sub-Slice には、実行ユニットである EU、テクスチャ/メディアサンプラー、ロード/ストアユニット、SLM (Shared Local Memory)、L1データ/命令キャッシュが格納されている。  

前世代の Gen11アーキテクチャは全体で Slice 1基、Sub-Slice は最大 8基、Sub-Slice内の EU は 8基という構成を採っていた。  
Gen12LPアーキテクチャでも Slice 1基というのは変わらないが、Sub-Slice が Dual Sub-Slice と名を変え、Dual Sub-Slice内の EU数は 16基となっている。  
サンプラーの数は見かけ上では減ったが、スループットは倍となっており、EU数に対する性能は変わらない。  
ロード/ストアユニットのデータポートは、Gen11 では Sub-Slice 2基あたり 1ポートだったのが、Gen12 では Dual Sub-Slice 1基あたり 1ポートとなり、L3キャッシュ、SLM に対するロード/ストア性能は大きく向上している。  
ただ、Dual Sub-Slice とするにあたって、内部の規模を全て倍とした訳ではないようで、L1命令キャッシュは 48KB のままとなっており、相対的に減っていると見ることもできる部分もある。  

サンプラー等の固定機能ユニットが利用/管理するオンチップメモリ、URB (Unified Return Buffer) は、Gen11 では L3キャッシュの一部を割り当てる方式 (プログラマブル) だったが、*DG1* では全体で 768KB の固定サイズが割り当てられるようになった。  
Gen11 では L3キャッシュバンクあたり最大 128KB を URB に割り当てることが可能であり、*DG1* は L3キャッシュバンクを 6基持つため、容量自体は Gen11 から引き継いでいる。  
*DG1* は 16MB という巨大な L3キャッシュを持つが、URB を固定サイズとし、L3キャッシュとは別に持つことで、より多い容量をデータキャッシュやタイルキャッシュに割り当てられるようになっている。  
しかし、L3キャッシュと URB を完全に分離した訳ではなく、Read はそれぞれ独立した帯域を確保できるが、Write は L3キャッシュと URB で共有するとある。  
また、L3キャッシュとメモリコントローラーとを繋ぐ GTI (Graphics Technology Interface) / Ring Interface は 2基に増やされている。  

これらの変更点から、Gen12LPアーキテクチャは前世代から単純に強化したのではなく、dGPU として独立してメモリを扱うことを想定して演算性能とそれに対するキャッシュサイズ、帯域を見直したという印象を受ける。  


{{< ref >}}

 * [4つのマイクロアーキテクチャからなるIntelの新GPU「Xe」 - Hot Chips 32 (1) | TECH+](https://news.mynavi.jp/article/20200901-1262986/)
 * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)

{{< /ref >}}
