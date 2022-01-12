---
title: "Alder Lake-N を搭載する Chromebookボード Nissa, Nivviks, Nereid"
date: 2022-01-12T21:31:27+09:00
draft: false
tags: [ "Alder_Lake", "Coreboot", "Chromebook" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Google の Reka Norman 氏より、Coreboot に *Alder Lake-N* の Chromebookボード、**Nissa /Nivviks /Nereid** のサポートを追加するパッチが投稿された。  
**Nissa** のベースボード、**Nivviks /Nereid** はそこから派生したリファレンスボードとなる。  

 * [mb/google/brya: Add new baseboard nissa with variants nivviks and nereid (I2a3975fb) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/60271)

*Alder Lake-N* のサポートが Coreboot に追加され始めたとき、自分はモバイル向けの *Alder Lake-P/M* とは異なる CPUID Model が割り当てられていることから、別の用途、組み込み向けのバリアントではないかと推測した。  
しかし、*Alder Lake-N* が Intel の RVP (Reference Validation Platform) だけでなく、Chromebookプラットフォームでも搭載ボードが開発、サポートが進められていることから、*Alder Lake-N* がモバイル向けとして登場する可能性が濃くなった。  
{{< link >}} [Alder Lake-N は Atom系コアのみの構成か | Coelacanth's Dream](/posts/2021/11/25/adl_n-atom-only/) {{< /link >}}

*Alder Lake-N* では他バリアントよりも I/O の規模が小さくなっている。  
*Alder Lake-S/P/M* では CPU側に PCIeインターフェイスが実装されているが、*Alder Lake-N* では実装されず、PCH側のみに実装されている。  
また PCIe Root Port は 5-ports、最大 9-Lanes までとされ、現世代の Atom系プロセッサ *Jasper Lake* に近い規模となる。  
{{< link >}} [Alder Lake-N の PCIe I/O 構成 | Coelacanth's Dream](/posts/2021/12/02/adl_n-io/) {{< /link >}}
USB Type-C についても、*Alder Lake-P* では最大 4-Ports をサポートするが、*Alder Lake-N/M* は 2-Ports までとされる。  
メモリには LPDDR5 をサポートしている。[^adl_n-lp5] *Alder Lake-P/M* 同様に LPDDR4, DDR5 もサポートしているかは不明。  

[^adl_n-lp5]: [mb/intel/adlrvp_n: Add support for ADL-N LP5 RVP (Ib2f53e65) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/60193)

 > 		adlrvp: Configure typec ports at runtime
 > 		
 > 		Default board ADL-P has 4 typec ports , while
 > 		ADL-M/N has only 2 typec ports. Configure the
 > 		number of typec ports based on board id. Also,
 > 		enable the config - CONFIG_USB_PD_COUNT_RUNTIME
 > 		to indicate that usb pd port count is retrieved
 > 		at run time.
 >
 > {{< quote >}} [adlrvp: Configure typec ports at runtime (I7daf1f72) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/3220900) {{< /quote >}}

Intel はハイブリッドアーキテクチャを採用する *Alder Lake-S/P/M* は正式に発表しているが、Atomコアのみの構成と目される *Alder Lake-N* は未発表となっている。  
元より Atom系プロセッサは *Tremont アーキテクチャ* の世代、*Elkhart/Jasper Lake* で最後になるという話があったことから、どこか発表しにくいものがあるのかもしれない。[^ascii-atom]  

[^ascii-atom]: [ASCII.jp：最後のAtomとなるChromebook向けプロセッサーのJasper Lake　インテル CPUロードマップ (3/3)](https://ascii.jp/elem/000/004/040/4040489/3/)

