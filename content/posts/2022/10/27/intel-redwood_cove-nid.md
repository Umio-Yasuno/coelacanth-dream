---
title: "Redwood Cove の Native Model ID が更新"
date: 2022-10-27T12:12:13+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Meteor_Lake", "CPUID" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

以前に *Meteor Lake* に採用される *Redwood Cove (Core, P-Core)* と *Crestmont (Atom, E-Core)* の存在と、それらマイクロアーキテクチャを識別するために割り当てられる NativeModelID について取り上げた。  
その時点では、*Redwood Cove* の NativeModelID は *Golden Cove* と同じ `0x1`、*Crestmont* は *Gracemont* から更新された `0x2` が割り当てられているとされていたが、情報が更新され、*Redwood Cove* の NativeModelID が `0x2` となった。  

NativeModelID の値に関する情報元は、主に Intel が公開している TMA (Top-down Microarchitecture Analysis), PMU (Performance Monitoring Unit) のイベント情報を記述したマップファイルとなる。  
マップファイルは <https://download.01.org/perfmon/> で公開されていたが、最近になって Github レポジトリ [intel/perfmon](https://github.com/intel/perfmon) に移行した。  

## Redwood Cove {#rwc}
[intel/perfmon](https://github.com/intel/perfmon) に
Intel の Ed Baker 氏により、マップファイルを更新するプルリクエストが投稿されており、そこでは *Meteor Lake-S* の `CPUID Model (0xAC)` を追加すると同時に、*Redwood Cove* の NativeModelID が `0x2` に更新されている。  

 * [mapfile: Update MTL entries by edwarddavidbaker · Pull Request #25 · intel/perfmon](https://github.com/intel/perfmon/pull/25)

 >         --- a/mapfile.csv
 >         +++ b/mapfile.csv
 >         @@ -152,4 +152,6 @@ GenuineIntel-6-BF,v1.16,/ADL/events/alderlake_uncore_experimental.json,uncore ex
 >          GenuineIntel-6-BE,v1.16,/ADL/events/alderlake_gracemont_core.json,core,,,
 >          GenuineIntel-6-BE,v1.16,/ADL/events/alderlake_uncore.json,uncore,,,
 >          GenuineIntel-6-AA,v1.00,/MTL/events/meteorlake_crestmont_core.json,hybridcore,0x20,0x000002,Atom
 >         -GenuineIntel-6-AA,v1.00,/MTL/events/meteorlake_redwoodcove_core.json,hybridcore,0x40,0x000001,Core
 >         +GenuineIntel-6-AA,v1.00,/MTL/events/meteorlake_redwoodcove_core.json,hybridcore,0x40,0x000002,Core
 >         +GenuineIntel-6-AC,v1.00,/MTL/events/meteorlake_crestmont_core.json,hybridcore,0x20,0x000002,Atom
 >         +GenuineIntel-6-AC,v1.00,/MTL/events/meteorlake_redwoodcove_core.json,hybridcore,0x40,0x000002,Core
 >         
 >
 > {{< quote >}} [mapfile: Update MTL entries by edwarddavidbaker · Pull Request #25 · intel/perfmon](https://github.com/intel/perfmon/pull/25) {{< /quote >}}

NativeModelID についてプルリクエスト内では特に触れられていないが、*Meteor Lake* の初期サンプリングでは *Redwood Cove* が NativeModelID に `0x1` を返すようになっていたか、単なるタイプミスだったのだと思われる。  

| Native Model ID<br>(2022-Oct-27) | Type: Core<br>(0x40) | Type: Atom<br>(0x20) |
| :--             | :--:       | :--:       |
| Lakefield[^lkf] | 0x0 (SNC)  | 0x0 (TNT)  |
| Alder Lake      | 0x1 (GLC)  | 0x1 (GRT)  |
| Raptor Lake     | 0x1 (GLC)  | 0x1 (GRT)  |
| Meteor Lake     | *0x2 (RWC)*  | 0x2 (CMT)  |

[^lkf]: [InstLatx64/GenuineIntel00806A1_Lakefield_CPUID.txt at 382580dc9f5fbed23cb1945c6b4a259a49cab484 · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/382580dc9f5fbed23cb1945c6b4a259a49cab484/GenuineIntel/GenuineIntel00806A1_Lakefield_CPUID.txt)

ある意味不自然だった点が解消された訳だが、未だ NativeModelID の更新基準はよく分からないままだ。  
先日 **Intel® Architecture Instruction Set Extensions Programming Reference** が更新され、*Sierra Forest (Atom), Grand Ridge (Atom?), Granite Rapids (Core)* それぞれに追加される命令が公開されたが、*Meteor Lake* については明記されておらず、それらの命令に対応しないと考えられる。  
**Intel® Architecture Instruction Set Extensions Programming Reference** を基に、*Meteor Lake* のサポートを追加するパッチが GCC や Clang/LLVM に投稿されたが、対応命令範囲は *Alder Lake (Golden Cove + Gracemont)* と同じだとされている。[^gcc]  

*Sierra Forest, Grand Ridge* では対応命令に `AVX-IFMA, AVX-NE-CONVERT, AVX-VNNI-INT8` 命令が追加される。  
*Meteor Lake* がそれらに対応しない理由としては、*Sierra Forest, Grand Ridge* が *Crestmont* の次世代 Atom 系マイクロアーキテクチャを採用しているか、*Redwood Cove* がそれら命令に対応しないため、ハイブリッドアーキテクチャの特性上、無効化されていることが考えられる。  

[^gcc]: [[PATCH 2/2] Initial Meteorlake Support](https://gcc.gnu.org/pipermail/gcc-patches/2022-October/603542.html)

今回の更新により、NativeModelID の更新基準、言い換えればマイクロアーキテクチャが異なるとされる基準は対応命令範囲に限らないと言えるのかもしれない。  

{{< ref >}}
 * [Intel® Architecture Instruction Set Extensions Programming Reference](https://www.intel.com/content/www/us/en/content-details/671368/intel-architecture-instruction-set-extensions-programming-reference.html)
 * [Intel® 64 and IA-32 Architectures Software Developer's Manual Volume 2A: Instruction Set Reference, A-L](https://www.intel.com/content/www/us/en/content-details/671199/intel-64-and-ia-32-architectures-software-developer-s-manual-volume-2a-instruction-set-reference-a-l.html)
 * [Native Model ID が更新される Crestmont と更新されない Redwood Cove | Coelacanth's Dream](/posts/2022/08/14/intle-mtl-native-model-id/)
 * [Intel Sierra Forest, Grand Ridge, Granite Rapids でサポートされる新命令 | Coelacanth's Dream](/posts/2022/10/04/intel-ise-rev_46/)
{{< /ref >}}
