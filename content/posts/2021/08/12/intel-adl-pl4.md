---
title: "Alder Lake-P/M の PL4 参考値と Brya の設定値"
date: 2021-08-12T03:05:04+09:00
draft: false
tags: [ "Alder_Lake", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Coreboot に、*Alder Lake* の PL4 (Power Limit 4) を ADLRVP (Reference Validation Platform)、Chromebook向けリファレンスボードである Brya に追加するパッチが投稿されている。  
PL4 は SoC、パッケージレベルの最大電力制限となり、電力管理機能は瞬間的にも消費電力が PL4、その基準となる電源アダプタやバッテリーの許容範囲を超えないよう先制的に動作クロック等を調整、制限することになる。  
PL3/PL4 は、プロセッサのデータシートには記載されているもののデフォルトでは無効化されていたが、*Tiger Lake* の世代からサポート、有効化する向きを見せている。  
{{< link >}} [Intel Tiger Lake は "Power Limit4" をサポート | Coelacanth's Dream](/posts/2020/07/29/intel-tgl-pl4-support/) {{< /link >}}

 * [mb/google/brya: set PL4 value dynamically for thermal (I20b98ccd) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56915/1)
 * [mb/google/brya/variants/brya0: add PL4 values for different SKUs (I095e9eda) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56916/1)
 * [soc/intel/alderlake: set default PL4 values for differnt SKUs (I53791bad) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56917/1)

追加された PL4 の値を見てみると、*Alder Lake-P (2+8+2)* で 123W、*4+8+2* で 140W、*6+8+2* で 215W、*Alder Lake-M (2+8+2)* は 68W が設定されている。(2+8+2) といったフォーマットについては以前の記事を参照。  
{{< link >}} [Alder Lake-P/M のステッピング、PL1/PL2 参考値 | Coelacanth's Dream](/posts/2021/05/26/adl-recent-info-2021-05-26/#adl-power) {{< /link >}}

 > 			register "power_limits_config[ADL_P_POWER_LIMITS_282_CORE]" = "{
 > 				.tdp_pl1_override = 15,
 > 				.tdp_pl2_override = 55,
 > 				.tdp_pl4 = 123,
 > 			}"
 > 			register "power_limits_config[ADL_P_POWER_LIMITS_482_CORE]" = "{
 > 				.tdp_pl1_override = 28,
 > 				.tdp_pl2_override = 64,
 > 				.tdp_pl4 = 140,
 > 			}"
 > 			register "power_limits_config[ADL_P_POWER_LIMITS_682_CORE]" = "{
 > 				.tdp_pl1_override = 45,
 > 				.tdp_pl2_override = 115,
 > 				.tdp_pl4 = 215,
 > 			}"
 > 			register "power_limits_config[ADL_M_POWER_LIMITS_282_CORE]" = "{
 > 				.tdp_pl1_override = 9,
 > 				.tdp_pl2_override = 30,
 > 				.tdp_pl4 = 68,
 > 			}"
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/56917/1/src/soc/intel/alderlake/chipset.cb> {{< /quote >}}

パッと見ての印象は高く、実際 *Tiger Lake-U (UP3/UP4)* と比べても高い値が設定されているのだが、気を付けなければならないのは、PL4 はパッケージレベルの電力制限であり、電源アダプタやバッテリーの許容範囲を超えないよう設定されたものであること、  
そして上記引用部は、*Alder Lake-P/M* における各 Power Limit の参考値、あるいは最大値であることだ。  

|     | Tiger Lake-U | Tiger Lake-H | Alder Lake-P<br>(ADLRVP) | Alder Lake-M |
| :-- | :--:       | :--:         | :--:        | :--: |
| PL1 | UP3: <=28W<br>UP4: <=9W | <=45W | (2+8+2): <=15W<br>(4+8+2): <=28W<br>(6+8+2): <=45W | (2+8+2): <=9W |
| PL2 | UP3: <=38W (2C), <=60W (4C) <br> UP4: <=35W (2C), <=40W (4C) | 107-135W | (2+8+2): <=55W<br>(4+8+2): <=64W<br>(6+8+2): <=115W | (2+8+2): <=30W |
| PL4 | UP3: <=71W (2C), <=105W (4C) <br> UP4: <=66W (2C), <=83W (4C) |   | (2+8+2): <=123W <br> (4+8+2): <=140W <br> (6+8+2): <=215W | (2+8+2): <=68W |

そして、*Alder Lake-P/M* を搭載する Chromebook のリファレンスボードとなる **Brya0** の構成ファイルを見ると、以下のように上とは異なる低い値が設定されている。  

 > 		const struct cpu_power_limits limits[] = {
 > 			/* SKU_ID, pl1_min, pl1_max, pl2_min, pl2_max, pl4 */
 > 			/* All values are for baseline config as per bug:191906315 comment #10 */
 > 			{ PCI_DEVICE_ID_INTEL_ADL_P_ID_7, 3000, 15000, 39000, 39000, 100000},
 > 			{ PCI_DEVICE_ID_INTEL_ADL_P_ID_5, 4000, 28000, 43000, 43000, 105000},
 > 			{ PCI_DEVICE_ID_INTEL_ADL_P_ID_3, 5000, 45000, 80000, 80000, 159000},
 > 		};
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/56916/1/src/mainboard/google/brya/variants/brya0/ramstage.c> {{< /quote >}}

PL1 から想定している *Alder Lake-P* のコア構成を推測し、整理した表が以下。  
**Brya0** の設定は控えめなものと言え、PL1 が同じ *Alder Lake-P (4+8+2)* と *Tiger Lake-U (UP3) 4-Core* の参考値と比べても低く、PL4 は同じ値となる。  

| | Alder Lake-P<br>(brya0, Chromebook reference board) |
| :-- | :--: |
| PL1 | (2+8+2)?: 3-15W <br> (4+8+2)?: 4-28W <br> (6+8+2)?: 5-45W |
| PL2 | (2+8+2)?: 39W <br> (4+8+2)?: 43W <br> (6+8+2)?: 80W |
| PL4 | (2+8+2)?: 100W <br> (4+8+2)?: 105W <br> (6+8+2)?: 159W |

結局のところ Power Limit はボードと冷却システムを設計する OEMベンダー次第となり、参考値、あるいは最大値だけを見て *Alder Lake-P/M* の消費電力を判断するのは危険だろう、というのが自分の意見となる。  
*Tiger Lake* の時点でそういった話があったが、電源供給部や冷却システムの設計が性能により影響する点ではユーザーが混乱しやすく、それはそれで別の意見が出てくるかもしれないが、自分はレビュアーでもなければ、優秀な消費者という訳でもないため、そういったことに関して自分に言えることはそうないと感じている。  

