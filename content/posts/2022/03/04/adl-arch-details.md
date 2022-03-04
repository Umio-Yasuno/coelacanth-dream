---
title: "Intel、SDM に Golden Cove、Gracemont アーキテクチャの詳細を追加"
date: 2022-03-04T16:58:45+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Alder_Lake", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は Software Developer Manuals (SDM) の 1つ、Intel® 64 and IA-32 Architectures Optimization Reference Manualを更新し、*Alder Lake* のハイブリッドアーキテクチャとそのスケジューリング、*Alder Lake* に採用されているアーキテクチャ、 *Golden Cove (Core)* と *Gracemont (Atom)* の詳細を追加した。  
その中の特に関心のある部分、知識がギリギリ何とか及びそうな部分だけをまとめる。  

* [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html#inpage-nav-5)

## Intel EHFI / Intel Thread Director {#ehfi}
*Alder Lake* が対応する EHFI (Enhanced Hardware Feedback Interface)、マーケティング的には Intel Thread Director と呼ばれるそれは、OS に対してスケジューリングのヒントとなる情報を提供する。  
提供される情報には、電力あるいは熱の制限に基づいた性能と電力効率の相対的なレベルがある。  
{{< link >}} [Linux Kernel に Intel HFI をサポートするパッチ | Coelacanth's Dream](/posts/2022/01/02/intel-hfi/) {{< /link >}}
同様の情報を提供する HFI には、*Alder Lake* だけでなく *Lakefield* も対応している。  

EHFI が HFI と異なるのは、各論理コアで実行しているソフトウェアスレッドの特性を区別する Class がある点で、OS はこれを性能と電力効率のレベルと一緒にヒントとして使うことができる。  
Class がどう区別されるのか、その詳細が今回ドキュメントに追加された。  
Class 0 にはベクトル化されていない整数、浮動小数点のコード、Class 1 には VNNI系命令を除いたベクトル化されたコード、Class 2 には VNNI系命令、Class 3 にはスピン・ウェイト・ループが割り当てられる。  
OS のスケジューラーはこれらを基に、時にはスレッドの移行を行う。  
また、ドキュメントではスピンロックに対して、古い `PAUSE` 命令に代わって、より軽量な `UMWAIT/TPAUSE` 命令の使用を推奨している。  

## Gracemont {#gracemont}
*Gracemont* は *Tremont* の後継であり、多くの改良が全体に盛り込まれている。  

ロード/ストアバッファを増やすと同時に、ロード/ストア実行ポート (AGU, Address Generation Unit) は *Tremont* の 2ポートから 4ポートに増やされている。  

AVX/AVX2 (256-bit) 命令をサポートし、新たな AVX-VNNI (128/256-bit) にも対応する。  
VNNI (Vector Neural Network Instructions, `VPDPBUSD`) 命令は推論処理の高速化が主な用途であり、マーケティング的には Intel DL Boost とも呼ばれている。  
通常では `VPMADDUBSW`, `VPMADDWD`, `VPADDD` という 3つの命令を必要とする INT8 の 畳み込み演算を、VNNI (`VPDPBUSD`) 命令 1つで実行できるのが特徴であり、命令の統合によるキャッシュ効率の向上、必要とする帯域の削減を可能とする。[^vnni] 

ビット操作命令 (BMI1, BMI2, ADX, LZCNT) のサポートも命令も追加され、Intel® Processors based on Gracemont Microarchitecture Instruction Throughput and Latency では MSROM (Microcode sequencer ROM) を用いないネイティブ実装だとしている。  
*AMD Zen 1/2* ではマイクロコード実装、*AMD Zen 3* からネイティブ実装となった PDEP/PEXT 命令も、*Gracemont* では最初からネイティブ実装となっている。  

[^vnni]: [ベクトル・ニューラル・ネットワーク命令はインテル® アーキテクチャーによる INT8 推論を可能に | iSUS](https://www.isus.jp/products/mkl/vnni-enables-inference/)

*Tremont/Gracemont* では 4-Core で L2キャッシュを共有し、クラスターかモジュール、タイルとも呼ばれる構成を採る。  
*Tremont* では L2キャッシュサイズを 1MB から 4.5MB の範囲で選択可能であり、マイクロサーバー向けの *Snow Ridge* では 4.5MB、それ以外の *Japser Lake, Elkhart Lake, Lakefield* では 1.5MB となっていた。  
*Gracemont* ではサイズ範囲が 2MB から 4MB に変更され、*Alder Lake* は 2MB の構成を採っている。  

### Decoder {#decoder}
*Tremont* から 2x3-wide デコードというクラスター構成を *Gracemont* は引き継ぎながら、L1命令キャッシュは 32KB から 64KBに、フェッチ幅をクラスターあたり 16Byte から 32Byte に増やしている。  
またデコーダーのクラスタリングによって発生する負荷分散も、専用のハードウェアバランサーが導入されたことで *Tremont* からボトルネックが軽減されたとする。  
*Tremont* では負荷分散は分岐予測に依存していたため、分岐の少ない長いコード、浮動小数点演算等でコンパイラが適用するループアンローリングのようなケースでは負荷が片方に集中し、ボトルネックとなっていた。  
{{< link >}} [Intel Architecture Day 2021 個人的まとめ　―― Gracemont, Golden Cove, Alder Lake | Coelacanth's Dream](/posts/2021/08/24/intel-arch-day-2021/) {{< /link >}}

### FPU {#fpu}
*Gracemont* では *Xeon Phi* 系を除いて、初めて AVX/AVX2 (256-bit) 命令をサポートする。  
FPU部のデータパスは 128-bit 実装であり、256-bit命令は 2個の 128-bit uops に分割されて実行する。  
*AMD Zen 1* を思い出す方式だが規模は異なり、*AMD Zen 1* は FPU部の実行ポートが 2xFADD + 2xFMUL という構成なのに対し、*Gracemont* は 2x(FADD+FMUL) 構成となる。  

