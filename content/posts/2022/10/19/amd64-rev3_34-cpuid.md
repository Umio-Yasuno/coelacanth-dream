---
title: "AMD64 Architecture Programmer’s Manual アップデート、CPUID に非対称トポロジ関連の項目が追加"
date: 2022-10-19T01:49:47+09:00
draft: false
categories: [ "Software", "AMD", "CPU" ]
tags: [ "CPUID", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD が公開している AMD64 Architecture Programmer’s Manual が Revision 3.34 にアップデートされ、`CPUID` 命令が返す機能フラグや情報の項目が追加された。  
`CPUID` 命令は、実行した CPU の各種情報を示す値を `EAX, EBX, ECX, EDX` レジスタに出力する命令であり、内容は実行時の `EAX, ECX` レジスタの値で異なる。  
実行時の `EAX, ECX` レジスタは `Leaf, Sub-Leaf` とも呼ばれる。  
CPU の識別子 (Family, Model, Stepping)、機能フラグ、命令のサポート有無、アドレスサイズ等を確認するのにも使われる基本的な命令。  

主な AMD CPU と Intel CPU とで実装の有無が異なる機能もあるが、機能フラグやシステム部で使うことを想定した機能は基本共通している。  
**AMD64 Architecture Programmer’s Manual Processor** に掲載されていないが、**Intel® Architecture Instruction Set Extensions Programming Reference** では定義されている機能が AMD CPU に実装されていることもある。  
AMD CPU のドキュメントとしては、AMD が別に公開している **Programming Reference Manual** の方が優先される。  

**AMD64 Architecture Programmer’s Manual Processor** の内容から、将来の AMD CPU に実装される機能の推測はできるが、それがいつになるかは当然不明であるし、また確実という訳でもないことを留意する必要がある。  

## Fn 8000_0026h - Extended CPU Topology {#ext-topo}
今回のアップデートで CPU トポロジ情報を返す `Fn 8000_0026h - Extended CPU Topology` が追加された。  
同様の機能は Intel CPU では `Fn Bh, Fn 1Fh` に実装されている。  

`Fn 1Fh, Fn 8000_0026h` はどちらも `Fn Bh` をベースにしているが、`Fn 8000_0026h` では `Fn Bh` で Reserved (予約部) とされていた部分に定義を追加している。  
また、`Fn Bh, Fn 1Fh` は主に Intel CPU で実装されてきたこともあり、`Fn 1Fh` は `Fn Bh` の上位互換 (super-set) となっており、トポロジのレベルタイプと値の関係を合わせているが、`Fn 8000_0026h` では異なっている。  
トポロジのレベルタイプは `ECX[Bit15:8]` に定義されている。  

返す CPU トポロジ情報とそのトポロジのレベルは `Sub-Leaf (ECX)` によって変わるが、`Sub-Leaf (ECX)` とトポロジのレベルタイプを示す値が一致する訳ではないことに注意。  

`Fn Bh` は AMD CPU では *Zen 2 アーキテクチャ* の世代から実装されている。  

| Value(ECX[Bit15:8]) - Level Type | Fn Bh | Fn 1Fh | Fn 8000_0026h |
| :--                | :--:  | :--:   | :--:          |
| 0h                 | Reserved | Reserved | Reserved |
| 1h                 | SMT   | SMT    | Core          |
| 2h                 | Core  | Core   | Complex       |
| 3h                 | Reserved | Module | Die        |
| 4h                 | Reserved | Tile   | Socket     |
| 5h                 | Reserved | Die    | Reserved   |

`Fn 8000_0026h` では Intel CPU におけるハイブリッドアーキテクチャ、非対称トポロジに関する情報も定義されている。一部は Intel CPU に合わせた仕様になっているが、割り当てられたビット位置、ビット範囲が異なっているものもある。  
ハイブリッドアーキテクチャの情報は、Intel CPU では `Fn 1Ah` に定義されており、`EAX[Bit31:24]` が CoreType、`EAX[Bit24:0]` が NativeModelID と定義されているが、  
`Fn 8000_0026` では `EBX[Bit31-28]` が CoreType、`EBX[Bit27:24]` が NativeModelID と定義されている。  
また `Fn 8000_0026` では、電力効率のランキング順位を示す PwrEfficiencyRanking が `EBX[Bit23:16]` に定義されている。PwrEfficiencyRanking の値が低いほど、消費電力と性能が低いことを示す。  

非対称トポロジに関する機能フラグは `EAX` レジスタに定義されている。  
AsymmetricTopology (`EAX[Bit31]`) は現在のトポロジレベルにおいて、すべてのトポロジが同じスレッド数 (論理プロセッサ数, NumLogProc) を報告しないことを示す。これはトポロジのレベルタイプに Complex が定義されていることと合わせて、仮定ではあるが 4C/8T と 8C/16T のトポロジが同じ CPU に混在する場合等に立てられるフラグと思われる。  
HeterogeneousCores (`EAX[Bit30]`) は現在のトポロジレベルに異なる CoreType が存在することを示す。  
実質、ハイブリッドアーキテクチャであることを示す機能フラグと言うことができ、同様の機能フラグは Intel CPU では `Fn 7h (EDX[Bit15])` に定義されている。  
EfficiencyRankingAvailable (`EAX[Bit29]`) は先の PwrEfficiencyRanking をサポートしていることを示す機能フラグとなる。  

## CpuidUserDis
`Fn 8000_0021 (EAX[Bit17])` に CpuidUserDis の機能フラグが新たに定義された。  
CpuidUserDis は非特権ソフトウェア (CPL \> 0) での `CPUID` 命令無効化をサポートしていることを示すが、`CPUID` 命令は通常のソフトウェアでも機能フラグの確認のため使われる命令であり、どういったケースを想定しているのかは不明。  

{{< ref >}}
 * [AMD64 Architecture Programmer’s Manual, Volume 3: General-Purpose and System Instructions, 24594 - 24594.pdf](https://www.amd.com/system/files/TechDocs/24594.pdf)
 * [Intel® Architecture Instruction Set Extensions Programming Reference](https://www.intel.com/content/www/us/en/content-details/671368/intel-architecture-instruction-set-extensions-programming-reference.html)
{{< /ref >}}
