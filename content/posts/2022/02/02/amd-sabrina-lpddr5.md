---
title: "AMD Sabrina SoC は LPDDR5メモリをサポート"
date: 2022-02-02T15:09:19+09:00
draft: false
tags: [ "Sabrina", "Coreboot", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

Google の Karthik Ramasubramanian 氏より、Coreboot のメモリツールに *AMD Sabrina SoC* のサポートを追加するパッチが投稿された。  
パッチコメントでは、*AMD Sabrina SoC* は LPDDR5メモリをサポートする予定にあるとしている。  
Coreboot では、*Alder Lake-N/M/P* と LPDDR5メモリを組み合わせたプラットフォームをサポートしているが、*Alder Lake-N/M/P* は SPD (Serial Presence Detect) のバージョンに 1.0、*AMD Sabrina SoC* では 1.1 を想定している。パッチはその違いに対応することを目的としたもの。  

* [spd/lp5: Generate initial SPDs for Sabrina SoC (Ibb43f26b) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61543/2)
* [util/spd_tools/spd_gen: Add support for Sabrina SoC (I2a2c0d0e) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61542/2)

 > 		Add support to generate SPD binary for Sabrina SoC. Mainboards using
 > 		Sabrina SoC are planning to use LP5 memory technology. Some of the SPD
 > 		bytes expected by Sabrina differ from the existing ADL. To start with,
 > 		memory training code for Sabrina expects SPD Revision 1.1. More patches
 > 		will follow to accommodate additional differences.
 >
 > {{< quote >}} [util/spd_tools/spd_gen: Add support for Sabrina SoC (I2a2c0d0e) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61542/2) {{< /quote >}}

*AMD Sabrina SoC* は現在 Coreboot でサポートが進められており、CPUID (Family, Model)、GPU DeviceID、一部 I/O 構成は明かされているが、その他 OSS、コンパイラや Linux Kernel の各種ドライバーにはまだパッチが投稿されていない。  
{{< link >}} [AMD Sabrina SoC に向けたさらなるパッチ | Coelacanth's Dream](/posts/2022/01/14/amd-sabrina-soc-more-patch/) {{< /link >}}
まだ謎の多い AMD APU/SoC だが、今回でメモリに LPDDR5 をサポートすることが公開された。  
GPU部に *GFX9/Vega* を共通して採用する *Renoir, Lucienne, Green Sardine (Cezanne), Barcelo* は DDR4/LPDDR4メモリのサポートに留まるため、*AMD Sabrina SoC* ではメモリインターフェイスの世代が進んでいると見られる。  
LPDDR5メモリをサポートする AMD APU/SoC は現在、*VanGogh/Aerith (Zen 2 + RDNA 2)* と *Yellow Carp/Rembrandt (Zen 3 + RDNA 2)* があり、メモリインターフェイスだけで言えば *AMD Sabrina SoC* は *GFX9/Vega* 系 APU より、*GFX10.3/RDNA 2* 系 APU に近い。  
しかし、*AMD Sabrina SoC* では SATAコントローラを持たず、CPU は CPPC2 をサポートしないなど、それらより小規模な APU/SoC であることが予想される。  

