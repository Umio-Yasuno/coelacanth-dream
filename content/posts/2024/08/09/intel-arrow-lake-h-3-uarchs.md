---
title: "3種類の CPU コアを持つ Intel Arrow Lake-H"
date: 2024-08-09T08:52:24+09:00
draft: false  
categories: [ "Intel", "CPU" ]
tags: [ "Arrow_Lake", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Linux Kernel における PMU (Performance Monitoring Unit) に *Arrow Lake-H* のサポートを追加するパッチが Intel の Dapeng Mi 氏によって公開されている。  
そのパッチのコメントにて Dapeng Mi 氏は *Arrow Lake-H* は *Lion Cove (P-core), Skymont (E-core)* に加え、*Crestmont (E-core)* を持つことを明らかにした。  

 >         ArrowLake-H is a specific variant of regular ArrowLake. It shares same
 >         PMU features on lioncove P-cores and skymont E-cores with regular
 >         ArrowLake except ArrowLake-H adds extra crestmont uarch E-cores.
 >
 > {{< quote >}} [[PATCH 0/4] Enable PMU for ArrowLake-H - Dapeng Mi](https://lore.kernel.org/lkml/20240808140210.1666783-1-dapeng1.mi@linux.intel.com/) {{< /quote >}}

*Crestmont* は *Meteor Lake* に E-core, SoC タイルに搭載される LP-E core に採用されていたマイクロアーキテクチャとなる。  
最近の Intel CPU はマイクロアーキテクチャに割り当てられた Native Model ID をサポートしており、`CPUID.1AH.EAX[31:0]` から読み取ることができる。  
パッチではそれを利用して PMU の初期化処理等を実装している。  

 >         +
 >         +/*
 >         + * CPUID.1AH.EAX[31:0] uniquely identifies the microarchitecture
 >         + * of the core. Bits 31-24 indicates its core type (Core or Atom)
 >         + * and Bits [23:0] indicates the native model ID of the core.
 >         + * Core type and native model ID are defined in below enumerations.
 >         + */
 >
 > {{< quote >}} [[PATCH 3/4] perf/x86/intel: Support hybrid PMU with multiple atom uarchs - Dapeng Mi](https://lore.kernel.org/lkml/20240808140210.1666783-4-dapeng1.mi@linux.intel.com/) {{< /quote >}}
 >
 >         +enum atom_native_id {
 >         +	cmt_native_id           = 0x2,  /* Crestmont */
 >         +	skt_native_id           = 0x3,  /* Skymont */
 >          };
 >
 > {{< quote >}} [[PATCH 3/4] perf/x86/intel: Support hybrid PMU with multiple atom uarchs - Dapeng Mi](https://lore.kernel.org/lkml/20240808140210.1666783-4-dapeng1.mi@linux.intel.com/) {{< /quote >}}

また、今回のパッチで *Arrow Lake-H (CPUID Model: 0xC5)* と *Arrow Lake-S (CPUID Model: 0xC6)* とでサポートする拡張命令が異なる理由が明らかになった。  
*Arrow Lake-S* は *Arrow Lake-H* がサポートする拡張命令に加えて、`AVX-VNNI-INT16, SHA512, SM3, SM4` といった命令もサポートすることが以前からドキュメントやコンパイラへのパッチで公開されていた。  
恐らく *Arrow Lake-H* はハイブリッドアーキテクチャの制約として、*Lion Cove, Skymont* はサポートしていても、*Crestmont* がサポートしていない命令は制限されているのだと思われる。  

なお *Arrow Lake-H* の対応命令範囲が *Alder Lake, Meteor Lake* と一致しない。  
ドキュメント、コンパイラへのパッチによれば *Arrow Lake-H* は *Meteor Lake* の範囲に加え、`UINTR, AVXIFMA, AVXVNNIINT8, AVXNECONVERT, CMPCCXADD` といった命令をサポートしている。  
一見不可解に思えるが、このことについては *Crestmont* は元々それらの命令をサポートしており、*Meteor Lake* では *Redwood Cove (P-core)* 側がサポートしていないために制限されていたのだと説明できる。  
実際、*Crestmont* のみで構成されている *Sierra Forest* ではそれらの命令をサポートするとされている。  

| Native Model ID | Type: Core<br>(0x40) | Type: Atom<br>(0x20) |
| :--             | :--:       | :--:       |
| Lakefield | 0x0 (SNC)  | 0x0 (TNT)  |
| Alder Lake      | 0x1 (GLC)  | 0x1 (GRT)  |
| Raptor Lake     | 0x1 (GLC)  | 0x1 (GRT)  |
| Meteor Lake     | 0x2 (RWC)  | 0x2 (CMT)  |
| Lunar Lake, Arrow Lake-S | 0x3 (LNC) | 0x3 (SKT) |
| Arrow Lake-H | 0x3 (LNC) | 0x2 (CMT), 0x3 (SKT) |

{{< ref >}}
 * [x86 Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)
 * [Redwood Cove の Native Model ID が更新 | Coelacanth's Dream](/posts/2022/10/27/intel-redwood_cove-nid/)
{{< /ref >}}
