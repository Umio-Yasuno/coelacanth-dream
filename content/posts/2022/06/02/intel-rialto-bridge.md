---
title: "Intel、次世代の HPC 向け GPU となる Rialto Bridge を公開"
date: 2022-06-02T07:24:29+09:00
draft: false
categories: [ "Intel", "CPU", "GPU", "Hardware" ]
tags: [ "Rialto_Bridge", "Xe-HPC" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は ISC 2022 における基調講演にて、現世代の HPC、データセンタ向け GPU *Ponte Vecchio* の次世代となる *Rialto Bridge* の存在を公開した。  

 * [Accelerated Innovations for Sustainable, Open HPC](https://www.intel.com/content/www/us/en/newsroom/opinion/accelerated-innovations-sustainable-open-hpc.html)
 * [Intel Editorial: Accelerated Innovations for Sustainable, Open HPC :: Intel Corporation (INTC)](https://www.intc.com/news-events/press-releases/detail/1550/intel-editorial-accelerated-innovations-for-sustainable)

## Rialto Bridge
*Rialto Bridge* の特徴として、最大 {{< xe >}}-Core 160基 (*Ponte Vecchio 2-Stack* は最大 128基) を搭載し、演算性能、メモリ帯域、I/O帯域、電力効率の向上を実現するとしている。また許容 TDP の引き上げにより、面積あたりの性能も向上する。  

今回公開された *Rialto Bridge* の CG画像では、*Ponte Vechhio* の Compute Tile 8基と Rambo (Random Access Memory, Bandwidth, Optimized) Tile 4基にあたる部分が、4基の Tile に置き換えられている。  

*Ponte Vecchio* では、実行ユニット (EU, Vector Engine, XMX)、Load/Store ユニット、命令キャッシュ、L1データキャッシュ/Shared Local Memory をまとめた {{< xe >}}-Core を、Compute Tile あたり 4基搭載している。  
そして 1個の Hardware Context を保持する {{< xe >}}-Slice を {{< xe >}}-Core 8基、要は Compute Tile 2基で構成する。  
*Rialto Bridge* で Compute Tile を *Ponte Vecchio* の 8基から 4基にしたのは、{{< xe >}}-Core の増量と密度向上の他に、Tile 構成をシンプルにし、Base Tile に搭載された Fabric への帯域を削減する目的があるのではないかと思われる。  

*Rialto Bridge* は 2023年にサンプリング品の出荷を目標としている。  

## Falcon Shores
x86 CPU と {{< xe >}} GPU を一つのソケットに統合する *Falcon Shores* の存在は 2022/02/17 の Intel’s 2022 Investor Meeting で既に発表されていたが、ソケットあたり 4基の x86/{{< xe >}} Tile で構成され、それぞれの数を柔軟に変更可能なアーキテクチャであることが今回発表された。[^intel-2022]  
x86/{{< xe >}} Tile どちらかのみの構成も可能と思われ、CG画像からは 4x x86/{{< xe >}}、2x x86 + 2x {{< xe >}} の構成が確認できる。  

*Falcon Shores* は現世代の x86 CPU プラットフォームと比較して 5倍以上の電力効率、面積あたりの演算性能、メモリ容量と帯域の向上を実現するとし、2024年を目標としている。  

[^intel-2022]: [Intel Technology Roadmaps and Milestones](https://www.intel.com/content/www/us/en/newsroom/news/intel-technology-roadmaps-milestones.html)

{{< ref >}}
 * [Intel® Iris® Xe GPU Architecture](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-gpu-optimization-guide/top/xe-arch.html)
 * [ISSCC 2022: Wie Intel für Ponte Vecchio 63 Tiles in ein Package bringt - Hardwareluxx](https://www.hardwareluxx.de/index.php/news/hardware/grafikkarten/58176-isscc-2022-wie-intel-fuer-ponte-vecchio-63-tiles-in-ein-package-bringt.html)
 * [Behind Intel’s HPC Chip that Will Pierce the Exascale Barrier - IEEE Spectrum](https://spectrum.ieee.org/intel-s-exascale-supercomputer-chip-is-a-master-class-in-3d-integration)
 * [Intel’s Ponte Vecchio And AMD’s Zen 3 Show The Promise Of Advanced Semiconductor Packaging Technology](https://www.forbes.com/sites/willyshih/2022/02/22/intels-ponte-vecchio-and-amds-zen-3-show-the-promise-of-advanced-semiconductor-packaging-technology/?sh=7f65a6515abe)
{{< /ref >}}
