---
title: "Intel、デスクトップ向け DG2/Alchemist SKU の仕様を公開"
date: 2022-09-11T07:56:29+09:00
draft: false
categories: [ "Intel", "Hardware", "GPU" ]
tags: [ "DG2", "Xe-HPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は 2022/09/08 付でデスクトップ向け *DG2/Alchemist* SKU の仕様を公開した。  
内 **Arc A380** は以前より仕様が発表されており、一部では販売もされていた。今回新たに仕様が発表された SKU は **Arc A580/A750/A770** となる。  
**Arc A580/A750/A770** は *ACM-G10/DG2-G10* をベースにしており、SKU 間で {{< xe >}}-Core、Ray Tracing Unit、EU (XVE, XMX Engine)、メモリチップ速度が異なっている。  

 * [Intel® Arc™ Graphics - Q&A: Hardware Specs Explained](https://game.intel.com/story/intel-arc-graphics-qa-hardware-specs/)
 * [Products formerly Alchemist](https://ark.intel.com/content/www/us/en/ark/products/codename/226095/products-formerly-alchemist.html)

| Intel Arc | A380 | A580 | A750 | A770 |
| :--       | :--: | :--: | :--: | :--: |
| GPU       | ACM-G11 | ACM-G10 | ACM-G10 | ACM-G10 |
| {{< xe >}}-Core | 8 | 24 | 28 | 32 |
| Ray Tracing Unit | 8 | 24 | 28 | 32 |
| EU (XVE, XMX Engine) | 128 | 384 | 448 | 512 |
| Memory Bus Width | 96-bit | 256-bit | 256-bit | 256-bit |
| Memory Bandwidth | 186 GB/s | 512 GB/s | 512 GB/s | 560 GB/s |
| Render Slice | 2 | 6? | 7? | 8 |

ニュースリリース内では *full specs* と書いてあるが、Render Slice 数とや Pixel Backend 数については記載が無い。  
Render Slice は {{< xe >}}-Core や各種固定機能ユニットをまとめた GPUクラスタの単位であり、*{{< xe >}}-HPG アーキテクチャ* では基本 {{< xe >}}-Core 4基、Ray Tracing Unit 4基、Texture Sampler 4基、Pixel Backend 2基 (ROP 16基相当)、Geometry Unit、Rasterizer Unit、HiZ (Hierarchical Z) Unit それぞれ 1基で構成されている。  
Render Slice 数の違いはピクセル処理性能やジオメトリ性能等、一部固定機能ユニットによる処理性能に影響する。  

今回発表された SKU では、{{< xe >}}-Core あたりの EU数は 16基で統一されており、{{< xe >}}-Core 内部の構成変更で仕様を調節していないことはわかる。  
*Gen9 アーキテクチャ* までは Sub-Slice ({{< xe >}}-Core 相当) あたりの EU数を変更した調整が見られたが、*Gen11* からはなくなり、Sub-Slice の有効数を変更した調整のみとなっている。  

*{{< xe >}}-HPG アーキテクチャ* における調整方法には、Render Slice あたりの有効 {{< xe >}}-Core 数の変更と、Render Slice 自体の有効数変更があると考えられる。  
現時点で Render Slice 数が公開されている *DG2/Alchemist* SKU では、**Arc A550M/A730M** が Render Slice 数の変更によって、**Arc 350M** では Render Slice あたりの {{< xe >}}-Core 数の変更によって仕様への調節が行われている。  
**Arc 350M** では Render Slice あたりの {{< xe >}}-Core 数が 3基となっている。**Arc 350M** は EU 96基となっており、Render Slice 数の変更では仕様内に収めることが不可能という理由が大きいのかもしれないが、少なくともそうした調節が可能ということが示されている。  

どちらかと言えば、今回発表された **Arc A580/A750** では Render Slice 数の変更が採用されている可能性が高いように思うが、まだはっきりとはしていない部分ではある。  
それにしても SKU が発表された以上、近い内に ark.intel.com で Render Slice 数も含めた仕様が公開されると思われ、この記事は半分以上メモを目的としたものとなる。  

