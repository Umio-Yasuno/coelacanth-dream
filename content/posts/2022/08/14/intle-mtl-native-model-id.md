---
title: "Native Model ID が更新される Crestmont と更新されない Redwood Cove"
date: 2022-08-14T15:31:57+09:00
draft: false
categories: [ "Intel", "Hardware", "CPU" ]
tags: [ "Meteor_Lake", "Lakefield", "Alder_Lake", "Raptor_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は PMU (Performance Monitoring Unit) で検出したハードウェアイベントをデコードそ、計測に用いるため、ハードウェアイベントの生データと概要をマッピングした JSONファイルを <https://download.01.org/perfmon/> で公開している。  
すでに *Meteor Lake-M/P (Family: 0x6, Model: 0xAA)* 用のファイルも公開されており、アーキテクチャ名が P-Core (`Type: Core`) は *Redwood Cove*、E-Core (`Type: Atom`) は *Crestmont* であることが明かされている。  
同時に `mapfile.csv` に `Native Model ID` の記載も追加されており、*Redwood Cove* は *Golden Cove* と同じ `0x00000001`、*Crestmont* は `0x00000002` となる。  

## Native Model ID {#native-model-id}
`Native Model ID` は `CPUID (Leaf:0x1A)` 命令から取得できるハイブリッドアーキテクチャに関する情報であり、各マイクロアーキテクチャを一意に識別するための ID となる。  
`CPUID (Leaf:0x1)` 命令から取得できる `Family, Model, Stepping` とは関係がなく、`Family, Model, Stepping` は CPU 全体の ID なのに対し、`Native Model ID` はマイクロアーキテクチャの ID だと言える。  

`CPUID (Leaf:0x1A)` 命令からは他に `CPU Type[31:24]` も取得でき、`Type: Atom` に `0x20`、`Type: Core` に `0x40` が割り当てられている。  

現在 Intel CPU でハイブリッドアーキテクチャを採用し、`Native Model ID` が公開情報となっている CPU には、*Lakefield, Alder Lake, Raptor Lake, Meteor Lake* があり、それぞれの `Native Model ID` は以下の表のようになっている。  
*Lakefield* の `Native Model ID` は InstLatx64 氏が公開している CPUID dump 結果から確認した。  

| Native Model ID | Type: Core | Type: Atom |
| :--             | :--:       | :--:       |
| Lakefield[^lkf] | 0x0        | 0x0        |
| Alder Lake      | 0x1        | 0x1        |
| Raptor Lake     | 0x1        | 0x1        |
| Meteor Lake     | 0x1        | 0x2        |

`Native Model ID` がマイクロアーキテクチャに対して付けられるユニークな ID ということを考えれば、*Raptor Lake* は *Alder Lake* と同じ *Golden Cove (Core)* と *Gracemont (Atom)* の構成を取り、*Meteor Lake* はコードネームは違うが `Native Model ID` が *Golden Cove* と同じ *Redwood Cove (Core)* と、*Gracemont (Atom)* から ID が更新された *Crestmont* を採用していることになる。  
以上は Intel が公式に公開している情報とドキュメントに記載された仕様から推測できる内容である。  
Intel は *Meteor Lake* で新たにサポートする命令を発表していないため、*Golden Cove* と *Redwood Cove* のマイクロアーキテクチャが同じということは現状矛盾しない。  
しかし、*Meteor Lake* に関しては正式リリース前の早期に公開された情報であること、それと `Nativi Model ID` はドキュメントにはあるが、`Native Model ID` によって具体的に分岐するようなコードを今の所 Linux Kernel 等で確認できていないため、抜けがあるかもしれない。  
あくまでもオタクの推測とメモ書き。  

[^lkf]: [InstLatx64/GenuineIntel00806A1_Lakefield_CPUID.txt at 382580dc9f5fbed23cb1945c6b4a259a49cab484 · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/382580dc9f5fbed23cb1945c6b4a259a49cab484/GenuineIntel/GenuineIntel00806A1_Lakefield_CPUID.txt)

{{< ref >}}
 * [InstLatx64/GenuineIntel00806A1_Lakefield_CPUID.txt at 382580dc9f5fbed23cb1945c6b4a259a49cab484 · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/382580dc9f5fbed23cb1945c6b4a259a49cab484/GenuineIntel/GenuineIntel00806A1_Lakefield_CPUID.txt)
 * [InstLatx64/GenuineIntel0090672_AlderLake_CPUID01.txt at 229ebf299f6cda5f85e1934a391e34cbc80da00d · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/229ebf299f6cda5f85e1934a391e34cbc80da00d/GenuineIntel/GenuineIntel0090672_AlderLake_CPUID01.txt)
 * [[PATCH 0/3] x86: Add initial support to discover Intel hybrid CPUs - Ricardo Neri](https://lore.kernel.org/lkml/20201002201931.2826-1-ricardo.neri-calderon@linux.intel.com/)
 * [Top-down Microarchitecture Analysis Method](https://www.intel.com/content/www/us/en/develop/documentation/vtune-cookbook/top/methodologies/top-down-microarchitecture-analysis-method.html)
 * [intel/event-converter-for-linux-perf](https://github.com/intel/event-converter-for-linux-perf)
 * <https://download.01.org/perfmon/>
{{< /ref >}}
