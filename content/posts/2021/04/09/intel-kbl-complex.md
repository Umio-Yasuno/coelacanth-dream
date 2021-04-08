---
title: "Intel Kaby Lake 周りの CPU Stepping を整理するパッチ"
date: 2021-04-09T01:56:25+09:00
draft: false
tags: [ "Sapphire_Rapids", "Cannon_Lake" ]
# keywords: [ "", ]
categories: [ "Intel", "CPU", "Hardware" ]
noindex: false
# summary: ""
---

Linux Kernel には Intel CPU (`Family: 0x6`) の判別をするため、`Model` をまとめたヘッダファイル [intel-family.h](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h) があるが、Intel の Peter Zijlstra 氏により、そこに複雑な *Kaby Lake* 周りを整理するためのコメントを追加するパッチが投稿された。  

 * [LKML: Peter Zijlstra: [PATCH] x86/cpu: Resort and comment Intel models](https://lkml.org/lkml/2021/3/15/2071)
 * [[PATCH] x86/cpu: Resort and comment Intel models](https://lore.kernel.org/lkml/YE+HhS8i0gshHD3W@hirez.programming.kicks-ass.net/T/)


{{< pindex >}}

 * [Kaby Lake](#kbl)
 * [Palm Cove](#cnl)
 * [Sapphire Rapids == Golden Cove? / Willow Cove?](#spr)

{{< /pindex >}}

## Kaby Lake {#kbl}

CPUマイクロアーキテクチャに *Sky Lake アーキテクチャ* を採用し、Intel 14nmプロセスで製造されるプロセッサにはモバイル向け、デスクトップ向けといった括りを無視すれば、*Sky Lake / Kaby Lake / Amber Lake / Whiskey Lake / Coffee Lake / Comet Lake* の 6種類が存在する。  
`Family, Model` による CPU の判別において、*Sky Lake (_L/X)* 、*Comet Lake (_L)* は `Model` がそれぞれ違っているため判別することは容易だが、*Kaby Lake* についてはそうはいかない。  
{{< link >}} [【雑記】CPUID の Family と Model、表示する一部ソフトウェアの問題点 | Coelacanth's Dream](/posts/2021/03/01/cpuid-family-model/) {{< /link >}}

モバイル向け *Kaby Lake (KABYLAKE_L)* は `Model: 0x8E (142)` が与えられているが、同じ Model が *Amber Lake(-Y) / Coffee Lake(-U) / Whiskey Lake(-U)* が使われている。  
`intel-family.h` では、末尾 `_L` がモバイル向け、`_X` がサーバー向け、`_D` がマイクロサーバー向け、無印がデスクトップ向けと区別されている。  
デスクトップ向け *Kaby Lake* は `Model: 0x9E (158)` が与えられており、同様に *Coffee Lake(-S)* が同じ Model となる。モバイル向けよりはマシだが、ややこしいことに変わりはない。  
同じ Model を使う *Kaby Lake* 群は代わりに Stepping で区別され、それをコメントではっきりと示したのが今回のパッチ内容となる。  

コメントによれば、*Amber Lake(-Y) (AMBERLAKE_L)* が `Stepping: 9` 、*Coffee Lake(-U) (COFFEELAKE_L)* が `Stepping: 0xA (10)`、*Whiskey Lake(-U) (WHISKEYLAKE_L)* が `Stepping 0xB/0xC (11/12)` を、  
*Coffee Lake(-S)* が `Stepping: 0xA/0xB/0xC/0xD (10/11/12/13)` を用いる。  

 > 		+#define INTEL_FAM6_SKYLAKE_L		0x4E	/* Sky Lake             */
 > 		+#define INTEL_FAM6_SKYLAKE		0x5E	/* Sky Lake             */
 > 		+#define INTEL_FAM6_SKYLAKE_X		0x55	/* Sky Lake             */
 > 		 
 > 		-#define INTEL_FAM6_CANNONLAKE_L		0x66
 > 		+#define INTEL_FAM6_KABYLAKE_L		0x8E	/* Sky Lake             */
 > 		+/*                 AMBERLAKE_L		0x8E	   Sky Lake -- s: 9     */
 > 		+/*                 COFFEELAKE_L		0x8E	   Sky Lake -- s: 10    */
 > 		+/*                 WHISKEYLAKE_L	0x8E       Sky Lake -- s: 11,12 */
 > 		 
 > 		-#define INTEL_FAM6_ICELAKE_X		0x6A
 > 		-#define INTEL_FAM6_ICELAKE_D		0x6C
 > 		-#define INTEL_FAM6_ICELAKE		0x7D
 > 		-#define INTEL_FAM6_ICELAKE_L		0x7E
 > 		-#define INTEL_FAM6_ICELAKE_NNPI		0x9D
 > 		+#define INTEL_FAM6_KABYLAKE		0x9E	/* Sky Lake             */
 > 		+/*                 COFFEELAKE		0x9E	   Sky Lake -- s: 10-13 */
 > 		 
 > 		-#define INTEL_FAM6_TIGERLAKE_L		0x8C
 > 		-#define INTEL_FAM6_TIGERLAKE		0x8D
 > 		+#define INTEL_FAM6_COMETLAKE		0xA5	/* Sky Lake             */
 > 		+#define INTEL_FAM6_COMETLAKE_L		0xA6	/* Sky Lake             */
 >
 > {{< quote >}} [LKML: Peter Zijlstra: [PATCH] x86/cpu: Resort and comment Intel models](https://lkml.org/lkml/2021/3/15/2071) {{< /quote >}}


Stepping は基本大きい程進んだものとなっており、以前の Stepping に存在したエラッタの修正や脆弱性への対応が盛り込まれている。  
*Coffee Lake(-S)* の Stepping 数が多いのは脆弱性への対策が理由として大きく、Intel が公開している情報によれば、`B0 (0xB) -> P0 (0xC)` で、Spectre v2/v4、Meltdown、L1TF、MDS への策がハードウェア側に追加されている。 (Spectre v2/v4 はソフトウェアと合わせての対策) [^intel-step]  

[^intel-step]: [Affected Processors: Transient Execution Attacks & Related Security Issues by CPU](https://software.intel.com/security-software-guidance/processors-affected-transient-execution-attack-mitigation-product-cpu-model)


| Coffee Lake(-S)[^8th-9th-doc] | Stepping |
| :-- | :--: |
| S (U0) | 0xA (10) |
| S (B0) | 0xB (11) |
| S (P0) | 0xC (12) |
| H (U0) | 0xA (10) |
| H (R0) | 0xD (13) |

<!--
| U (D0) | 0xA (10) |
-->

[^8th-9th-doc]: [8th and 9th Generation Intel® Core™ Processor Family Specification Update](https://cdrdv2.intel.com/v1/dl/getContent/338025)

しかし今回追加されたコメントでもややこしいでも *Kaby Lake* 周りをすべてカバーできているとは言えず、*Kaby Lake* にはまだ AMD GPU とコラボした *Kaby Lake-G* 、LGA2066ソケットに対応したエンスージアスト向けの *Kaby Lake-X* が存在するからだ。それらは `Stepping: 9` となっている。  

さらには *Comet Lake-U* に *Whiskey Lake-U* が紛れ込んでいる。  
*Comet Lake-U* の 1つに **Core i3-10110U** が存在するが、[InstLatx64](https://github.com/InstLatx64) 氏が収集、公開しているデータによれば `Family: 0x6, Model: 0x8E, Stepping: 0xC` となっており [^i3-10110u]、これは *Comet Lake-U* ではなく *Whiskey Lake-U* のものと一致する。  
しかし ark.intel.com では *Comet Lake* としており [^ark-10110u]、iGPU の DeviceID (`0x9B41/0x9BCA/0x9BCC`) も *Comet Lake* とされている。[^cml-did]  
マーケティング、GPU的には *Comet Lake* とされるが、CPUID (`Family, Model, Stepping`) 的には *Whiskey Lake-U* という、曖昧で複雑な CPU だ。  
*Coffee Lake(-S)* は脆弱性対策で、*Kaby Lake-U* と *Amber Lake-Y / Coffee Lake-U / Whiskey Lake-U* は Intel 10nmプロセスの遅延で複雑化した CPU 達とも言えるかもしれない。  

[^cml-did]: [include/pci_ids/i965_pci_ids.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/include/pci_ids/i965_pci_ids.h#L210)
[^i3-10110u]: [InstLatx64/index.htm at 17ee800ec643319215d0b9cb0c59c578ca805e7d · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/17ee800ec643319215d0b9cb0c59c578ca805e7d/index.htm#L548)
[^ark-10110u]: [Intel® Core™ i3-10110U Processor (4M Cache, up to 4.10 GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/196451/intel-core-i3-10110u-processor-4m-cache-up-to-4-10-ghz.html)

## Palm Cove {#cnl}

パッチでは *Cannon Lake* についても触れており、CPUマイクロアーキテクチャ名 *Palm Cove* をコメントに記している。  
*Palm Cove* というのは以前からネット上では知られていたが、OSS へのパッチ、それも補足コメントという形とはいえ、Intel が公式的に *Palm Cove* の名を出したのは初めてな気がする。  

 > 		+#define INTEL_FAM6_CANNONLAKE_L		0x66	/* Palm Cove */
 >
 > {{< quote >}} [LKML: Peter Zijlstra: [PATCH] x86/cpu: Resort and comment Intel models](https://lkml.org/lkml/2021/3/15/2071) {{< /quote >}}

ただ、パッチ内で Peter Zijlstra 氏は、*Cannon Lake (Palm Cove)* は Linux Kernel の CPU性能解析ツール `perf` コマンドのサポートがされていない、とコメントしており、それは `intel-family.h` に *Cannon Lake* のためのマクロを残す必要性が薄いことを意味する。  
*Palm Cove* の名がようやく? Intel より出されたは、将来的には GPUドライバーのように、コードから *Cannon Lake* のための部分が削除される可能性も出てきた。  
{{< link >}} [Cannon Lake を追う | Coelacanth's Dream](/posts/2020/09/29/intel-cnl-chase/) {{< /link >}}

## Sapphire Rapids == Golden Cove? / Willow Cove? {#spr}

パッチでは他に、*Tiger Lake* 、*Sapphire Rapids* 、*Alder Lake* の CPUマイクロアーキテクチャも補完しているが、*Sapphire Rapids* が *Alder Lake* と同じ *Golden Cove* ではなく、*Tiger Lake* と同じ *Willow Cove* とされている。  
しかし、個人的には Peter Zijlstra 氏の間違いではないかと考えている。  

 > 		+#define INTEL_FAM6_TIGERLAKE_L		0x8C	/* Willow Cove */
 > 		+#define INTEL_FAM6_TIGERLAKE		0x8D	/* Willow Cove */
 > 		+#define INTEL_FAM6_SAPPHIRERAPIDS_X	0x8F	/* Willow Cove */
 > 		+
 > 		+#define INTEL_FAM6_ALDERLAKE		0x97	/* Golden Cove / Gracemont */
 > 		+#define INTEL_FAM6_ALDERLAKE_L		0x9A	/* Golden Cove / Gracemont */
 > {{< quote >}} [LKML: Peter Zijlstra: [PATCH] x86/cpu: Resort and comment Intel models](https://lkml.org/lkml/2021/3/15/2071) {{< /quote >}}

まず、*Alder Lake* は Core (Big) 側に *Golden Cove アーキテクチャ* を採用することが明言されている。  
そして、*Alder Lake* も *Sapphire Rapids* も 128-bit/256-bit幅の `AVX_VNNI` 命令に対応しており、他にも `CLDEMOTE / PTWRITE / Architectural LBRs / HLAT / SERIALIZE` 等の最新の命令における対応も共通している。  
また、*Sapphire Rapids* は、用途に 5G基地局等を想定した `AVX512_FP16/HFNI` 命令に対応しており、それは *Golden Cove* の特徴として挙げられた Network/5G Perf の文言と一致する。  
{{< link >}} [Intel Sapphire Rapids は AVX512_FP16 をサポート | Coelacanth's Dream](/posts/2021/01/11/intel-spr-avx512_fp16/) {{< /link >}}
さらには、上で少し触れた `perf` コマンドの対応に関して、*Alder Lake (Golden Cove)* の PMU (Performance Monitoring Unit) は *Sapphire Rapids* のそれと近いというコメントがされている。[^perf-golden_cove]  
そういう訳で *Sapphire Rapids* の CPUマイクロアーキテクチャは *Golden Cove* だと考えているが、果たして。  

[^perf-golden_cove]: [perf/x86/intel: Add Alder Lake Hybrid support · kliang2/perf@83afcae](https://github.com/kliang2/perf/commit/83afcae9c7081c8d64a1343262c3e659bf8262c0)

{{< ref >}}
 * [Intel、次世代CPUアーキテクチャ「Sunny Cove」の概要を明らかに ～Willow Cove、Golden Coveと進化予定、Atomのロードマップも更新 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1158093.html)
    * [[画像] Intel、次世代CPUアーキテクチャ「Sunny Cove」の概要を明らかに ～Willow Cove、Golden Coveと進化予定、Atomのロードマップも更新(3/15) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1158/093/html/003.jpg.html)
{{< /ref >}}
