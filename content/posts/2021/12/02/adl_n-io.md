---
title: "Alder Lake-N の PCIe I/O 構成"
date: 2021-12-02T01:37:55+09:00
draft: false
tags: [ "Alder_Lake", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

CPUID、DeviceID の追加に続いて、Intel の [Usha P](https://review.coreboot.org/q/owner:usha.p%2540intel.com) 氏により、Coreboot に *Alder Lake-N* の PCIe I/O 構成情報を追加するパッチが投稿、公開されている。  
{{< link >}} [N バリアントも存在する Alder Lake | Coelacanth's Dream](/posts/2021/11/16/coreboot-intel-adl_n/) {{< /link >}}

 * [soc/intel/alderlake: Add support for ADL-N PCH (I7ebbcdcd) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59752/7/)

コミットメッセージとパッチ内容によれば、*Alder Lake-N* では CPU側の PCIe root ports を持たず、PCH側は 12 PCIe root ports を持つが、PCIe clock sources、clock request signals は 5基となっているため、有効可能なのは 5-Ports までとされる。PCIe Lane 数は最大 9-Lanes だとコメントで触れられている。  

 > 		soc/intel/alderlake: Add support for ADL-N PCH
 > 		
 > 		Introduce the `SOC_INTEL_ALDERLAKE_PCH_N` Kconfig option and
 > 		use it to specify the correct amount of PCIe I/O.
 > 		
 > 		Document number 645550 indicates that Alder Lake-N has
 > 		12 PCH root ports and no CPU root ports.
 > 		
 > 		Document number 645548 indicates ADL-N has 5 clock sources
 > 		and 5 clock request signals.
 >
 > {{< quote >}} [soc/intel/alderlake: Add support for ADL-N PCH (I7ebbcdcd) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59752/8) {{< /quote >}}

### Jasper Lake と近い構成の Alder Lake-N {#jsl}

CPU側 PCIe は、*Alder Lake-S (Desktop)* では Gen5 8-Lanes x2・Gen4 4-Lanes x2、*Alder Lake-P (Mobile)* では Gen5 8-Lanes・Gen4 4-Lanes x2、*Alder Lake-M (Mobile)* では Gen4 4-Lanes をサポートするとされている。  
*Alder Lake-N* は CPU側 PCIe を持たない構成となるが、これは現世代の Atomプロセッサ [Elkhart/Jasper Lake](/tags/jasper_lake/) ([Tremont](/tags/tremont/)) の構成に近い。  

*Elkhart Lake* は組み込み向け、*Jasper Lake* はモバイル向けという違いはあるが、Compute die と PCH die の 2つのダイを 1つのパッケージに実装するという構成は共通する。  
*Jasper Lake* では、Compute die に Tremont 4-Core CPU、Gen11 32EU GPU、IPU (Image Processing Unit)、GNA、FIVR、メモリコントローラ/インターフェイス、ディスプレイコントローラ/インターフェイスが集積されており、*Ice Lake* と同じ Intel 10+nmプロセスで製造される。  
PCH die にはその他各種 I/O が集積され、PCIe I/O は Gen3、最大 5-ports、最大 8-Lanes をサポートしている。製造プロセスは Intel 14nm。[^jsl-block-diagram]  
CPU/Compute die側に PCIeインターフェイスを持たない、PCH die側が最大 5-ports という点が *Jasper Lake* と *Alder Lake-N* で共通する。  
PCIe Lane数も *Jasper Lake* が 8-Lanes、*Alder Lake-N* が 9-Lanes とほとんど同じ。  

[^jsl-block-diagram]: [Block Diagram - 005 - ID:633935 | Intel® Pentium® Silver and Intel® Celeron® Processors Datasheet, Volume 1](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/servers/platforms/intel-pentium-silver-and-intel-celeron-processors-datasheet-volume-1-of-2/005/block-diagram/)

*Alder Lake-N* のサポート追加に向けた他パッチから、*Alder Lake-N* は他バリアントと異なりハイブリッドコア構成を採らず、 *Gracemont (Atom/small)* のみの CPU構成になるとされる。  
{{< link >}} [Alder Lake-N は Atom系コアのみの構成か | Coelacanth's Dream](/posts/2021/11/25/adl_n-atom-only/) {{< /link >}}
*Jasper Lake* は TDP 6W・10W・15W の SKU が、*Elkhart Lake* は TDP 4.5-10W の SKU がラインナップされているが、*Alder Lake-N* も近い TDP (PL1) 設定になっているのではないかと思われる。  

*Tiger Lake UP4* の後継になると目される *Alder Lake-M* は、現在 3種類の構成が Coreboot でサポートされている。  
一番 PowerLimit が小さい 2+4+2 (big, small, GPU GT) 構成で PL1: 9W、PL2: 30W。*Alder Lake-N* はそれより PowerLimit を抑えた、低消費電力なバリアントとなることが考えられる。[^adl_m-power]  
{{< link >}} [Alder Lake-P/M の PL4 参考値と Brya の設定値 | Coelacanth's Dream](/posts/2021/08/12/intel-adl-pl4/) {{< /link >}}

[^adl_m-power]: [coreboot/chipset.cb at 0c54461cf99010d9ebeae869f0a486b0268ec860 · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/0c54461cf99010d9ebeae869f0a486b0268ec860/src/soc/intel/alderlake/chipset.cb#L29-L43)

*Elkhart/Jasper Lake* では GPU部は Gen11 32EU が最大規模の構成だった。*Alder Lake-N* は Gen12 GT1、最大 EU 32基である可能性が出てきており、EU数で見れば同規模だが、アーキテクチャ更新による性能向上は期待できる。  
CPU部においても、*Elkhart/Jasper Lake* は最大 4-Core、L2キャッシュを共有するクラスタは 1基となる構成だったが、*Alder Lake-S/P/M* では *Gracemont (Atom/small)* コアを最大 8-Core、クラスタ 2基の構成であるため、*Alder Lake-N* もそれを踏襲することも考えられる。  
だが CPUクラスタあたり 4-Core で L2キャッシュを共有する構成は *Gracemont* でも引き継いでいるため、*Alder Lake-N* ではクラスタ 1基の採ることも可能。  
*Jasper Lake* の後継とするならば、クラスタ 1基 (4-Core) 構成の可能性が高いだろう。  

*Alder Lake-N* がモバイル向けにもリリースされるかは、*Alder Lake-P/M* とは異なる Model ID が割り当てられていることから、その可能性が低いように思える。  
だが、元々 Atom系コアのみで構成されたプロセッサは *Elkhart/Jasper Lake* で最後になるという話があったが、それに反して *Alder Lake-N* の存在が浮かび上がってきたことから何らかの方針転換があったと思われ、今後どう展開されるかは判然としない。[^only-atom]

[^only-atom]: [ASCII.jp：最後のAtomとなるChromebook向けプロセッサーのJasper Lake　インテル CPUロードマップ (3/3)](https://ascii.jp/elem/000/004/040/4040489/3/)

| Alder Lake | -S | -P | -M | -N |
| :-- | :--: | :--: | :--: | :--: |
| Variant (Model) | Desktop (0x97) | Mobile (0x9A) | Mobile (0x9A) | Embedded? (0xBE) |
| CPU (big + small) | 8 + 8 | 6 + 8 | 2 + 8 | 0 + ? |
| GPU (Gen12.2) | GT1 32EU | GT2 96EU | GT2 96EU | GT1 32EU? |
| CPU PCIe | Gen5 8L x2,<br>Gen4 4L x2 | Gen5 8L,<br>Gen4 4L x2 | Gen4 4L ? | N/A |
| TDP (PL1) | - | 15-45W | 9-15W ? | |

{{< ref >}}
 * [topic:ADL_N_UPSTREAM · Gerrit Code Review](https://review.coreboot.org/q/topic:ADL_N_UPSTREAM)
 * [soc/intel/alderlake: Refactor SoC code to maintain CPU and PCH PCIE RPs (I92123450) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/49136/8)
{{< /ref >}}
