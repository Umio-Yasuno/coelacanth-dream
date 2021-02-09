---
title: "ALDERLAKE_L と Alder Lake-P/M"
date: 2021-02-09T19:49:54+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

先日 2021/01/21、Linux Kernel に *ALDERLAEK_L / Alder Lake mobile* の x86_model を追加するパッチが投稿された。  
このことで自分が言いたいのは、*Alder Lake-L* なるものが存在し、*Alder Lake-S/P/M* に並ぶバリアントとなる、訳ではないだろうということだ。  

 * [x86/cpu: Add another Alder Lake CPU to the Intel family · torvalds/linux@6e1239c](https://github.com/torvalds/linux/commit/6e1239c13953f3c2a76e70031f74ddca9ae57cd3#diff-7bf85b32beb96091abd89790e701cd01fb13bafbbbca17433ad47830820c1391)
 * [[PATCH] x86/cpu: Add another Alder Lake CPU to the Intel family - Tony Luck](https://lore.kernel.org/lkml/20210121215004.11618-1-tony.luck@intel.com/)

 >         #define	INTEL_FAM6_LAKEFIELD		0x8A
 >         #define INTEL_FAM6_ALDERLAKE		0x97
 >        +#define INTEL_FAM6_ALDERLAKE_L		0x9A
 >        
 >         /* "Small Core" Processors (Atom) */
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add another Alder Lake CPU to the Intel family - Tony Luck](https://lore.kernel.org/lkml/20210121215004.11618-1-tony.luck@intel.com/) {{< /quote >}}

## ALDERLAKE_L と Alder Lake-P/M
### intel-family.h

まず今回、パッチの投稿先であるファイル [intel-family.h](https://github.com/torvalds/linux/blob/6e1239c13953f3c2a76e70031f74ddca9ae57cd3/arch/x86/include/asm/intel-family.h) は、Intelプロセッサの x86_model をまとめたヘッダファイルであり、  
そのファイルを先頭部分に記述されたコメントによると、末尾の `_L, _X, _D` 等はマーケットセグメントにおける担当を示すものであることが分かる。  

 >         * OPTDIFF	If needed, a short string to differentiate by market segment.
 >         *
 >         *		Common OPTDIFFs:
 >         *
 >         *			- regular client parts
 >         *		_L	- regular mobile parts
 >         *		_G	- parts with extra graphics on
 >         *		_X	- regular server parts
 >         *		_D	- micro server parts
 >         *
 >         *		Historical OPTDIFFs:
 >         *
 >         *		_EP	- 2 socket server parts
 >         *		_EX	- 4+ socket server parts
 >         *
 >         * The #define line may optionally include a comment including platform names.
 >         */

そのため、{{< comple >}} そもそもパッチのコメントにも書かれているが {{< /comple >}} 追加された `ALDERLAKE_L` はモバイル向け *Alder Lake* となり、その x86_model は `0x9A (154)` ということになる。  

そして、*Alder Lake-S/P/M* の x86_model については、Coreboot へのパッチから既に判明しており、  
*Alder Lake-S* は `Family: 0x6, Model: 0x97` 、*Alder Lake-P/M* は `Family: 0x6, Model: 0x9A` とされる。  
それらは Coreboot のファイルに記述された CPUID が読み取れ、以下の場合は左から 16進数であることを示す `0x` の後に続いて、`Extended Model`、`Extended Family`、`Family`、`Model` といった風に並び、そして末尾が `Stepping` を意味する。[^coreboot-cpuid]  
{{< link >}} [Alder Lake-P を搭載する Chromebookボード 「Brya」、そして Alder Lake-M | Coelacanth's Dream](/posts/2020/12/02/adl-p-chromebook-board-adl-m/) {{< /link >}}

 >        #define CPUID_ALDERLAKE_S_A0	0x90670
 >        #define CPUID_ALDERLAKE_P_A0	0x906a0
 >        #define CPUID_ALDERLAKE_M_A0	0x906a1
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/49629/3/src/soc/intel/common/block/include/intelblocks/mp_init.h> {{< /quote >}}

[^coreboot-cpuid]: [inteltool: Print raw CPUID and make hexadecimal values unambiguous · coreboot/coreboot@dcea700](https://github.com/coreboot/coreboot/commit/dcea700762bc97bd7fcabf2e960d47805129aeb1?branch=dcea700762bc97bd7fcabf2e960d47805129aeb1&diff=unified)

`ALDERLAKE_L` と *Alder Lake-P/M* の x86_model は一致する。  
まとめると、マーケットセグメントとして `ALDERLAKE_L [Model: 0x9A (154)]` があり、それに *Alder Lake-P/M* 2種があたる、という解釈が最も自然と思われる。  

 * ALDERLAKE (Model: 0x97 {151})
    * *Alder Lake-S*
 * ALDERLAKE_L (Model: 0x9A {154})
    * *Alder Lake-P*
    * *Alder Lake-M*

それだけのお話。  
それとついでに言えば、モバイル向け *Alder Lake* の x86_model が `0x9A (154)` であることは、2020/11/25 に Linux向け電力管理ツールである [PowerTOP](https://github.com/fenrus75/powertop) に投稿されたパッチ、プルリクエストから確認されていた。[^powertop]  
それを投稿した [Gayatri Kammela (ユーザー名: gkammela)](https://github.com/gkammela) 氏は Intel のソフトウェアエンジニアであり、Linux Kernel 周りを担当している。  

[^powertop]: [Add Sapphire rapids, Alder Lake mobile and desktop support in Powertop by gkammela · Pull Request #77 · fenrus75/powertop](https://github.com/fenrus75/powertop/pull/77/commits/b4026d1996a6e091aef313c0b2ad4851b1dee56f)
