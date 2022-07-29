---
title: "Alder Lake-N のコア数と TDP (PL1), PL2"
date: 2022-05-19T02:26:02+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Alder_Lake", "Coreboot" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Vidya Gopalakrishnan 氏より、Coreboot に *Alder Lake-N* SKU の TDP, Power Limit の設定値を追加するパッチが投稿、公開されている。  
パッチでは *Alder Lake-N* の各設定値以外に、コア数に関する記述も見られる。  

 * [soc/intel/alderlake: add power limits for Alder Lake-N SKUs (I24c18a27) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/64472/1)
 * [soc/intel/alderlake: Configure the SKU specific parameters for VR domains (I3d6ae203) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/63369/6)

*Alder Lake-N* は以下の引用部から、*Gracemont (Small)* コアのみの構成であり、コア数は 2/4/8 の 3種類が用意されていると思われる。TDP (PL1) は 2/4-Core が 6W、8-Core が 15W。  
以下引用部では、*Alder Lake* のバリアント、コア構成と GPU GT、TDP (PL1) を次のようなフォーマットで記述している。 `ADL_{M,N,P}_{BIG}{SMALL}{GPU_GT}_{TDP}W_CORE`  

*Alder Lake-N* が *Alder Lake-S/P/M* に搭載されている *Golden Cove (Big)* コアを持たず *Gracemont (Small)* コアのみの構成、最大 8-Core/8-Thread、GPU は Gen12 GT1 (32 EU)、といった情報は、これまでに公開されてきたパッチやログの情報と一致する。  
{{< link >}} [CPU 8-Thread、GPU 32EU を持つ Alder Lake-N | Coelacanth's Dream](/posts/2022/02/04/adl_n-8thread/) {{< /link >}}
ChromeOS に関するパッチのレビューが行われる Chromium Gerrit では、*Alder Lake-N (Family: 0x6, Model: 0xBE)* を *Gracemont* とするパッチも公開されている。  

[^gracemont]: [Updating INTEL_UARCH_TABLE with Alder Lake-N platform info (I9a27bc1e) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/autotest/+/3590236)

*Alder Lake-N* 2/4-Core SKU の TDP (PL1) 6W というのは、TDP (PL1) 9W をサポートする *Alder Lake-M (2+8+2, 2+4+2)* SKU より省電力、ファンレス構成等の製品をカバーするためのものと思われる。  

 > 		 	ADL_M_282_15W_CORE,
 > 		 	ADL_M_242_CORE,
 > 		 	ADL_P_442_45W_CORE,
 > 		+  ADL_N_081_15W_CORE,
 > 		+  ADL_N_041_6W_CORE,
 > 		+  ADL_N_021_6W_CORE,
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/64472/1/src/soc/intel/alderlake/chip.h#33> {{< /quote >}}

Coreboot でサポートしている Intel ADLRVP ボードでは、*Alder Lake-M (2+8+2)* の PL1/PL2 設定値は以下のようになっている。  
Big コアを持たない *Alder Lake-N* で、*Alder Lake-M (2+8+2)* と同じ TDP (PL1) 15W の設定が用意されているのは少し意外かもしれない。なお PL2 は *Alder Lake-N* の方が 10W 低い 35W 設定となる。  
また、別のパッチの内容から、*Alder Lake-N* 8-Core には TDP (PL1) 7W の設定もあるようだ。  

 > 		/*
 > 		 * VR Configurations for IA and GT domains for ADL-N SKU's.
 > 		 * Per doc#646929 ADL N Platform Design Guide -> Power_Map_Rev1p0
 > 		 *
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 * |      SKU       | Setting   | AC LL | DC LL | ICC MAX | TDC Current | TDC Time |
 > 		 * |                |           |(mOhms)|(mOhms)|   (A)   |     (A)     |   (msec) |
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 * | ADL-N 081(15W) |    IA     |  4.7  |  4.7  |    53   |      22     |  28000   |
 > 		 * +                +-----------+-------+-------+---------+-------------+----------+
 > 		 * |                |    GT     |  6.5  |  6.5  |    29   |      22     |  28000   |
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 * | ADL-N 081(7W)  |    IA     |  5.0  |  5.0  |    37   |      14     |  28000   |
 > 		 * +                +-----------+-------+-------+---------+-------------+----------+
 > 		 * |                |    GT     |  6.5  |  6.5  |    29   |      14     |  28000   |
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 * | ADL-N 041(6W)  |    IA     |  5.0  |  5.0  |    37   |      12     |  28000   |
 > 		 * +  Pentium       +-----------+-------+-------+---------+-------------+----------+
 > 		 * |                |    GT     |  6.5  |  6.5  |    29   |      12     |  28000   |
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 * | ADL-N 041(6W)  |    IA     |  5.0  |  5.0  |    37   |      12     |  28000   |
 > 		 * +  Celeron       +-----------+-------+-------+---------+-------------+----------+
 > 		 * |                |    GT     |  6.5  |  6.5  |    26   |      12     |  28000   |
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 * | ADL-N 021(6W)  |    IA     |  5.0  |  5.0  |    27   |      10     |  28000   |
 > 		 * +                +-----------+-------+-------+---------+-------------+----------+
 > 		 * |                |    GT     |  6.5  |  6.5  |    23   |      10     |  28000   |
 > 		 * +----------------+-----------+-------+-------+---------+-------------+----------+
 > 		 */
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/63369/6/src/soc/intel/alderlake/vr_config.c> {{< /quote >}}

*Alder Lake-N* 8-Core の PL4 は 83W、2/4-Core は 78W となっているが、PL4 はあくまでもパッケージレベルの最大電力制限であり、電源アダプタやバッテリーの許容範囲となる。電力管理機能はそれを超えないよう、先制して周波数を制限、調整する。  

Atom (Small) 系プロセッサとして *Alder Lake-N* を *Japser Lake (Tremont, 10nm)* と較べると、Intel JSLRVP ボードでは TDP (PL1) 6W、PL2 20W の設定となっており[^jslrvp]、*Alder Lake-N* 2/4-Core は TDP (PL1) は同じだが、PL2 は 5W 高いと見ることができる。  

[^jslrvp]: <https://review.coreboot.org/plugins/gitiles/coreboot/+/refs/changes/72/64472/1/src/mainboard/intel/jasperlake_rvp/variants/jslrvp/devicetree.cb#126>

 > 			register "power_limits_config[ADL_M_282_12W_CORE]" = "{
 > 				.tdp_pl1_override = 12,
 > 				.tdp_pl2_override = 35,
 > 			}"
 > 			register "power_limits_config[ADL_M_282_15W_CORE]" = "{
 > 				.tdp_pl1_override = 15,
 > 				.tdp_pl2_override = 45,
 > 			}"
 > 			register "power_limits_config[ADL_M_242_CORE]" = "{
 > 				.tdp_pl1_override = 9,
 > 				.tdp_pl2_override = 30,
 > 				.tdp_pl4 = 68,
 > 			}"
 >
 > {{< quote >}} <https://review.coreboot.org/plugins/gitiles/coreboot/+/refs/changes/72/64472/1/src/soc/intel/alderlake/chipset.cb#35> {{< /quote >}}
 >
 > 		+	register "power_limits_config[ADL_N_081_15W_CORE]" = "{
 > 		+		.tdp_pl1_override = 15,
 > 		+		.tdp_pl2_override = 35,
 > 		+		.tdp_pl4 = 83,
 > 		+	}"
 > 		+
 > 		+	register "power_limits_config[ADL_N_041_6W_CORE]" = "{
 > 		+		.tdp_pl1_override = 6,
 > 		+		.tdp_pl2_override = 25,
 > 		+		.tdp_pl4 = 78,
 > 		+	}"
 > 		+
 > 		+	register "power_limits_config[ADL_N_021_6W_CORE]" = "{
 > 		+		.tdp_pl1_override = 6,
 > 		+		.tdp_pl2_override = 25,
 > 		+		.tdp_pl4 = 78,
 > 		+	}"
 > 		+
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/64472/1/src/soc/intel/alderlake/chipset.cb#51> {{< /quote >}}

