---
title: "Intel Emerald Rapids の CPUID Model と Granite Rapids のセグメント"
date: 2022-11-07T08:51:52+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "CPUID", "Emerald_Rapids", "Granite_Rapids", "Sierra_Forest", "Grand_Ridge" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Tony Luck 氏により、Linux Kernel に Intel の次世代サーバー向けプロセッサ *Emerald Rapids, Granite Rapids, Sierra Forest, Grand Ridge* の CPUID Model 情報を追加するパッチが投稿されている。  
`CPUID[LEAF=0x1].EAX` から取得できる CPUID Family, Model, Stepping は、主にプロセッサの識別に用いられる。  

追加された CPUID Model の一部は、*Intel® Architecture Instruction Set Extensions Programming Reference: Revision -046* で先に公開されていたが、今回のパッチでは *Emerald Rapids* の CPUID Model と *Granite Rapids* のマーケットセグメントの情報が追加されている。  

 * [[PATCH] x86/cpu: Add several Intel server CPU mode numbers - Tony Luck](https://lore.kernel.org/lkml/20221103203310.5058-1-tony.luck@intel.com/)

*Emerald Rapids* の CPUID Model は `0xCF (207)`、サーバー向けの *Granite Rapids-SP* は `0xAD (173)`、マイクロサーバー向けの *Granite Rapids-D* は `0xAE (174)` とされている。  

 >         +#define INTEL_FAM6_EMERALDRAPIDS_X	0xCF
 >         +
 >         +#define INTEL_FAM6_GRANITERAPIDS_X	0xAD
 >         +#define INTEL_FAM6_GRANITERAPIDS_D	0xAE
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add several Intel server CPU mode numbers - Tony Luck](https://lore.kernel.org/lkml/20221103203310.5058-1-tony.luck@intel.com/) {{< /quote >}}

サーバー向け Atom プロセッサとなる *Sierra Forest* の CPUID Model は `0xAF (175)`、*Grand Ridge* は `0xB6 (182)`。  
だが *Grand Ridge* がどのマーケットセグメントに位置するかは情報が無い。また、*Grand Ridge* は現状 Intel が公開しているロードマップ上には現れていない。  
*Ridge* 系のコードネームからは、*Snow Ridge (Tremont)* のようにマイクロサーバー向け (`_D`) という印象を受けるが。  
Intel のサイトを検索したところでは、ISA ドキュメントの他に *Grand Ridge* と FPGA を組み合わせた例のダイアグラムが引っ掛かるくらいだった。[^grand-ridge-fpga]  
また、今回のパッチや LLVM へのパッチの中で *Grand Ridge* が Atom 系であることが一応明言されている。  

[^grand-ridge-fpga]: [1.2.2.2. Ethernet Bridge + Inline MACsec](https://www.intel.com/content/www/us/en/docs/programmable/736108/22-3-1-2-0/ethernet-bridge-inline-macsec.html)

*Grand Ridge* では *Sierra Forest* の対応命令範囲に加え、`ADD/AND/OR/XOR` 命令に Atomic 操作を追加した `RAD-INT` 命令に対応することから、*Sierra Forest* と *Grand Ridge* は異なるマイクロアーキテクチャを採用する可能性が考えられる。  

 >         +#define INTEL_FAM6_SIERRAFOREST_X	0xAF
 >         +
 >         +#define INTEL_FAM6_GRANDRIDGE		0xB6
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add several Intel server CPU mode numbers - Tony Luck](https://lore.kernel.org/lkml/20221103203310.5058-1-tony.luck@intel.com/) {{< /quote >}}

 >          * OPTDIFF	If needed, a short string to differentiate by market segment.
 >          *
 >          *		Common OPTDIFFs:
 >          *
 >          *			- regular client parts
 >          *		_L	- regular mobile parts
 >          *		_G	- parts with extra graphics on
 >          *		_X	- regular server parts
 >          *		_D	- micro server parts
 >          *		_N,_P	- other mobile parts
 >          *		_S	- other client parts
 >         
 >
 > {{< quote >}} [Linux Kernel] arch/x86/include/asm/intel-family.h {{< /quote >}}

{{< ref >}}
 * [Intel Sierra Forest, Grand Ridge, Granite Rapids でサポートされる新命令 | Coelacanth's Dream](/posts/2022/10/04/intel-ise-rev_46/)
 * [[PATCH 0/2] Intel Grand Ridge Support](https://gcc.gnu.org/pipermail/gcc-patches/2022-November/605144.html)
 * [⚙ D137153 [WIP][X86] Support -march=sierraforest, grandridge, graniterapids.](https://reviews.llvm.org/D137153)
{{< /ref >}}
