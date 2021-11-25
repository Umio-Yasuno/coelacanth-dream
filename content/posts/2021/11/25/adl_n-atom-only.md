---
title: "Alder Lake-N は Atom系コアのみの構成か"
date: 2021-11-25T12:53:10+09:00
draft: false
tags: [ "Alder_Lake", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Coreboot に、*Alder Lake (Golden Cove + Gracemont)* のようにハイブリットコア構成を採る CPU における CPPC v3 のサポートに向けて、コアタイプを取得するためのパッチが投稿、公開された。  

 * [soc/intel/alderlake, soc/common: Add method to determine the cpu type mask (If4ceb24d) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59360/3)

パッチで追加されたコードの一部が、以前に同じく Coreboot へのパッチ内で存在が明らかにされた *Alder Lake-N* が他バリアントとは異なり、Atom (Small) 系コアのみで構成されることを示している。  
{{< link >}} [N バリアントも存在する Alder Lake | Coelacanth's Dream](/posts/2021/11/16/coreboot-intel-adl_n/) {{< /link >}}

 > 		+uint8_t get_soc_cpu_type(void)
 > 		+{
 > 		+	struct cpuinfo_x86 cpuinfo;
 > 		+
 > 		+	if (cpu_is_hybrid_supported())
 > 		+		return cpu_get_cpu_type();
 > 		+
 > 		+	get_fms(&cpuinfo, cpuid_eax(1));
 > 		+
 > 		+	if (cpuinfo.x86 == 0x6 && cpuinfo.x86_model == 0xBE)
 > 		+		return CPUID_CORE_TYPE_INTEL_ATOM;
 > 		+	else
 > 		+		return CPUID_CORE_TYPE_INTEL_CORE;
 > 		+}
 > 		+
 >
 > {{< quote >}} <https://review.coreboot.org/plugins/gitiles/coreboot/+/3352093f2bb1f6912dd57d19a5e564e8d21ecc26%5E%21/#F0> {{< /quote >}}

上記引用部が該当のコード `get_soc_cpu_type` 関数だが、処理内容は非常にシンプルで、まず実行する CPU がハイブリッドコア構成を採っている場合は CPUコアタイプを取得する `cpu_get_cpu_type` 関数を呼んでいる。  
CPUID では、ハイブリッドコア構成であることを示すビットフラグ (`EAX=0x7, ECX=0 [EDX: Bit15]`) と、CPUコアタイプを示す値 (`EAX=0x1A [0x20: Atom, 0x40: Core]`) が定義されている。  
*Alder Lake-S/P/M* ではその時点で関数はコアタイプを返すが、ハイブリッド構成を採っていない場合は `CPUID (EAX=1)` から `Family, Model, Stepping` を取得、そして `Faily: 0x6, Model: 0xBE` の場合はコアタイプに `Atom (0x20)` を、それ以外は `Core (0x40)` を返す。  

`Family: 0x6, Model: 0xBE` には *Alder Lake-N* が該当し、その `Family, Model, Stepping` を示す CPUID は以前に追加されていた。  

 > 		#define CPUID_ALDERLAKE_S_A0		0x90670
 > 		#define CPUID_ALDERLAKE_A0		0x906a0
 > 		#define CPUID_ALDERLAKE_A1		0x906a1
 > 		#define CPUID_ALDERLAKE_A2		0x906a2
 > 		#define CPUID_ALDERLAKE_A3		0x906a4
 > 		#define CPUID_ALDERLAKE_N_A0		0xb06e0
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/59306/1/src/include/cpu/intel/cpu_ids.h> {{< /quote >}}

ここでの CPUID では、Bit 0-3 が `Stepping`、Bit 8-11 が `Family ID`、Bit 4-7 が `Model ID`、Bit 20-27 が `Extended Family ID`、Bit 16-19 が `Extended Model ID` を表現しており、`0xb06e0` をデコードすると `Family: 0x6, Model: 0xBE, Stepping: 0` となる。  
`Extended Model ID` は 1桁左にずらして (`<< 4`) から `Model ID` と組み合わされる。  

以上のことから、*Alder Lake-N* はハイブリッドコア構成を採らず、Atom系コアのみで構成されると考えられる。  
モバイル向けの `ALDERLAKE_L (Model: 0x9A)` (*Alder Lake-P/M*) とは異なる Display Model ID であることから、*Alder Lake-N* は組み込み向けとして想定されているのではないかと思われる。  
Atom系コアで構成される製品は、[Tremont アーキテクチャ](/tags/tremont) を採用する組み込み向けの *Elkhart Lake* 、モバイル向けの *Jasper Lake* で最後となるという話があったが[^ascii-atom]、方針転換したのかもしれない。  
今回調べ直したところ、特別ハイブリッドコア構成は組み込みに向いていないという訳ではないらしいが[^emb-big-little]、*Alder Lake* においては消費電力やダイサイズから Atomコアのみで構成する方が適していると判断された可能性がある。  

また N バリアントは以前にも使われたことがあり、MacbookPro向けの *Ice Lake* や *Jasper Lake* が当たるが [^icp-n]、CPU ではなく PCH部に使われており、意味合いとしては異なると思われる。  

[^ascii-atom]: [ASCII.jp：最後のAtomとなるChromebook向けプロセッサーのJasper Lake　インテル CPUロードマップ (3/3)](https://ascii.jp/elem/000/004/040/4040489/3/)
[^emb-big-little]: [if07_060.pdf](https://interface.cqpub.co.jp/wp-content/uploads/interface/2014/07/if07_060.pdf)
[^icp-n]: [[PATCH v1 1/1] mei: me: add Ice Lake-N device id. - Andy Shevchenko](https://lore.kernel.org/all/20211001173644.16068-1-andriy.shevchenko@linux.intel.com/)
