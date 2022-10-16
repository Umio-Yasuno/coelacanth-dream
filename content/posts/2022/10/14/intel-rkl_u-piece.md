---
title: "Intel Rocket Lake-U の断片"
date: 2022-10-14T14:15:05+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Rocket_Lake", "CPUID" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は 2021 年に第 11 世代プロセッサとして *Tiger Lake-U (4-Core, Gen12 GT2 96EU)* と *Tiger Lake-H (8-Core, Gen12 GT1 32EU)* がモバイル向けに、*Rocket Lake-S (8-Core, Gen12 GT1 32EU)* がデスクトップ向けにリリースした。  
*Rocket Lake* にはモバイル向けのバリアントも存在したが、これはリリースされることはなかった。  

## Rocket Lake-U {#rkl-u}
先日 <https://01.org/linuxgraphics> で公開された 「Intel® UHD Graphics Open Source Programmer's Reference Manual for the 2021 11th Generation Intel Core™ Processors, Intel Xeon® Processors, and Intel 500 Series Chipsets based on the "Rocket Lake" Platform」 にはわずかながら *Rocket Lake-U* に関する記述が存在する。  
「Volume 2: Configurerations」 の Stepping Info のテーブルに *U61 (SOC Type)* が含まれている。  
他の記述から、"U" はバリアントを、"6" は CPUコア数、"1" は GPU GT を表していると思われる。  

 * [Volume 2: Configurations - intel-gfx-prm-osrc-rkl-vol03-configurations_1.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-rkl-vol03-configurations_1.pdf)

{{< figure src="../intel-rkl-table.webp" caption="引用元: [Volume 2: Configurations - intel-gfx-prm-osrc-rkl-vol03-configurations_1.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-rkl-vol03-configurations_1.pdf)" >}}

Graphics/Media/Display Stepping が *Rocket Lake-S (S81)* よりも早いものであることから、開発自体は *Rocket Lake-U* の方が *Rocket Lake-S* より始まっていた可能性がある。  
SKU と DeviceID の関係をまとめたテーブルも掲載されているが、*Rocket Lake-U (U61)* は無く、*Rocket Lake-S (S81)* のみで構成されている。  

最大 4-Core だが Intel 10nm SF プロセスで製造され、CPU アーキテクチャに *Sunny/Cypress Cove* より進んだ *Willow Cove* を採用し、GPU も最大 Gen12 GT2 96EU を持つ *Tiger Lake-U* と比較すると、  
最大 6-Core を持つが、14nm プロセス、CPU アーキテクチャは *Cypress Cove*、GPU は最大 Gen12 GT1 32EU という *Rocket Lake-U* は、CPU/GPU性能と電力効率の面で差別化が厳しかったと思われる。  
*Tiger Lake-U* より上の消費電力帯向けに *Tiger Lake-H* が後にリリースされたことも考えると、詳細な時期は不明だが、*Rocket Lake-U* をリリースする意義が薄いとされ、キャンセルされたものと思われる。  

## CPUID Model: 0xA8 {#06_a8h}
x86_64 CPU には識別子として `Family, Model, Stepping` が割り当てられており、`CPUID` 命令経由で確認できるようになっている。  
*Rocket Lake-S* は `CPUID Model` に `Model: 0xA7 (167)` が、そしてモバイル向けである *Rocket Lake-U* には `Model: 0xA8 (168)` が割り当てられている。  

そして Intel が公開している「Data Operand Independent Timing ISA Guidance」では、`06_A8H (Family: 0x6, Model: 0xA8)` が割り当てられているプロセッサとして、**Intel Xeon W-1300 Processor Family (Rocket Lake)** が挙げられている。  

 * [Data Operand Independent Timing ISA Guidance](https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/best-practices/data-operand-independent-timing-isa-guidance.html#inpage-nav-5)

これを鵜呑みにすれば、キャンセルされたと思われていた *Rocket Lake-U* がワークステーション向けにひっそりとリリースされていた、という面白そうな話が浮かぶのだが、これは先に挙げた SKU と DeviceID の関係をまとめたテーブルと食い違っている。  
また **Intel Xeon W-1300** は 8-Core と 6-Core の SKU があり、6-Core の *Rocket Lake-U* だけですべてをカバーすることはできない。「Data Operand Independent Timing ISA Guidance」の記述が一部が間違っている可能性が高い。  
**Intel Xeon W-1350/W-1350P** が 6-Core SKU であり、それらで *Rocket Lake-U* がベースになっている可能性もあるが、それらの `CPUID` 情報を確認できるようなベンチマーク結果やログを見つけることができなかった。  
しかし、他に *Rocket Lake-U* に触れていて、そして記述が一致するような公式ドキュメントもまた見つけられなかった。**Intel Xeon W-1300** も基本 *Rocket Lake-S* ベースとされている。  
そのため、完全には否定できないが *Rocket Lake-U* が一部の SKU に使われている可能性は限りなく低いように思う。  

{{< ref >}}
 * [Intel® Core™ Processors Technical Resources](https://www.intel.co.jp/content/www/jp/ja/products/docs/processors/core/core-technical-resources.html)
 * [Intel® Xeon® W-1300 Product Brief](https://www.intel.com/content/www/us/en/products/docs/processors/xeon/xeon-w-1300-processors-brief.html)
 * [Data Operand Independent Timing ISA Guidance](https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/best-practices/data-operand-independent-timing-isa-guidance.html)
 * [2021 INTEL(R) PROCESSORS BASED ON THE Rocket Lake PLATFORM | 01.org](https://01.org/node/37341)
{{< /ref >}}
