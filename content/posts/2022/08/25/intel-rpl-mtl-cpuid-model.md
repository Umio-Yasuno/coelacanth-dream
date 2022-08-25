---
title: "Raptor Lake と Meteor Lake の CPUID Model"
date: 2022-08-25T10:51:47+09:00
draft: false
categories: [ "Intel", "CPU" ]
tags: [ "CPUID", "Raptor_Lake", "Meteor_Lake", "Alder_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Tony Luck 氏により、Linux Kernel (`arch/x86/include/asm/intel-family.h`) に *Raptor Lake-S* と *Meteor Lake-M/P, S* の `CPUID Model` を追加するパッチが投稿されている。  
`CPUID[LEAF=0x1].EAX` から取得できる `CPUID Family, Model, Stepping` は、主にプロセッサやプラットフォームの判定、ステイトの情報や `perf` ツールのための `PMU (Performance Monitoring Unit)` 情報をそれぞれに適用するのに用いられる。  

 * [[PATCH 1/4] perf/x86: Add new Raptor Lake S support - kan.liang](https://lore.kernel.org/all/20220823210129.979394-1-kan.liang@linux.intel.com/)
 * [[PATCH] x86/cpu: Add CPU model numbers for Meteor Lake](https://lore.kernel.org/all/20220824175718.232384-1-tony.luck@intel.com/T/#u)

## Raptor Lake {#rpl}
Linux Kernel (`arch/x86/include/asm/intel-family.h`) にはすでに `INTEL_FAM6_RAPTORLAKE (0xB7)` と `INTEL_FAM6_RAPTORLAKE_P (0xBA)` の `CPUID Model` が追加されていた。  
法則から、接尾辞が付いていない `INTEL_FAM6_RAPTORLAKE (0xB7)` がデスクトップ向けの *Raptor Lake-S* だと考えられ、実際 Bootlog でも *Raptor Lake-S* の `CPUID Model` は `0xB7` となっていた。[^rpl-bootlog]  

[^rpl-bootlog]: [Intel Raptor Lake-S Bootlog | Coelacanth's Dream](/posts/2022/01/07/intel-rpl_s-bootlog/)

それが今回のパッチで `INTEL_FAM6_RAPTORLAKE_S (0xBF)` が新たに追加された。  

 * [[PATCH] x86/cpu: Add new Raptor Lake CPU model number - Tony Luck](https://lore.kernel.org/all/20220823174819.223941-1-tony.luck@intel.com/)

 > 		  *		_X	- regular server parts
 > 		  *		_D	- micro server parts
 > 		  *		_N,_P	- other mobile parts
 > 		+ *		_S	- other client parts
 > 		  *
 > 		  *		Historical OPTDIFFs:
 > 		  *
 > 		@@ -112,6 +113,7 @@
 > 		 
 > 		 #define INTEL_FAM6_RAPTORLAKE		0xB7
 > 		 #define INTEL_FAM6_RAPTORLAKE_P		0xBA
 > 		+#define INTEL_FAM6_RAPTORLAKE_S		0xBF
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add new Raptor Lake CPU model number - Tony Luck](https://lore.kernel.org/all/20220823174819.223941-1-tony.luck@intel.com/) {{< /quote >}}

ここでの接尾辞無しは `regular client parts` を、`_S` は `other client parts` を意味するため、デスクトップ向け *Raptor Lake* として `INTEL_FAM6_RAPTORLAKE{,_S}` の 2種類があると考えれば、若干紛らわしくはあるが不自然ではない。  

ただ引っ掛かるのは、`CPUID Model: 0xBF` は以前に *Alder Lake* としてドキュメントに記載され[^doc]、コンパイラに追加された `CPUID Model` でもあることだ。[^adl]  
その時は *Alder Lake-N (CPUID Model: 0xBE)* の typo ではないかと考えたが、今回のパッチで *Raptor Lake* として追加されたことで、デスクトップ向け *Raptor Lake* ではあるが実質 *Alder Lake* であるのが `CPUID Model: 0xBF` という解釈も可能になる。  
`CPUID Model: 0xBF` は `CPUID Model, Stepping` こそ *Alder Lake-S* と異なるが、構成は *Alder Lake-S* と同じ、リフレッシュモデルのような立ち位置にあるのかもしれない。  

[^doc]: Volume 4: Model-Specific Registers - [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
[^adl]: [Intel Alder Lake と Rocket Lake に追加される CPUID Model | Coelacanth's Dream](/posts/2022/01/07/intel-adl-rkl-new-model/)

また、Tony Luck 氏はパッチ内で、ソフトウェア的な観点では `RAPTORLAKE*` と `ALDERLAKE*` の機能において、2つの間に異なる点はないとコメントしている。  
ここでの機能はあくまでステイトや `PMU` のことを指していると思われる。  

 > 		Note1: Model 0xB7 already claimed the "no suffix" #define for a regular
 > 		client part, so add (yet another) suffix "S" to distinguish this new
 > 		part from the earlier one.
 > 		
 > 		Note2: the RAPTORLAKE* and ALDERLAKE* processors are very similar from a
 > 		software enabling point of view.  There are no known features that have
 > 		model-specific enabling and also differ between the two.  In other words,
 > 		every single place that list *one* or more RAPTORLAKE* or ALDERLAKE*
 > 		processors should list all of them.
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add new Raptor Lake CPU model number - Tony Luck](https://lore.kernel.org/all/20220823174819.223941-1-tony.luck@intel.com/) {{< /quote >}}

## Meteor Lake {#mtl}
*Meteor Lake* の `CPUID Model` は、デスクトップ向けと思われる `INTEL_FAM6_METEORLAKE (regular client parts)` が `0xAC`、モバイル向けである `INTEL_FAM6_METEORLAKE_L` が `0xAC` とされている。  

`INTEL_FAM6_METEORLAKE` については、以前に触れた [intel/dptf ((Intel (R) Dynamic Tuning for Chromium OS)](https://github.com/intel/dptf) にある `CPUID_FAMILY_MODEL_MTL_S` の情報と異なるが、`CPUID_FAMILY_MODEL_MTL_S` は `CPUID Model` が *Ice Lake-D* と被っているため、今回のパッチが正しい情報だと思われる。[^dptf]  

[^dptf]: [N バリアントも存在する Alder Lake | Coelacanth's Dream](/posts/2021/11/16/coreboot-intel-adl_n/)

 * [[PATCH] x86/cpu: Add CPU model numbers for Meteor Lake](https://lore.kernel.org/all/20220824175718.232384-1-tony.luck@intel.com/T/#u)

 > 		+#define INTEL_FAM6_METEORLAKE		0xAC
 > 		+#define INTEL_FAM6_METEORLAKE_L		0xAA
 > 		+
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add CPU model numbers for Meteor Lake - Tony Luck](https://lore.kernel.org/all/20220824175718.232384-1-tony.luck@intel.com/) {{< /quote >}}

自分の主観だが、珍しくパッチ内でリリース予定時期と製造プロセスについても触れられている。  

 > 		---
 > 		These are destined for release in 2023 on Intel 4 process node.
 > 		
 > 		As is the custom, I'm just adding the model numbers so perf, power
 > 		etc. teams can added model specific code to their respective subsystems
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add CPU model numbers for Meteor Lake - Tony Luck](https://lore.kernel.org/all/20220824175718.232384-1-tony.luck@intel.com/) {{< /quote >}}
