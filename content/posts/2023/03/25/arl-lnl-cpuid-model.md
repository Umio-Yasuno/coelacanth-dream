---
title: "Arrow Lake と Lunar Lake-M の CPUID Model、Arrow Lake の GPU DeviceID"
date: 2023-03-25T17:15:29+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Arrow_Lake", "Lunar_Lake", "CPUID" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

## CPUID Model {#mod}

Intel の Tony Luck 氏により、Linux Kernel に *Arrow Lake* と *Lunar Lake-M* の CPUID Model 情報を追加するパッチが投稿された。  
*Arrow Lake* の CPUID Model は `0xC6 (198)`、*Lunar Lake-M* は `0xBD (189)` とされている。  
CPUID Model 情報を集約している `intel-family.h (arch/x86/include/asm/intel-family.h)` では主に、接尾辞がないものをデスクトップ向け、`_L, _N, _P` をモバイル向けとしている。  

そのため今回追加された *Arrow Lake* の CPUID Model はデスクトップ向けと思われる。*Lunar Lake-M* の `_M` については記載がないが、*Meteor Lake-M/P* の例から、おそらくモバイル向けだろう。  

 >         +#define INTEL_FAM6_LUNARLAKE_M		0xBD
 >         +
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add Lunar Lake M - Tony Luck](https://lore.kernel.org/lkml/20230208172340.158548-1-tony.luck@intel.com/) {{< /quote >}}
 >
 >         +#define INTEL_FAM6_ARROWLAKE		0xC6
 >         +
 >
 > {{< quote >}} [[PATCH] x86/cpu: Add model number for Intel Arrow Lake processor - Tony Luck](https://lore.kernel.org/lkml/20230324195932.241441-1-tony.luck@intel.com/) {{< /quote >}}

 * [[PATCH] x86/cpu: Add Lunar Lake M - Tony Luck](https://lore.kernel.org/lkml/20230208172340.158548-1-tony.luck@intel.com/)
 * [[PATCH] x86/cpu: Add model number for Intel Arrow Lake processor - Tony Luck](https://lore.kernel.org/lkml/20230324195932.241441-1-tony.luck@intel.com/)

CPUID Model 情報の追加によって Linux Kernel 内でそれら CPU の識別が可能になる。主に PMU (Performance Monitoring Unit) 部のサポートや、相性問題が発生する場合にはプラットフォームの検出のために使われる。  
別に CPUID Model 情報が無いからと言って Linux Kernel の起動が出来ない訳ではない。  

Intel® Architecture Instruction Set Extensions Programming Reference などのドキュメントに先んじて CPUID Model 情報が追加されたが、Tony Luck 氏は追加する理由として Intel がすでにロードマップで発表していることを挙げている。  
*Arrow Lake* は Intel 20A プロセスで製造されるタイル (チップレット、ダイ) を搭載し、2024年に登場予定。*Lunar Lake* については製造プロセスも登場時期も Intel は発表していない。  

## Arrow Lake-S/P の GPU DeviceID {#arl-did}
Intel の Garg, Nemesa 氏による *Arrow Lake-P* の GPU DeviceID を追加するパッチも投稿されている。  
パッチのタイトルやコメントでは *Arrow Lake-S* にも触れられているが、実際に追加されているのは *Arrow Lake-P* の DeviceID のみ。  

 * [[PATCH dii-client 0/3] Adding Device Ids for ARL-S and ARL-P](https://lore.kernel.org/lkml/20230301112445.2207064-1-nemesa.garg@intel.com/T/#u)

パッチにて、*Meteor Lake* と *Arrow Lake* の GPU 部はわずかな違いがあるだけで、基本的な動作は同じだと Garg, Nemesa 氏はコメントしている。  
*Meteor Lake, Arrow Lake* ではタイルアーキテクチャを採用しているため、GPU タイルについては共通のタイル、もしくは同じアーキテクチャとなることが考えられる。  

 >         Arrow Lake behaves the same as Meteor Lake with just minor differences.
 >         Add definitions for ARL_P. Update MTL and ARL definitions with new
 >         device IDs for ARL-S and ARL-P.
 >
 > {{< quote >}} [[PATCH dii-client 0/3] Adding Device Ids for ARL-S and ARL-P](https://lore.kernel.org/lkml/20230301112445.2207064-1-nemesa.garg@intel.com/T/#u) {{< /quote >}}

また、このパッチは intel-gfx メーリングリストには投稿されていないため、正式版のパッチでは内容が変更されている可能性がある。  

{{< ref >}}
 * [A New Era of Chipmaking to Meet the World’s Demand for Compute](https://www.intel.com/content/www/us/en/newsroom/news/hot-chips-34-new-era-chipmaking.html)
 * [Intel Technology Roadmaps and Milestones](https://www.intel.com/content/www/us/en/newsroom/news/intel-technology-roadmaps-milestones.html)
{{< /ref >}}
