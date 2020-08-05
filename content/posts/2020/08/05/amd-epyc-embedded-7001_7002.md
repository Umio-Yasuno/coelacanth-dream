---
title: "AMD、EPYC Embedded に 7001/7002シリーズを追加"
date: 2020-08-05T17:54:56+09:00
draft: false
tags: [ "Zen", "Zen_2" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
summary: "AMD の公式サイトにて EPYC Embedded 7001/7002 シリーズ のページが追加、モデルの仕様が記載されていた。モデルナンバーとそれに対応する仕様は、基本現行のサーバー向け EPYC 7001/7002 シリーズ のものと変わらない。"
---

AMD の公式サイトにて **EPYC Embedded 7001/7002 シリーズ** のページが追加、モデルの仕様が記載されていた。  
{{< link >}} [AMD™ EPYC Embedded Family | AMD](https://www.amd.com/en/processors/embedded-epyc-series) {{< /link >}}
{{< link >}} [AMD EPYC™ Embedded 7001 Series with Extended Availability | AMD](https://www.amd.com/en/processors/embedded-epyc-7001-series) {{< /link >}}
{{< link >}} [AMD EPYC™ Embedded 7002 Series with Extended Availability | AMD](https://www.amd.com/en/processors/embedded-epyc-7002-series) {{< /link >}}

AMD は **EPYC Embedded 7001/7002 シリーズ** を柔軟性、性能、セキュリティと広い可用性を併せ持つとプロセッサとしている。  
そして、7002 シリーズ は *Non-Transparent Bridging (NTB)* 、 *NVMe™ Hot Plug* といった組み込み市場に特化した機能を備える。  
*Non-Transparent Bridging (NTB)* は、PCIeデバイスに対して、複数のホストとなるシステムを 1つのシステムであるかのように見せる機能であり、冗長性を確保することができる。  

モデルナンバーとそれに対応する仕様は、基本現行のサーバー向け EPYC 7001/7002 シリーズ のものと変わらない。ソケット、パッケージもサーバー向けと同じ *SP3* とされている。  
EPYC Embedded 7001 シリーズ には見慣れないモデルナンバーの *755P /740P /735P* が存在するが、それらはサーバー向けの *7551P /7401P /7351P* と対応している。  

サーバー向けとの違いとしては、以下のリンク先より、インターフェイスに USB、SATA の数、*Last Time Buy* の年が明記されている点が見受けられる。  
{{< link >}} [Embedded Processor Specifications | AMD](https://www.amd.com/en/products/specifications/embedded) {{< /link >}}
*Last Time Buy* は既存の **EPYC Embedded 3000 シリーズ** よりも早く、これもサーバー向けと変わらない可能性がある。  
また、**EPYC Embedded 7001 シリーズ** のページから、7001 シリーズ はサーバー向け同様に 2ソケットのシステム構成に対応するが、7002 シリーズ は 1ソケットまでである可能性が考えられる。  

{{< ref >}}

 * [IDF Fall 2009 Preview - Nehalemの組み込み派生型「Jasper Forest」の直前情報 (3) 今回の公開情報(2) | マイナビニュース](https://news.mynavi.jp/article/20090921-idf00/3)

{{< /ref >}}
