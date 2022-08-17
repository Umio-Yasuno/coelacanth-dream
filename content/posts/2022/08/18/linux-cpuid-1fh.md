---
title: "Linux Kernel に V2 Extended Topology Enumeration の扱いを改善するパッチ"
date: 2022-08-18T00:22:32+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Alder_Lake", "Linux_Kernel", "CPUID" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Zhang Rui 氏により、`CPUID [Leaf=0x1F]` から取得できるトポロジ情報、`V2 Extended Topology Enumeration` の扱いを改善するパッチが投稿されている。  
パッチには、これまで *Alder Lake-S/P/M* の最大コア数の取得に関して、存在してはいたが実際に発生することは無かった問題の修正、*Alder Lake-N* に向けた `Module ID` のサポートが含まれている。  

 * [[PATCH V2 0/8] x86/topology: Improve CPUID.1F handling - Zhang Rui](https://lore.kernel.org/linux-hwmon/20220816051633.17775-1-rui.zhang@intel.com/)

## Extended Topology Enumeration {#ext_topo_enum}
まず `Extended Topology Enumeration` は `CPUID [Leaf=0xB]` と `CPUID [Leaf=0x1F]` を実行すると返されるトポロジ情報であり、その情報を基に `SMT (Thread), Core` の数や ID を取得することができる。  
`CPUID [Leaf=0xB]` は `SMT, Core` の情報を取得できる。Intel CPU は基本的にサポートしており、MAMD CPU も *Zen 2* 世代からサポートしている。  
`CPUID [Leaf=0x1F]` は `CPUID [Leaf=0xB]` のスーパーセットであり、`SMT, Core` に加えて `Module, Tile, Die` レベルの情報が取得できるようになっている。  
Intel CPU では少なくとも *Alder Lake-S/P/M* と *Sapphire Rapids* で `CPUID [Leaf=0x1F]` をサポートしているが、取得できる情報は `CPUID [Leaf=0xB]` と同じ `SMT, Core` までとなっている。AMD CPU は現状 `CPUID [Leaf=0x1F]` をサポートしていない。  

Linux Kernel では `Die` レベルのサポートが以前に追加されているため、*Alder Lake-S/P/M* と *Sapphire Rapids* は実際には `Die` レベルをサポートしている可能性があるが、インターネット上に公開されている情報で `CPUID [Leaf=0x1F, SubLeaf=0x4?]` を実行した結果を見つけられなかった。  

## Alder Lake {#adl}
Linux Kernel では `CPUID [Leaf=0xB, 0x1F]` の情報を活用しているが、ハイブリッドアーキテクチャを採用する *Alder Lake* ではトポロジ情報が従来の CPU より複雑であり、問題が起こる可能性があった。  
*Alder Lake* の P-Core (*Golden Cove*) は SMT をサポートし、1-Core/2-Thread の構成だが、E-Core (*Gracemont*) は 1-Core/1-Thread の構成となる。  
加えて、パッチ以前の Linux Kernel ではコアあたりのスレッド数 `smp_num_siblings` をグローバル変数で管理しており、すべてのコアあたりのスレッド数が同じであることを前提としていた。  
`smp_num_siblings` は `CPU0` の情報を用いており、もしも E-Core が `CPU0` となっていた場合、本来より少ない `smp_num_siblings`、スレッド数として認識されてしまう。  
これまですべての *Alder Lake-S/P/M* プラットフォームで `CPU0` に P-Core が配置されていたのはまったくの幸運によるものだと Zhang Rui 氏はコメントしている。  

 > 		This inconsistency brings several potential issues:
 > 		1. some kernel configuration like cpu_smt_control, as set in
 > 		   start_kernel()->check_bugs()->cpu_smt_check_topology(), depends on
 > 		   smp_num_siblings set by cpu0.
 > 		   It is pure luck that all the current hybrid platforms use Pcore as cpu0
 > 		   and hide this problem.
 > 		2. some per CPU data like cpuinfo_x86.x86_max_cores that depends on
 > 		   smp_num_siblings becomes inconsistent and bogus.
 > 		3. the final smp_num_siblings value after boot depends on the last CPU
 > 		   enumerated, which could either be Pcore or Ecore CPU.
 >
 > {{< quote >}} [[PATCH V2 6/8] x86/topology: Fix max_siblings calculation - Zhang Rui](https://lore.kernel.org/linux-hwmon/20220816051633.17775-7-rui.zhang@intel.com/) {{< /quote >}}

パッチでは、現在のレベルのプロセッサ数を示す `CPUID [Leaf=0xB, 0x1F].EBX` の代わりに、次のレベルのトポロジID を取得するために `x2APIC ID` を右シフトするビット数を示す `CPUID [Leaf=0xB, 0x1F].EAX` を用いることで対応している。  

## Alder Lake-N {#adl-n}
*Alder Lake-N* では *Alder Lake-S/P/M* では欠けていた `Module` レベルをサポートする。  
同時にパッチでは、これまで断片的に公開されてきてはいたが、*Alder Lake-N* が E-Core のみ、2-Module、Module あたり 4-Core の構成であることが明かされている。  

 > 		On Intel AlderLake-N platforms where there are Ecores only, the Ecore
 > 		Module topology is enumerated via CPUID.1F Module level, which has not
 > 		been supported by Linux kernel yet.
 >
 > {{< quote >}} [[PATCH V2 0/8] x86/topology: Improve CPUID.1F handling - Zhang Rui](https://lore.kernel.org/linux-hwmon/20220816051633.17775-1-rui.zhang@intel.com/) {{< /quote >}}
 >
 > 		For example, on an AlderLake-N platform, there are 2 Ecore Modules, which
 > 		has 4 atom cores in each module, in a single package.
 > 		Linux does not support Module level and interprets the Module ID bits as
 > 		package ID and erroneously reports a multi module system as a
 > 		multi-package system.
 >
 > {{< quote >}} [[PATCH V2 4/8] x86/topology: Fix multiple packages shown on a single-package system - Zhang Rui](https://lore.kernel.org/linux-hwmon/20220816051633.17775-5-rui.zhang@intel.com/) {{< /quote >}}

`Module` レベルのサポートにより、`CPUID [Leaf=0x1F]` から取得できる情報のみで、全体では 2-Module, 8-Core、Module あたり 4-Core であることが計算できるようになる。  

ただ、*Alder Lake-S/P/M* で E-Core 2-Module であることを判断するにはどうすればいいのかが少し気になる。  