余談だが、Intel が公開するプロセッサのドキュメント中において、以前は TDP (PL1) のみ具体的な数値が記載され、PL2 については PL1 x1.25 が推奨値とされながら、実際の設定は PL1 x3 近くとなっていたのが、  
*Alder Lake* の世代からは Coreboot 中の設定と同様の値、実際の設定値がドキュメントに記載されるようになった。  
今までがドキュメント中でぼかされていた状態だったとはいえ、ユーザーに対して公開される情報が増えたことは素直に喜ばしい。  

 * [Alder Lake P: Overview and Technical Documentation](https://www.intel.com/content/www/us/en/products/platforms/details/alder-lake-p.html)
 * [12th Gen Intel® Core™ Processors Product Brief](https://www.intel.com/content/www/us/en/products/docs/processors/core/12th-gen-core-mobile-processors-brief.html)
 * [Processor Line Thermal and Power - 006 - ID:655258 | Core™ Processors](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/client/platforms/alder-lake-desktop/12th-generation-intel-core-processors-datasheet-volume-1-of-2/006/processor-line-thermal-and-power/)

| Alder Lake | -S | -P | -M | -N |
| :-- | :--: | :--: | :--: | :--: |
| Variant (CPUID Model) | Desktop (0x97) | Mobile (0x9A) | Mobile (0x9A) | Mobile (0xBE) |
| CPU (big + small) | 8 + 8 | 6 + 8 | 2 + 8 | 0 + 8 |
| GPU (Gen12.2) | GT1 32EU | GT2 96EU | GT2 96EU | GT1 32EU |
| Display ver | Gen12 | Gen13 (XE_LPD) | Gen13 (XE_LPD) | Gen13 (XE_LPD) |
| CPU PCIe | Gen5 8L x2,<br>Gen4 4L x2 | Gen5 8L,<br>Gen4 4L x2 | Gen4 4L? | N/A |
| PCH PCIe | ? | ? | ? | 9-Lane |
| Memory Bus width | 128-bit |128-bit |128-bit | 64-bit |
