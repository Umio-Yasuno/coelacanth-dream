---
title: "Alder Lake-P/M のステッピング、PL1/PL2 参考値"
date: 2021-05-26T17:46:07+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Intel", "CPU" ]
noindex: false
# summary: ""
---

最近 Coreboot に投稿された *Alder Lake-P/M* サポートに向けたパッチから得られる情報をまとめた記事。  

{{< pindex >}}
 * [Alder Lake-P/M は Stepping で区別されるか](#adl-step)
 * [Alder Lake-P/M Power Limit](#adl-power)
    * [Alder Lake-P PL1/PL2](#adl_p-pl)
    * [Alder Lake-M PL1/PL2](#adl_m-pl)
 * [TGL/JSL/ADL](#table)
{{< /pindex >}}

## Alder Lake-P/M は Stepping で区別されるか {#adl-step}

 * [soc/intel/alderlake: Update CPU and IGD Device IDs (I2759a41a) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/54211/3/)

Coreboot に *Alder Lake-P B0* の `Family/Model/Stepping` による判定に使われる CPUID を追加するパッチが Maulik V Vaghela 氏により投稿された。  
だが、 *A0 ステッピング* (`0x90670`) から値を 1 追加した `0x90671` ではなく、`0x90672` が *B0 ステッピング* に割り当てられている。  
CPUID の末尾 1桁、正確に言えば 4-bit 分の領域はその CPU のステッピング/リビジョンを表現するのに使われている。  

 > 		#define CPUID_ALDERLAKE_S_A0	0x90670
 > 		#define CPUID_ALDERLAKE_P_A0	0x906a0
 > 		#define CPUID_ALDERLAKE_P_B0	0x906a2
 > 		#define CPUID_ALDERLAKE_M_A0	0x906a1
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/54211/3/src/soc/intel/common/block/include/intelblocks/mp_init.h> {{< /quote >}}

`0x90672` を割り当てたのは `0x90671` が既に *Alder Lake-M A0* に使われているからだと思われるが、同時にマーケットセグメント上では同じ `ALDERLAKE_L (Family: 0x6, Model: 0x9A [154])` に属す *Alder Lake-P/M* だが、ステッピングで区別されることを意味する。  
{{< link >}} [ALDERLAKE_L と Alder Lake-P/M | Coelacanth's Dream](/posts/2021/02/09/alderlake_l/) {{< /link >}}
ステッピングで区別すること自体は過去にも行われており、 *Kaby Lake(-U)* に対する *Amber Lake(-Y) / Coffee Lake(-U) / Whiskey Lake(-U)* 、*Skylake server* に対する *Cascade Lake / Cooper Lake* 等がそうだ。最近になってそれらをきちんと整理するためのコメントを追加するパッチが投稿されている。  
{{< link >}} [Intel Kaby Lake 周りの CPU Stepping を整理するパッチ | Coelacanth's Dream](/posts/2021/04/09/intel-kbl-complex/) {{< /link >}}
前例から考えれば、 *xxxx Lake* のような名前を無闇に増やすのではなく、基本を *Alder Lake* で統一し、誰にとっても関係性をわかりやすくしたと取れなくもない。  

## Alder Lake-P/M Power Limit {#adl-power}

あくまでも **Alder Lake RVP (Reference Validation Platform)** と、*Alder Lake-P* を搭載する Chromebookボード **Brya** での例とされるが、Coreboot の *Alder Lake* に関連した部分にその PL1/PL2 の値が追加された。  
例ではあるが、分離された別のパッチでは SKU ごとに異なる PCI ID を持つ、System Agent の部分で適用する Power Limit 設定を変えているため、実質 SKU のデフォルト設定だと言える。[^sa-pci-id]  

[^sa-pci-id]: [soc/intel/adl: Add SKU specific power limits support (Ic1676e2b) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/54676/4)

 * [mb/adlrvp: Override power limits with SKU specific values (I80ac9512) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/54926/1/)
 * [soc/intel/adl: Add SKU specific power limits support (Ic1676e2b) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/54676/4)
 * [[WIP] mb/intel/adlrvp: Enable DPTF functionality for ADL-M RVP (If7763ee2) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/54491/4)

パッチは [Sumeet R Pawnikar](https://in.linkedin.com/in/sumeet-pawnikar) 氏、Anil Kumar K 氏によって作成、投稿されている。  

### Alder Lake-P PL1/PL2 {#adl_p-pl}

 > ##### Alder Lake-P PL1/PL2 (adlrvp)
 >
 > 		On adlrvp (482):
 > 		 CPU PL1 = 28 Watts
 > 		 CPU PL2 = 64 Watts
 > 		On adlrvp (682):
 > 		 CPU PL1 = 45 Watts
 > 		 CPU PL2 = 115 Watts
 > 		On brya (282):
 > 		 CPU PL1 = 15 Watts
 > 		 CPU PL2 = 55 Watts
 >
 > {{< quote >}} [mb/adlrvp: Override power limits with SKU specific values (I80ac9512) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/54926/1/) {{< /quote >}}
 >
 > 			register "power_limits_config[ADL_P_POWER_LIMITS_282_CORE]" = "{
 > 				.tdp_pl1_override = 15,
 > 				.tdp_pl2_override = 55,
 > 			}"
 > 			register "power_limits_config[ADL_P_POWER_LIMITS_482_CORE]" = "{
 > 				.tdp_pl1_override = 28,
 > 				.tdp_pl2_override = 64,
 > 			}"
 > 			register "power_limits_config[ADL_P_POWER_LIMITS_682_CORE]" = "{
 > 				.tdp_pl1_override = 45,
 > 				.tdp_pl2_override = 115,
 > 			}"
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/54926/1/src/mainboard/intel/adlrvp/devicetree.cb> {{< /quote >}}

*Alder Lake-P* においては恐らく *Golden Cove (Big/Core)* コア数で分かれており、2 (Big) + 8 (Small) + GT2 は PL1: 15W/PL2: 55W、4 + 8 + GT2 は PL1: 28W/PL2: 64W、4 + 8 + GT2 は PL1: 45W/PL2: 115W となっている。  
この値をどう見るかはそれぞれだが、 *Tiger Lake UP3* では 2-Core が PL1: 15W/PL2: 38W、 4-Core が PL1: 15W/PL2: 60W となっており[^tgl_up3-pl]、*Alder Lake-P (2+8+2)* の PL2: 55W というのは *Golden Cove* 2-Core に加え *Gracemont (Small/Atom)* 8-Core も搭載していることを考えれば *Tiger Lake UP3* とそう変わらないように思える。  
*Alder Lake-P (4+8+2)* の PL1: 28W/PL2: 64W については、Coreboot に PL1: 28W 時の PL2 は記述されていないが、*Tiger Lake UP3 28W* と同じ設定となり、ここでは設定と Bigコアの数が *Tiger Lake* と一致する。[^tgl-28w]  

 > ##### Tiger Lake UP3 PL1/PL2
 > 			# Add PL1 and PL2 values
 > 			register "power_limits_config[POWER_LIMITS_U_2_CORE]" = "{
 > 				.tdp_pl1_override = 15,
 > 				.tdp_pl2_override = 38,
 > 				.tdp_pl4 = 71,
 > 			}"
 > 			register "power_limits_config[POWER_LIMITS_U_4_CORE]" = "{
 > 				.tdp_pl1_override = 15,
 > 				.tdp_pl2_override = 60,
 > 				.tdp_pl4 = 105,
 > 			}"
 >
 > {{< quote >}} <https://github.com/coreboot/coreboot/blob/9d4d2d014c91e041cf7fb8ea593364c6644dd644/src/mainboard/intel/tglrvp/variants/tglrvp_up3/devicetree.cb#L130> {{< /quote >}}

[^tgl_up3-pl]: <https://github.com/coreboot/coreboot/blob/9d4d2d014c91e041cf7fb8ea593364c6644dd644/src/mainboard/intel/tglrvp/variants/tglrvp_up3/devicetree.cb#L130>
[^tgl-28w]: [Initial MSI Stealth 15m Review With Thermal Performance | MSI Global English Forum - Index](https://forum-en.msi.com/index.php?threads/initial-msi-stealth-15m-review-with-thermal-performance.356516/)

*Alder Lake-P (6+8+2)* では PL1: 45W/PL2: 115W と特別高く、H-Series の一部、あるいは全体をもモバイル向けである *Alder Lake-P* で構成しようとしていることが予想できる。  
*Alder Lake-P* は CPU側だけで PCIe Gen5 8-Lane、PCIe Gen4 2x4-Lane を持つことが判明しており、PCIe Gen5 をサポートするような将来の Dicrete GPU とも組み合わせることが可能である。  
{{< link >}} [Alder Lake は PCIe Gen5 に対応、Tiger Lake-H は Gen4 20-Lane を備える | Coelacanth's Dream](/posts/2021/01/01/adl_p-gen5-tgl_h-gen4-x20/) {{< /link >}}
また、PL1/PL2 の値は *Tiger Lake-H* と変わらない。*Tiger Lake-H* では PL1: 45W/PL2: 107-135W となっており、PL2 は製品ごとに異なる。[^tgl-h]  

[^tgl-h]: [ASCII.jp：Core i9-11980HKやXeon W-11955Mなど、6コア/8コア版のTiger Lake-Hが正式発表 (1/3)](https://ascii.jp/elem/000/004/054/4054425/)


### Alder Lake-M PL1/PL2 {#adl_m-pl}

*Alder Lake-M* は 2+8+2 の構成の PL1/PL2 設定のみが追加されている。*Alder Lake-M* は *Alder Lake-P* と同じくモバイル向けだが、それよりも I/O の規模を縮小した SKU となる。  

 > ##### Alder Lake-M PL1/PL2 (adlrvp)
 > 			# DPTF
 > 			register "dptf_enable" = "1"
 > 			register "power_limits_config" = "{
 > 				.tdp_pl1_override = 9,
 > 				.tdp_pl2_override = 30,
 > 			}"
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/54491/4/src/mainboard/intel/adlrvp/devicetree_m.cb> {{< /quote >}}

*Alder Lake-M (2+8+2)* には PL1: 9W/PL2: 30W の値が設定されている。  
小型なパッケージを採用する *Tiger Lake UP4* も同様に PL1: 9W だが、PL2 は 2-Core が 35W、4-Core は 40W となっており[^tgl_up4-pl]、*Alder Lake-M (2+8+2)* のそれは *Tiger Lake UP4* よりもわずかに低い値だ。  

 > ##### Tiger Lake UP4 PL1/PL2
 > 			# Add PL1 and PL2 values
 > 			register "power_limits_config[POWER_LIMITS_U_2_CORE]" = "{
 > 				.tdp_pl1_override = 9,
 > 				.tdp_pl2_override = 35,
 > 				.tdp_pl4 = 66,
 > 			}"
 > 			register "power_limits_config[POWER_LIMITS_U_4_CORE]" = "{
 > 				.tdp_pl1_override = 9,
 > 				.tdp_pl2_override = 40,
 > 				.tdp_pl4 = 83,
 > 			}"
 >
 > {{< quote >}} <https://github.com/coreboot/coreboot/blob/9d4d2d014c91e041cf7fb8ea593364c6644dd644/src/mainboard/intel/tglrvp/variants/tglrvp_up4/devicetree.cb#L134> {{< /quote >}}

[^tgl_up4-pl]: <https://github.com/coreboot/coreboot/blob/9d4d2d014c91e041cf7fb8ea593364c6644dd644/src/mainboard/intel/tglrvp/variants/tglrvp_up4/devicetree.cb#L134>

PL1 からは *Tiger Lake UP4* の後継が *Alder Lake-M* になると考えられるが、*Alder Lake-M* の PL1 は 3-9W の範囲で設定可能であるらしく、9W より低い TDP/PL1 の製品帯も *Alder Lake-M* でカバーできる。  

[^ascii]: [ASCII.jp：最後のAtomとなるChromebook向けプロセッサーのJasper Lake　インテル CPUロードマップ (3/3)](https://ascii.jp/elem/000/004/040/4040489/3/)

 > ##### Alder Lake-M PL1/PL2 range (adlrvp)
 > 						## Power Limits Control
 > 						register "controls.power_limits" = "{
 > 							.pl1 = {
 > 									.min_power = 3000,
 > 									.max_power = 9000,
 > 									.time_window_min = 28 * MSECS_PER_SEC,
 > 									.time_window_max = 32 * MSECS_PER_SEC,
 > 									.granularity = 200,
 > 							},
 > 							.pl2 = {
 > 									.min_power = 30000,
 > 									.max_power = 30000,
 > 									.time_window_min = 28 * MSECS_PER_SEC,
 > 									.time_window_max = 32 * MSECS_PER_SEC,
 > 									.granularity = 1000,
 > 							}
 > 						}"
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/54491/4/src/mainboard/intel/adlrvp/devicetree_m.cb> {{< /quote >}}

以前にも書いたが、Atom系コアのみで構成されたプロセッサは *Elkhart/Jasper Lake* が最後となり、今後は Core系 (Big) と組み合わせる形となる。[^ascii]  
{{< link >}} [Alder Lake-M 関連のパッチが Coreboot に投稿される　―― ADL-M は ADL-P の I/O 縮小版か | Coelacanth's Dream](/posts/2021/01/21/adl_m-coreboot/#adl_m-atom) {{< /link >}}
*Alder Lake-M* がその後継をも担うと思われるが、PL2: 30W というのは *Jasper Lake* よりも 10W 高い設定だ。[^jsl-pl]  
*Jasper Lake* の前世代、*Gemini Lake* が PL2: 15W であったから、TDP/PL1: 6W 付近の省電力 Intel プロセッサの PL2 は世代ごとに上げられていることになる[^glk-pl]。  
性能向上のため、特に *Alder Lake-M (2+8+2)* は Atom/Smallコアだけでも 8-Core 搭載しているため、*Jasper Lake* から PL2 +10W は仕方ないとも思うが、省電力プロセッサを求める用途、ユーザーにとってブースト時の最大消費電力はどこまで許容されるものなのだろうか。  

[^jsl-pl]: <https://github.com/coreboot/coreboot/blob/8e57299a0da5dc9928b97d94b0265cf6883de005/src/mainboard/intel/jasperlake_rvp/variants/jslrvp/devicetree.cb#L128>
[^glk-pl]: <https://github.com/coreboot/coreboot/blob/2adb50d32e8cd9c61773b1d60de545255c6a4049/src/mainboard/intel/glkrvp/variants/baseboard/devicetree.cb#L58>

## TGL/JSL/ADL {#table}

|     | Tiger Lake-U | Tiger Lake-H | Jasper Lake | Alder Lake-P | Alder Lake-M | Alder Lake-S |
| :-- | :--:       | :--:         | :--:        | :--:         | :--:         | :--: |
| GPU | Gen12 96EU | Gen12 32EU | Gen11 32EU | Gen12 96EU | Gen12 96EU? | Gen12 32EU |
| CPU side PCIe | Gen4<br>4-Lane x2 | Gen4<br> 16-Lane + 4-Lane |  | Gen5 8-Lane,<br>Gen4 4-Lane x2 | Gen5 8-Lane? or<br>Gen4 4-Lane? | Gen5 8-Lane x2,<br>Gen4 4-Lane x2 |
| PCH | TGP_LP | TGP_H | JSP | ADP_P | ADP_M | ADP_S |
| PCH PCIe Ports | 16 | ? | 8 | 12 | 10 | 28 |
| Power Limit | PL1: 15W<br>/PL2: 38W(2C), 60W(4C)<br>PL1: 9W<br>PL2: 35W(2C), 40W(4C) | PL1: 45W<br>/PL2: 107-135W | PL1: 6W<br>/PL2: 20W | PL1: 15W<br>/PL2: 55W (2+8+2)<br>PL1: 28W<br>/PL2: 64W (4+8+2)<br>PL1: 45W<br>/PL2: 115W | PL1: 9W<br>/PL2: 30W (2+8+2) | ? |


最近、Intel は TDP として使われる PL1 の値は公開するが、ブースト機能 (Intel® Turbo Boost Technology) の維持時間等の基準に使われる PL2 の値についてはデータシート中にも記載しなくなった。PL1 * 1.25 がハードウェアのデフォルトだとはしているが、それは上記 Coreboot に記述された設定を見てもわかるが実際の設定とはかけ離れている。  
そういう訳で、Coreboot のコードに記述され、公開されている PL1/PL2 の情報はそれなりに貴重なものだと考えている。そのままデータシートに記載するのではダメなのかと思わなくもないが。  

