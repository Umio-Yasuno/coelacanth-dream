---
title: "メモリコントローラー 1基 (64-bit) の Alder Lake-N"
date: 2022-02-09T20:12:43+09:00
draft: false
tags: [ "Alder_Lake", "Coreboot", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

*Alder Lake-N* では他バリアント (*Alder Lake-S/P/M*) と異なり、メモリコントローラは 1基 (64-bit) のみを持つ。  
そうしたパッチが Intel のシステムエンジニア [Krishna Prasad Bhat](https://in.linkedin.com/in/krishna-prasad-bhat-b71a6120) 氏より、Coreboot に投稿された。  

* [mb/google/nissa: Set half_populated true (I414e5dc8) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61764/5)
* [soc/intel/alderlake: Add Alder Lake N memory data bus width (I71fd30ca) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61765/1)

{{< bq cite="[soc/intel/alderlake: Add Alder Lake N memory data bus width (I71fd30ca) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61765/1)" >}}
 > 		soc/intel/alderlake: Add Alder Lake N memory data bus width
 >
 > 		Alder Lake N has single memory controller with 64-bit bus width.
 > 		Accordingly, number of MRC Channels also gets reduced. Add checks to
 > 		populate meminit data only for required channels for Alder Lake N.
{{< /bq >}}

パッチでは、*Alder Lake-N* の場合、メモリチャネルに関する部分を半分にしている。そのため、メモリバス幅は 64-bit に削減されたが、DDR4/LPDDR4/DDR5/LPDDR5 に対応する点は他バリアントと変わらない可能性がある。[^mem-type]  

[^mem-type]: […/meminit.c · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61765/1/src/soc/intel/alderlake/meminit.c#70)
[^jsl-mem]: [Block Diagram - 005 - ID:633935 | Intel® Pentium® Silver and Intel® Celeron® Processors Datasheet, Volume 1](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/servers/platforms/intel-pentium-silver-and-intel-celeron-processors-datasheet-volume-1-of-2/005/block-diagram/)

現世代のモバイル向け Atomプロセッサ *Jasper Lake* はメモリインターフェイスが DDR4/LPDDR4 128-bit という仕様であるため、それと比較すると、対応メモリは拡張されたが、メモリバス幅は半分に減った形となる。  
最近の Intel CPU で、メモリバス幅が 64-bit だったものには *Lakefield* がいるが、それと同程度の SDP/TDP (7W?) を想定しているのかもしれない。  

他バリアントと比較すると、*Alder Lake-N* は間違いなく最小のバリアントと言える。  
*Alder Lake-N* は PCIe I/O を CPU側に持たず、PCH側に 9-Lane 持ち、また Core系/Big/Performance Core を搭載せず、Atom系/Small/Efficient Core のみの構成になるとされている。  
CPU部の規模については、ブートログから 8-Thread(s) あることが確認されている。  
GPU は Gen12.2、ディスプレイ部は *Alder Lake-P/M* 同様に Gen13 (XE_LPD) となるが、規模は GT1 32EU が上限になる。  
{{< link >}} [CPU 8-Thread、GPU 32EU を持つ Alder Lake-N | Coelacanth's Dream](/posts/2022/02/04/adl_n-8thread/) {{< /link >}}
そしてメモリバス幅が 64-bit となっている。  

*Alder Lake-N* を搭載する Chromebookボードのサポートも進められており、モバイル向けとして登場することはほぼ確実だと言える。  
CPU 8-Thread(s)、GPU GT1 32EU、DDR4/LPDDR4/DDR5/LPDDR5 64-bit、PCIe 9-Lane という仕様から、*Alder Lake-P/M* よりも省電力、低コストな製品が期待される。  
{{< link >}} [Alder Lake-N を搭載する Chromebookボード 「Nissa, Nivviks, Nereid」 | Coelacanth's Dream](/posts/2022/01/12/adl_n-chromebook-board/) {{< /link >}}

| Alder Lake | -S | -P | -M | -N |
| :-- | :--: | :--: | :--: | :--: |
| Variant (CPUID Model) | Desktop (0x97) | Mobile (0x9A) | Mobile (0x9A) | Mobile/Embedded? (0xBE) |
| CPU (big + small) | 8 + 8 | 6 + 8 | 2 + 8 | 0 + 8? |
| GPU (Gen12.2) | GT1 32EU | GT2 96EU | GT2 96EU | GT1 32EU |
| Display ver | Gen12 | Gen13 (XE_LPD) | Gen13 (XE_LPD) | Gen13 (XE_LPD) |
| CPU PCIe | Gen5 8L x2,<br>Gen4 4L x2 | Gen5 8L,<br>Gen4 4L x2 | Gen4 4L? | N/A |
| PCH PCIe | ? | ? | ? | 9-Lane |
| Memory Bus width | 128-bit |128-bit |128-bit | 64-bit |
