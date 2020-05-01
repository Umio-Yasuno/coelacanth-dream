---
title: "Intel Comet Lake PCHを整理する ――LPDDR4とか22nmとか"
date: 2020-05-01T23:41:56+09:00
draft: false
tags: [ "PCH", ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "Chipset" ]
noindex: false
---

先日、Intelがデスクトップ向けに第10世代Core *Comet Lake-S* 、それをサポートする *400シリーズチップセット* の情報を公開した。  
{{< link >}}[Intel Delivers World’s Fastest Gaming Processor | Intel Newsroom](https://newsroom.intel.com/news/intel-delivers-worlds-fastest-gaming-processor/){{< /link >}}
{{< link >}}[Intel® 400 Series Chipsets Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/series/201843/intel-400-series-chipsets.html){{< /link >}}

そしてその *400シリーズチップセット* だが、ベースとなるPCHが複数存在し、またそれぞれ微妙に違いが存在するため、自分の頭の中を整理するためにも今回記事としてまとめることとした。  

主な参考資料:

 * [IntelFsp/FSP: Intel(R) Firmware Support Package (FSP)](https://github.com/IntelFsp/FSP)
    * <https://github.com/IntelFsp/FSP/tree/master/CometLakeFspBinPkg>


## インデックス

 * [CometLake1](#cometlake1)
 * [CometLake2](#cometlake2)
 * [CometLakeS / CometLakeV](#cometlake-s-v)

## CometLake1
*Comet Lake U-Series /H-Series* に用いられるPCH。  
サポートするメモリタイプは LPDDR3 と DDR4だが、ここが面白い点であり、Intelは *Comet Lake U-Series* 発表時、LPDDR4/xメモリをサポートするとしていた。[^1]  
[AnandTech](https://www.anandtech.com)によれば、最初のB0ステッピングではサポートされず、K1ステッピングでサポートされることになっている。また、そうなった理由として、LPDDR3 と DDR4 はメモリコントローラーのデザインが似通っており、IPをほぼ再利用できるが、LPDDR4はそうではなく、デザインの変更を必要とするため検証に時間が掛かったのではないかと推測している。[^2]  

[^1]: [Intel Expands 10th Gen Intel Core Mobile Processor Family, Offering Double Digit Performance Gains | Intel Newsroom](https://newsroom.intel.com/news/intel-expands-10th-gen-intel-core-mobile-processor-family-offering-double-digit-performance-gains/)
[^2]: [Intel: 28 W Ice Lake Core i7-1068G7 Coming Q1](https://www.anandtech.com/show/15302/intel-28-w-ice-lake-core-i71068g7-coming-q1)

## CometLake2
そしてその LPDDR4メモリをサポートするPCHが *CometLake2* となっている。代わりに *CometLake1* でサポートされていた LPDDR3メモリが、サポートから外れている。  

違いとしてはそれだけだが、Intelは *Comet Lake U-Series* のB0ステッピングとK1ステッピングでSKU名を変えていないため、それだけでは判別が不可能となっている。  
具体的な名前を出すと、**Core i7-10710U** に *CometLake1* と *CometLake2* が混在している。  
搭載されているシステムのメモリが LPDDR3、または LPDDR4であれば判別は可能だが、この方法だと DDR4を採用している場合破綻する。  
そこで Intel は CPUIDのEAXレジスタの値で判別することを推奨しており、*CometLake1 /B0ステッピング* は値が `0x000A0660` または `0x000806EC` 、*CometLake2 /K1ステッピング* は値が `0x000A0661` となる。  
しかしこの方法にも問題があるはずだ。消費者からすれば、自分が買おうとしているPCに搭載されたCPUの CPUID を知ることなんてできない。DDR4メモリを採用している場合は完全に不明となる。  
メモリ以外で性能に関わる変更が為されていないことを願うのみだ。  


## CometLakeS / CometLakeV {#cometlake-s-v}
デスクトップ向けのPCHには *CometLakeS* と *CometLakeV* の2種類が存在し、  
*CometLakeS* は **14nmプロセス** 、*CometLakeV* は **22nmプロセス** で製造される。これが *CometLakeS* とのわかりやすい違いとなるだろう。  

前世代から *CometLakeS* に追加された新機能には 2.5G Ethernetコントローラー **I225**、Wi-Fi 6モジュール **AX200/201** のサポートがあげられる。  

*CometLakeS* の具体的な製品名は、*Z490 /H470* 、まだ Intel のサイトには掲載されていないが *Q470 /W480* 、そして意外なことに *B460* となっている。  

 > For all other Intel® 400 Series Chipsets, use CometLakeS. For the Intel® B460 Chipset use CometLakeS.

 > 引用元:<cite>[FSP/README.md at 8af76d732b006acceaaee184b597a46f39ec15e3 · IntelFsp/FSP](https://github.com/IntelFsp/FSP/blob/8af76d732b006acceaaee184b597a46f39ec15e3/CometLakeFspBinPkg/README.md)</cite>

意外だと感じたのは、[Coreboot](https://github.com/coreboo)のPCI IDリストに *Comet Lake PCH* の中で *H410* と *B460* が無く、*CometLakeV* だと思っていたからだ。*H410* と、これも未発表だが *B460C* は *CometLakeV* と記述されている。[^7]  
Intel のサイトを確認すると、確かに *B460* のブロックダイアグラムには 2.5G Ethernet とWi-Fi 6 **AX200** が描かれているのに対し、*H420E /H410* にはそれらがない。[^5][^6]  
Coreboot は単なる記載漏れなのだろうか？  
しかしパターンとして、リネームされたチップセットは記載されていない。(例: Z370)。  
*H470* と *B460* で PCI ID を共有している可能性も考えられるだろうか？  

言えるのは、少なくとも *B460* は *CometLakeS* ということだ。逆にそうわざわざ書いてあることが引っかかりもするが。  

[^5]: [Intel® B460 Chipset Product Brief](https://www.intel.com/content/www/us/en/products/docs/chipsets/desktop-chipsets/b460-chipset-brief.html?wapkw=b460)
[^6]: [Comet Lake S: Overview and Technical Documentation](https://www.intel.com/content/www/us/en/design/products-and-solutions/processors-and-chipsets/comet-lake-s/overview.html?wapkw=H410&grouping=EMT_Content%20Type&sort=title:asc)
[^7]: [FSP/README.md at 8af76d732b006acceaaee184b597a46f39ec15e3 · IntelFsp/FSP](https://github.com/IntelFsp/FSP/blob/8af76d732b006acceaaee184b597a46f39ec15e3/CometLakeFspBinPkg/README.md)

*CometLake1* 、*CometLake2* 、*CometLakeS* は、Linux Kernelからは *Comet Lake PCH* と、  
*CometLakeV* は *Comet Lake V PCH* と判定される。[^3][^4]  

[^3]: <https://github.com/coreboot/coreboot/blob/d7a6d61d51ebf4025484e567a20b2b50fda5a6f9/src/include/device/pci_ids.h#L2840>
[^4]: <https://github.com/torvalds/linux/blob/50a5065f4474c2dbc1f7462b45a32d33d7b48d88/drivers/gpu/drm/i915/intel_pch.h#L44>

| | Process | Product |
| :--- | :---: | :---: |
| CometLakeS | 14nm | Z490 /H470 /B460<br>/W480 /Q470 |
| CometLakeV | 22nm | B460C /H420E? /H410 |

{{< ref >}}

 * [IntelFsp/FSP: Intel(R) Firmware Support Package (FSP)](https://github.com/IntelFsp/FSP)
    * <https://github.com/IntelFsp/FSP/tree/master/CometLakeFspBinPkg>
 * [coreboot/coreboot: Mirror of https://review.coreboot.org/coreboot.git. We don't handle Pull Requests.](https://github.com/coreboot/coreboot)
    * [coreboot/pci_ids.h at d7a6d61d51ebf4025484e567a20b2b50fda5a6f9 · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/d7a6d61d51ebf4025484e567a20b2b50fda5a6f9/src/include/device/pci_ids.h#L2845)
 * [linux/intel_pch.h at 50a5065f4474c2dbc1f7462b45a32d33d7b48d88 · torvalds/linux](https://github.com/torvalds/linux/blob/50a5065f4474c2dbc1f7462b45a32d33d7b48d88/drivers/gpu/drm/i915/intel_pch.h#L44)
 * [linux/intel_pch.c at 48a1b8d4af01abd38e51cef205a0f2c4deeb092a · torvalds/linux](https://github.com/torvalds/linux/blob/48a1b8d4af01abd38e51cef205a0f2c4deeb092a/drivers/gpu/drm/i915/intel_pch.c)

{{< /ref >}}
