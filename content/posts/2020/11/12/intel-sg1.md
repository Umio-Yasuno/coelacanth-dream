---
title: "Intel、サーバー向け GPU 「SG1」 と搭載カードを正式発表"
date: 2020-11-12T08:10:43+09:00
draft: false
tags: [ "SG1", "DG1", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Intel は現地時間 2020/11/11 付で、*{{< xe class="lp" >}}アーキテクチャ* を採用するサーバー向け GPU *SG1* と、それを 4基搭載するデータセンター向け PCIeカード **H3C XG310 GPU** を正式発表した。  

 * [Intel® Server GPU Data Center Graphics](https://www.intel.com/content/www/us/en/products/discrete-gpus/server-graphics-card.html)
 * [Intel® Server GPUs for Cloud Gaming and Media Delivery](https://www.intel.com/content/www/us/en/products/docs/discrete-gpus/server-graphics-solution-brief.html)

## SG1 

*SG1* のチップは **Iris Xe MAX** 同様に、コードネーム *DG1* をベースとしていると思われ、パッケージ自体も **Iris Xe MAX** と同一に見える。  


| Intel {{< xe class="lp" >}}/Gen12 | GT0.5 | GT1 | GT2 | GT2 (DG1) |
| :-- | :--: | :--: | :--: | :--: |
| Dual-Sub Slices | 1 | 2 | 6 | 6 |
| &ensp;EUs | 16 | 32 | 96 | 96 |
| &ensp;&ensp;SPs | 128 | 256 | 768 | 768 |
| GPU L3$ Banks | 4 | 4 | 8 | 8 |
| GPU L3$ Size | 1920KB? | 1920KB | 3072KB | 16384KB[^dg1-l3] |
| | RKL / ADL-S | TGL /RKL<br>/ADL-S | TGL /ADL-P | DG1/SG1 |

[^dg1-l3]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)

メモリ構成は LPDDR4x-4267 128-bit、ピークメモリ帯域幅は 68.35GB/s と、帯域面では **Iris Xe MAX** と変わらないが、メモリ容量が **Iris Xe MAX** は 4GB だったのに対し、*SG1* は 8GB となっている。メモリとの接続方式も、x64\*2 から x32\*4 にしていると思われる。  

### H3C XG310 

**H3C XG310** は *SG1* とそれぞれに接続される LPDDR4x 8GB を 4セット、*SG1* 4基をまとめる PLXチップを搭載した PCIeカード。  
*DG1 (SG1)* は PCIe Gen4 に対応しているはずだが、**H3C XG310 GPU** は Gen3 x16 での接続となる。PCIe Gen4 の PLXチップがまだ登場していないからだろうか。  

**H3C XG310 GPU** の製品ページに記載されているスペックを見ると、TDP が 23W と 150W の 2つあるが、前者は GPUチップ単体、後者は GPU 4チップ、メモリ、PLXチップも含めたカード全体の TDP と思われる。  
{{< link >}} [Products & Technology- H3C XG310 GPU- H3C](https://www.h3c.com/en/Products_Technology/Enterprise_Products/Servers/GPU/XG310_GPU/) {{< /link >}}
GPU の動作クロックはそのページには無いが、Intel が性能比較に用いた環境では 1.1GHz の設定となっており、製品版のカードもそのようになっているのではないかと思われる。[^sg1-perf]  

*{{< xe class="lp" >}}アーキテクチャ* をベースとする各 GPU の動作クロックは、*TGL GT2* が 1.35GHz、**Iris Xe MAX** が 1.65GHz、*SG1* が 1.1GHzとなり、サーバー向けということもあり *SG1* は他よりも控えめとなっている。  
*{{< xe class="lp" >}}アーキテクチャ* は最も電力効率に優れる GPUアーキテクチャと Intel は主張しており、その中でも 10nm SuperFinプロセスにおいてスウィートスポットとなる動作クロックが *SG1* の 1.1GHz付近なのだろうか。  
ただ、この中でまともに単体での TDP がはっきりしてるのは *SG1* のみで、*Tiger Lake* は TDP 設定によるし、**Iris Xe MAX** はプラットフォーム全体に設定された消費電力に左右される部分がある。  

[^sg1-perf]: [Server GPU Performance Summary](https://www.intel.com/content/www/us/en/benchmarks/server/graphics/intelservergpu.html)

| Xe-LP | TGL GT2 | Iris Xe MAX | SG1<br>(H3C XG310 DVT) |
| :-- | :--: | :--: | :--: |
| GPU Clock | ~1.35 GHz | 1.65 GHz | 1.1 GHz |

具体的な性能と用途に関しては、*2nd Gen Intel Xeon プロセッサ (Cascade Lake)* 2ソケット、**H3C XG310** 2枚の構成で、Android向けゲームをクラウド上で 120〜160インスタンス (720p, 30fps) を動かすことができるとしている。  

*DG1 (SG1)* は統合GPUである *TGL GT2* と比べて巨大なキャッシュを持ち、またより多くのスレッドを生成、保持する構成となっている。[^dg1-cache-thread]  
こうした構成はまさしくクラウドゲーミングといった処理に向いていると言えるだろう。*{{< xe class="lp" >}}アーキテクチャ* で強化されたデコード/エンコード性能も、ストリーミング処理では活きてくるだろう。  
そうした点で、*SG1* とそれを搭載する **H3C XG310** はモバイル向けの **Iris Xe MAX** よりも *DG1* を最大限活用しているように個人的には思える。  

[^dg1-cache-thread]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)
