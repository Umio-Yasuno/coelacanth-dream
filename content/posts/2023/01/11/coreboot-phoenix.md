---
title: "Coreboot で Phoenix APU のサポートが進められる"
date: 2023-01-11T00:32:51+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Coreboot", "Phoenix" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

様々なアーキテクチャに対応する BIOS/UEFI、ブートファームウェアを開発するオープンソースプロジェクト Coreboot で *AMD Phoenix APU* のサポートが進められている。  
2022/10 頃から *Morgana* と呼ぶ APU/SoC のサポートが進められていたが、これが実は *Phoenix APU* に向けたものであり、先日 AMD が *Phoniex APU* ベースの Ryzen 7040HS Series を正式発表したことで Coreboot のコード上で名前を隠す必要性がなくなった。  
名前を明かす以前のコードは他の APU/SoC のコードをコピーしたものがほとんどであり、名前だけでなくスペックに関する情報も隠されていた。  
*Mendocino APU (Ryzen 7020 Series)* も当初は *Sabrina APU* の名でサポートが進められていた。  
こうした動きからは正式発表のハードウェアに関する情報を守りながら、いかにオープンソースプロジェクトにサポートを追加し、企業とその開発者として貢献を続けていくかという問題を見ることができる。  

 >         soc/amd: Change Morgana codename to Phoenix
 >         
 >         Now that the next generation of APUs is officially announced, we can
 >         unmask morgana.
 >         
 >         The chip formerly known as Morgana is actually Phoenix.
 >         
 >         Surprise!
 >
 > {{< quote >}} [soc/amd: Change Morgana codename to Phoenix (Ie9492a30) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/71731/2) {{< /quote >}}

 * [soc/amd: Change Morgana codename to Phoenix (Ie9492a30) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/71731/2)

*Phoenix APU* の `CPUID Family/Model` は `Family: 0x19 (25), Model: 0x78 (120)` とされている。  
AMD は `Model: 0x70-0x7F` の範囲を *Phoenix APU* の `CPUID Model` に割り当てていると考えられ、`Model: 0x78 (120)` 以外で範囲内の `CPUID Model` を持つエンジニアサンプリング品 (`Model: 0x70, 0x74`) がネット上で確認されたりもしている。[^model-112] [^model-116]  
AMD CPU では、エンジニアサンプリング品、Ax ステッピングと Bx ステッピングで範囲内の別の `CPUID Model` が使われることがある。  

[^model-112]: <https://milkyway.cs.rpi.edu/milkyway/show_host_detail.php?hostid=931791>
[^model-116]: <https://boinc.bakerlab.org/rosetta/show_host_detail.php?hostid=6214905>

 >         /* SPDX-License-Identifier: GPL-2.0-only */
 >         
 >         #ifndef AMD_MORGANA_CPU_H
 >         #define AMD_MORGANA_CPU_H
 >         
 >         #define MORGANA_A0_CPUID	0x00a70f80
 >         
 >         #endif /* AMD_MORGANA_CPU_H */
 >
 > {{< quote >}} […/cpu.h · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/71689/4/src/soc/amd/morgana/include/soc/cpu.h) {{< /quote >}}

Coreboot では *Morgana/Phoenix* に近い APU/SoC として、*Glinda* の名でサポートが進められているものもあるが、こちらは AMD より正式発表がされていないことからまだ実際のコードネームや `CPUID Family/Model` は明かされていない。  
