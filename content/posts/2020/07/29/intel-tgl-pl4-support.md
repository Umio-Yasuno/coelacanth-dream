---
title: "Intel Tiger Lake は \"Power Limit4\" をサポート"
date: 2020-07-29T16:33:57+09:00
draft: false
tags: [ "Tiger_Lake", ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

Intel の次世代モバイル向けプロセッサ *Tiger Lake* では、SoCパッケージレベルの最大電力制限、*Power Limit4* をサポートすることがわかった。  
{{< link >}} [powercap: Add Power Limit4 support](https://git.kernel.org/pub/scm/linux/kernel/git/rafael/linux-pm.git/commit/?h=bleeding-edge&id=8365a898fe53f85529566501d3b3d88640b3975e) {{< /link >}}
この *Power Limit4* (以下 *PL4* ) は、以前から Intelプロセッサのデータシートに記述されてはいたが、サポートするプロセッサはこれまで無かった。[^10th-gen-core-datasheet]  

[^10th-gen-core-datasheet]: <https://www.intel.com/content/dam/www/public/us/en/documents/datasheets/10th-gen-core-families-datasheet-vol-1-datasheet.pdf#page=56>

*PL4* は SoC の瞬間的な消費電力が電源アダプタやバッテリーの許容範囲を超えないよう、先制的に SoC の消費電力、動作クロックを制限するためのものとされている。  
*PL4* が *Tiger Lake* でサポートされるということは、*Comet Lake* から一部のモデルに導入された *Velocity Boost* のような、温度と消費電力に余裕がある場合に動作クロックを引き上げる機能が *Tiger Lake* にも導入されるのかもしれない。  

そして、*Tiger Lake* の *PL4* にはどのような値が設定されているのかというと、ある例では以下のようになっている。  

 >       register "power_limits_config[POWER_LIMITS_U_2_CORE]" = "{
 >             .tdp_pl1_override = 15,
 >             .tdp_pl2_override = 38,
 >             .tdp_pl4 = 71,
 >       }"
 >       register "power_limits_config[POWER_LIMITS_U_4_CORE]" = "{
 >             .tdp_pl1_override = 15,
 >             .tdp_pl2_override = 60,
 >             .tdp_pl4 = 105,
 >       }"
 >       register "power_limits_config[POWER_LIMITS_Y_2_CORE]" = "{
 >             .tdp_pl1_override = 9,
 >             .tdp_pl2_override = 35,
 >             .tdp_pl4 = 66,
 >       }"
 >       register "power_limits_config[POWER_LIMITS_Y_4_CORE]" = "{
 >             .tdp_pl1_override = 9,
 >             .tdp_pl2_override = 40,
 >             .tdp_pl4 = 83,
 >       }"
 >
 > 引用元: [soc/intel/tigerlake: Set power limits for Tiger Lake Y-SKU (If4f12264) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/43607)

この時気を付けたいのは、まず上記の値が設定されているのは、*Tiger Lake* を搭載する Chromebookボードのベースボード、親的な存在である **Volteer** 用の値であり、*Tiger Lake* のデフォルト設定ではないこと、  
そして、あくまでも *PL4* は SoC のピーク消費電力を制限するものであり、そこに到達するまで動作クロックを引き上げるものではなく、そもそも到達する前に温度的な限界が来るだろう。  
*Tiger Lake* は iGPU を搭載し、TDP などを CPU と GPU で分け合うため、CPU だけで食い切れるものでもない。  

まあそれ抜きにしても、仕様における TDP を示す *PL1* と *PL2* がかけ離れており、中々に凄いものがあるが。  
モバイル向け *Ice Lake* の *PL2* は [AnandTech](https://www.anandtech.com/) による検証から、50W に設定されていると考えられるが[^icelake-pl2-anandtech]、上記 *Tiger Lake-U (4-Core)* はそれを超えて 60W に設定されている。  

[^icelake-pl2-anandtech]: [Power Results (15W and 25W) - The Ice Lake Benchmark Preview: Inside Intel's 10nm](https://www.anandtech.com/show/14664/testing-intel-ice-lake-10nm/5)

しかし、繰り返すが *PL2* の値もあくまで **Volteer** 用に設定された値であり、*Tiger Lake* のデフォルト設定ではないことを留意したい。  

{{< ref >}}

 * [Linux To Allow Limiting Tiger Lake SoC PL4 Package Maximum Power Limit - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Intel-TGL-Linux-Power-Limit4)

{{< /ref >}}
