---
title: "Intel Raptor Lake-P の PCIe I/O 構成"
date: 2022-10-09T12:53:43+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Raptor_Lake", "Alder_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

*Raptor Lake-S* は *Alder Lake-S* と CPU部を除いて設計がほとんど共通しており、I/O 部も基本同じとされているが、モバイル向けの *Raptor Lake-P* は *Alder Lake-P* から変更される可能性がある。  

*Alder Lake* 世代では *Alder Lake-S/P/M* でそれぞれ CPU 側の PCIe I/O 構成が異なり、デスクトップ向けの *Alder Lake-S (8P+8E)* では 2x Gen5 8-Lane + 1x Gen4 4-Lane、*Alder Lake-P (6+8)* では 1x Gen5 8-Lane + 2x Gen4 4-Lane、*Alder Lake-M (2+8)* では 1x Gen4 4-Lane という構成になっている。  
現状 *Alder Lake-P* の 1x Gen5 8-Lane は *H Series* でのみ有効とされているが、Gen5 ではなく Gen4 での利用となっている。これはドキュメント中では TTM (time-to-market, 製品化に掛かる期間) のためだとしている。[^ttm]  
搭載するデバイスがほとんど固定されているモバイル向けでは、PCIe Gen5 対応デバイスが少ない中わざわざ Gen5 認証を取るコストに対してリターンが見合わないというのもあるだろう。  
また、CPU部が Atom系 (E-Core) *Gracemont* のみの *Alder Lake-N* は CPU側 PCIe I/O を持たない。  

[^ttm]: [Introduction - 009 - ID:655258 | 12th Generation Intel® Core™ Processors](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/client/platforms/alder-lake-desktop/12th-generation-intel-core-processors-datasheet-volume-1-of-2/introduction/)

先日、2022/09/28 付で「13th Generation Intel® Core™ Processors Datasheet, Volume 1 of 2」が公開された。[^13th]  
そのドキュメントによれば、*Raptor Lake-S* は PCIe Root Port (RP) /Root Complex (RC) に割り当てられた DeviceID は違うが、2x Gen5 4-Lane + 1x Gen4 4-Lane と *Alder Lake-S* と同様の構成を採っている。  
PCIe RC と DeviceID はそれぞれ `PCIe RC 010 G5 (DeviceID: 0xA70D), PCIe RC 011 G5 (DeviceID: 0xA72D), PCIe RC 060 (x4) G4 (DeviceID: 0xA74D)` となる。  
*Raptor Lake-P* はまだ SKU が正式発表されていないため、同様のドキュメントは公開されていないが、*Raptor Lake-P* のサポートが進められている Coreboot では各種 DeviceID の情報が追加されている。  
そして *Raptor Lake-P* の PCIe RP/RC には *Raptor Lake-S* と同じ DeviceID が割り当てられている。  

[^13th]: [13th Generation Intel® Core™ Processors Datasheet, Volume 1 of 2](https://www.intel.com/content/www/us/en/content-details/743844/13th-generation-intel-core-processors-datasheet-volume-1-of-2.html)

 >         @@ -3434,6 +3463,10 @@
 >          #define PCI_DID_INTEL_MTL_IOE_P_PCIE_RP11		0x7ecb
 >          #define PCI_DID_INTEL_MTL_IOE_P_PCIE_RP12		0x7ecc
 >          
 >         +#define PCI_DID_INTEL_RPL_P_PCIE_RP1			0xa74d
 >         +#define PCI_DID_INTEL_RPL_P_PCIE_RP2			0xa70d
 >         +#define PCI_DID_INTEL_RPL_P_PCIE_RP3			0xa72d
 >         +
 >
 > {{< quote >}} [soc/intel: Add Raptor Lake device IDs · coreboot/coreboot@a15b25f](https://github.com/coreboot/coreboot/commit/a15b25f6fd3b121913508bf6b603856d5026be2c) {{< /quote >}}

このことから、*Raptor Lake-P* では *Alder Lake-S/Raptor Lake-S* と同じ PCIe I/O 構成を採っていると考えられる。  
*Raptor Lake-P* と *Alder Lake-P* を比較した時、*Raptor Lake-P* では Gen5 Lane 数が増えると同時に全体の Lane 数も増えたと見ることができる。  
*Alder Lake-S/Raptor Lake-S* と同様の構成とすることで、開発期間も短縮することができる。  

しかし、*Raptor Lake-P* では *H Series* だけでなく *U Series* でも追加の PCIe Lane が有効化されるのか、また *Alder Lake-P* では有効化されなかった Gen5 での利用が可能になるかは不明である。  

| | ADL-S | ADL-P | ADL-M | ADL-N | RPL-S | RPL-P |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: |
| Variant (CPUID Model) | Desktop (0x97) | Mobile (0x9A) | Mobile (0x9A) | Mobile (0xBE) | Desktop (0xB7) | Mobile (0xBA) |
| CPU (big + small) | 8 + 8 | 6 + 8 | 2 + 8 | 0 + 8 | 8 + 16 | 6 + 8? |
| GPU (Gen12.2) | GT1 32EU | GT2 96EU | GT2 96EU | GT1 32EU | GT1 32EU | GT2 96EU |
| Display ver | Gen12 | Gen13 (XE_LPD) | Gen13 (XE_LPD) | Gen13 (XE_LPD) | Gen12 | Gen13 (XE_LPD) |K
| CPU PCIe | 2x Gen5 8L,<br>1x Gen4 4L | 1x Gen5 8L,<br>2x Gen4 4L | 1x Gen4 4L | N/A | 2x Gen5 8L,<br>1x Gen4 4L | 2x Gen5 8L?,<br>1x Gen4 4L? |

{{< ref >}}
 * [【笠原一輝のユビキタス情報局】Alder Lake/Raptor Lakeの「高性能の秘密」はPコアに内蔵されたマイクロコントローラにあった！ - PC Watch](https://pc.watch.impress.co.jp/docs/column/ubiq/1442699.html)
 * [13th Generation Intel® Core™ Processors Datasheet, Volume 1 of 2](https://www.intel.com/content/www/us/en/content-details/743844/13th-generation-intel-core-processors-datasheet-volume-1-of-2.html)
 * [soc/intel: Add Raptor Lake device IDs · coreboot/coreboot@a15b25f](https://github.com/coreboot/coreboot/commit/a15b25f6fd3b121913508bf6b603856d5026be2c)
 * [soc/intel/alderlake: Add new pcie5 alias for raptorlake · coreboot/coreboot@9e86b71](https://github.com/coreboot/coreboot/commit/9e86b71e7936fd17a2b6d2c15ccd81442f21c576)
 * [Alder Lake P: Overview and Technical Documentation](https://www.intel.com/content/www/us/en/products/platforms/details/alder-lake-p.html)
{{< /ref >}}
