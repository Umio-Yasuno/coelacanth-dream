---
title: "Intel Sierra Forest, Grand Ridge, Granite Rapids でサポートされる新命令"
date: 2022-10-04T15:56:58+09:00
draft: false
categories: [ "Software", "Intel", "CPU" ]
tags: [ "CPUID", "Sierra_Forest", "Grand_Ridge", "Granite_Rapids", "Meteor_Lake", "Emerald_Rapids" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel より、*Intel® Architecture Instruction Set Extensions Programming Reference* Revision -046 が公開された。  
今回のアップデートでは、Atom系コア (E-Core) のみの構成となる Xeon プロセッサ *Sierra Forest*、Core系コア (P-Core) のみの構成の *Granite Rapids*、それから Intel 公式からは初めて出てきたコードネームである *Grand Ridge* の CPUID Model とそれぞれでサポートする新命令の情報が追加されている。  
*Grand Ridge* は対応する新命令が *Sierra Forest* と近いことから、Atom系と思われる。  
*Sierra Forest, Grand Ridge, Granite Rapids* で共通してサポートする新命令はなく、また *Sierra Forest, Grand Ridge* でも命令範囲が完全には一致していない。  

*Sierra Forest* と *Granite Rapids* については Intel 3 プロセスが採用し、ロードマップ上では 2024年に位置することが発表されている。  

 * [Intel® Architecture Instruction Set Extensions Programming Reference](https://www.intel.com/content/www/us/en/content-details/671368/intel-architecture-instruction-set-extensions-programming-reference.html)

また、命令とそれをサポートするプロセッサを集めたテーブルの `AVX512_VP2INTERSECT` 命令の行に、*Tiger Lake* 以外ではサポートされていないとする記述が追加された。  
`AVX512_VP2INTERSECT` 命令は *Tiger Lake (Willow Cove)* からサポートされているはずだが、GCC と LLVM では Intel ISE の記述に沿う方向を見せている。[^llvm] [^gcc]  
今後の Intel プロセッサでは `AVX512_VP2INTERSECT` のサポートを廃止、あるいは実装しない方針なのだろうか。  

[^llvm]: [[X86] Remove AVX512VP2INTERSECT from Sapphire Rapids. · llvm/llvm-project@566c277](https://github.com/llvm/llvm-project/commit/566c277c64f8f76d8911aa5fd931903a357ed7be)
[^gcc]: [[PATCH] Remove AVX512_VP2INTERSECT from PTA_SAPPHIRERAPIDS](https://gcc.gnu.org/pipermail/gcc-patches/2022-October/603329.html)

## AVX-[IFMA, NE-CONVERT, VNNI-INT8] {#avx}
*Golden Cove (P-Core), Gracemont (E-Core)* では `AVX512_VNNI` 命令の 128/256-bit 版とも言える `AVX_VNNI` 命令を新たにサポートした。  
そして *Sierra Forest, Grand Ridge* では `AVX_IFMA, AVX_NE_CONVERT, AVX_VNNI_INT8` 命令をサポートする。  

`AVX_IFMA (Integer Fused Multiply Add Instructions)` 命令は `AVX_VNNI` 命令と同様、`AVX512_IFMA` 命令の 128/256-bit 版だとされる。  

`AVX_NE_CONVERT` 命令は FP16/BF16 - FP32 の変換命令となる。  
AVX512 では `AVX512_BF16` で BF16 フォーマットに、`AVX512-FP16` で FP16 フォーマットに対応している。  
`AVX512_BF16` と `AVX512-FP16` には変換命令に加え、各種演算命令も含まれているが、`AVX_NE_CONVERT` はその中から FP16/BF16 - FP32 との変換命令の 128/256-bit 版のみに対応したものとなる。  

`AVX_VNNI_INT8` 命令は *Multiply and Add Unsigned and Signed Bytes With and Without Saturation* と説明されているが、`AVX_VNNI` 命令 (`VPDPBUSD[S]`) との違いは `VPDPB[SU,UU,SS]D[,S]` 命令があること、ソースオペランドのデータフォーマットの組み合わせが増やされていることなのだろうか？ 正直自信がない。  
x86, AMD64, x86-64 向けの C++ JITアセンブリ Xbyak の開発者である herumi 氏の記事と、Intel の Srinivas Putta 氏 (nivas-x86) のコメントでは、`AVX_VNNI` では `<uint8_t>x<int8_t>` の組み合わせのみをサポートするが、`AVX_VNNI_INT8` は他の組み合わせ (`<uint8_t/int8_t>x<uint8_t/int8_t>`) もサポートするのが違いだとしている。[^zenn]  

[^zenn]: [Granite Rapids/Sierra ForestのAMX/AVXの新命令](https://zenn.dev/herumi/articles/granite-rapids-sierra-forest)

これらの命令は *Sierra Forest, Grand Ridge* でサポートされている。  
`AVX (128/256)` 系命令の拡張を Atom系で行っていることから、まだ Atom系が `AVX512` 系命令をサポートするつもりがないとも捉えられる。  
また、サポートするプロセッサに *Meteor Lake* が入っていないが、*Meteor Lake* に採用される *Crestmont (E-Core)* と *Sierra Forest, Grand Ridge* はマイクロアーキテクチャが異なるか、*Meteor Lake* では *Redwood Cove (P-Core)* 側が `AVX_IFMA, AVX_NE_CONVERT, AVX_VNNI_INT8` 命令に対応しないため無効化していることが考えられる。  
 
## RAO-INT {#rao-int}
`RAO-INT (AADD, AAND, AOR, AXOR)` 命令セットは `ADD, AND, OR, XOR` 命令といった基本的な命令に Atomic 操作を追加した命令となる。  
`RAO-INT` 命令の Memory Ordering は、`LFENCE, SFENCE, MFENCE` 命令と組み合わせた場合よりも弱いものだとしている。  

`RAO-INT` 命令は *Grand Ridge* でサポートされるとしており、*Sierra Forest, Granite Rapids* は含まれていない。  
このことから *Sierra Forest* と *Grand Ridge* は近くはあるが、異なるマイクロアーキテクチャであることが考えられる。  

`RAO-INT` 命令では、プロセッサが特定のキャッシュレベルのみに結果を書き込むことでキャッシュ可用性 (キャッシュヒット率？) を最適化することがあるとしており、L2キャッシュを複数のコアで共有する構成を取る Atom系プロセッサをある程度想定した命令ではないかと思われる。  

## PREFETCHIT0/1 {#prefetch-code}
`PREFETCHIT0/1` 命令はデータではなくコード (命令) のプリフェッチを行い、命令キャッシュの最適化を目的とした命令となる。  
`PREFETCHIT0/1` は *Granite Rapids* でサポートされる。  

## AMX-FP16 {#amx-fp16}
*Sapphire Rapids* では `AMX` 系命令に `AMX-BF16, AMX-TILE, AMX-INT8` 命令をサポートしていた。*Granite Rapids* ではそこに FP16 データフォーマット向けの `AMX-FP16` 命令が追加される。  

今回のアップデートでは *Emerald Rapids* には触れられていないが、*Emerald Rapids* は *Sapphire Rapids* と同じ Intel 7 プロセスが採用することが発表されており、マイクロアーキテクチャの拡張は行われないのかもしれない。  
しかし、*Raptor Lake* では *Alder Lake* から改良された Intel 7 プロセスが採用されており、*Emerald Rapids* でも同様に改良版の Intel 7 プロセスで製造されることが考えられる。  
 
| Family 0x6 | CPUID Model |
| :--        | :--:        |
| Alder Lake-S | 0x97      |
| Alder Lake-M/P | 0x9A    |
| Alder Lake-N   | 0xBE    |
| Raptor Lake-S  | 0xB7    |
| Raptor Lake-P  | 0xBA    |
| Raptor Lake-S (Alder Lake-R?) | 0xBF |
| Meteor Lake-?  | 0xB5 |
| Meteor Lake-S  | 0xAC |
| Meteor Lake-M/P | 0xAA |
| Sapphire Rapids | 0x8F |
| Emerald Rapids  | 0xCF[^lkml] |
| Granite Rapids  | 0xAD, 0xAE |
| Sierra Forest | 0xAF |
| Grand Ridge | 0xB6 |

[^lkml]: [[PATCH] x86/cpu: Add several Intel server CPU mode numbers - Tony Luck](https://lore.kernel.org/lkml/20221103203310.5058-1-tony.luck@intel.com/)

{{< ref >}}
 * [後藤弘茂のWeekly海外ニュース](https://pc.watch.impress.co.jp/docs/2008/0407/kaigai434.htm)
 * [サンプルコード: インテル® AVX-512 とインテル® ディープラーニング・ブースト: 組込み関数 | iSUS](https://www.isus.jp/embeded/avx-512-vnni/)
 * [Intel Sierra Forest the E-Core Xeon Intel Needs - ServeTheHome](https://www.servethehome.com/intel-sierra-forest-the-e-core-xeon-intel-needs/)
 * [Granite Rapids/Sierra ForestのAMX/AVXの新命令](https://zenn.dev/herumi/articles/granite-rapids-sierra-forest)
 * [Adding AMX_FP16, AVN_VNNI_INT8 and AVX_NE_CONVERT instructions by nivas-x86 · Pull Request #162 · herumi/xbyak](https://github.com/herumi/xbyak/pull/162)
{{< /ref >}}
