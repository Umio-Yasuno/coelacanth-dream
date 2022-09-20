---
title: "AMD、Mendocino SoC ベースのモバイル向け Ryzen/Athlon 7020 シリーズを正式発表"
date: 2022-09-21T04:40:27+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Mendocino", "RDNA_2", "GC_10_3_7" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

2022/09/20 付、AMD は *Mendocino SoC* をベースとしたモバイル向け Ryzen/Athlon シリーズを発表した。  
*Mendocino SoC* は TSMC 6nm FinFETプロセスで製造され、*CPU: Zen 2 4-Core/8-Thread + GPU: RDNA 2 1-WGP (2-CU)* の構成を取り、メモリには 64-bit LPDDR5-5500 をサポートしている。  

 * [AMD Ryzen 7020 Series Processors for Mobile Bring High-End Performance and Long Battery Life to Everyday Users | AMD](https://www.amd.com/en/press-releases/2022-09-20-amd-ryzen-7020-series-processors-for-mobile-bring-high-end-performance)

## Ryzen/Athlon 7020 Series {#7020}
今回発表された Ryzen/Athlon 7020 シリーズ SKU は **Athlon Gold 7220U, Ryzen 3 7320U, Ryzen 5 7520U**、そしてニュースリリースには何故か含まれていないが **Athlon Silver 7120U** の 4種類となる。  
4種類ともメモリと TDP、GPU部の仕様は共通しており、GPU部はモデル名 **AMD Radeon 610M**、1-WGP (2-CU)、1900MHz、最大ディスプレイ出力数 4基となっている。  
Default TDP は 15W であり、今回 **7020e シリーズ (TDP: 6W)** は発表されなかった。  
また、CPU ソケット、パッケージ名は公開されていない。  

 * [AMD Athlon™ Silver 7120U | AMD](https://www.amd.com/en/product/12211)
 * [AMD Athlon™ Gold 7220U | AMD](https://www.amd.com/en/product/12206)
 * [AMD Ryzen™ 3 7320U | AMD](https://www.amd.com/en/product/12201)
 * [AMD Ryzen™ 5 7520U | AMD](https://www.amd.com/en/product/12196)

| Mendocino | Ryzen 5 7520U | Ryzen 3 7320U | Athlon Gold 7220U | Athlon Silver 7120U |
| :--       | :--:          | :--:          | :--:              | :--:                |
| Core/Thread | 4/8         | 4/8           | 2/4               | 2/2                 |
| Base/Boost Clock | 2.8GHz/4.3GHz | 2.4GHz/4.1GHz | 2.4GHz/3.7GHz | 2.4GHz/3.5GHz    |
| L3 Cache  | 4MB           | 4MB           | 4MB               | 2MB                 |

**Athlon Silver 7120U** は 2-Core/2-Thread、L3 Cache 2MB と、フルスペックから半分の Core/Thread が無効化されているだけでなく、SMT と L3 Cache の半分 (2MB) も無効化した珍しい構成となっている。  
2-Core/2-Thread の構成には、*Raven2/Dali (14nm, Zen, Vega)* ベースの **Athlon Silver 3050C** や *Raven2/Pollock* ベースの **AMD 3015e** もいるが、それらは L3 Cache 4MB であり、L3 Cache を部分的に無効化してはいない。  

## Mendocino {#mdn}
*Mendocino SoC* のダイサイズは AMD 公式では 100mm2 としており、チップレット構成を採用しないモノリシック構成な最近の AMD SoC としてはかなり小さいダイサイズとなっている。  
GF 14nmプロセスで製造され、*CPU: 2-Core/4-Thread + GPU: Vega 3-CU* という構成の *Raven2 (Dali/Pollock)* は 148.55mm2、  
TSMC 7nmプロセス、*CPU: 8-Core/16-Thread + GPU: Vega 8-CU* の *Renoir (Lucienne)* は 155.01mm2 である。  
*Mendocino SoC* では CPU と GPU の規模を抑えると同時に、メモリバス幅を 64-bit に、PCIe I/O の規模を従来から小さく、SATAコントローラを搭載しないといったダイサイズを小さくする様々な方策を取っており、その結果 100mm2 というダイサイズになったと考えられる。  

{{< ref >}}
 * [AMD Raven2 のダイ観察 & Zen系 APU を比較 | Coelacanth's Dream](/posts/2020/07/24/raven2-dieshot-and-compare-zen-apu/)
 * [AMD Sabrina SoC に向けたさらなるパッチ | Coelacanth's Dream](/posts/2022/01/14/amd-sabrina-soc-more-patch/)
{{< /ref >}}
