---
title: "GCC と LLVM に Rocket Lake のオプションが追加"
date: 2021-04-12T14:53:19+09:00
draft: false
tags: [ "Rocket_Lake", "Ice_Lake" ]
# keywords: [ "", ]
categories: [ "Software", "Intel", "CPU" ]
noindex: false
# summary: ""
---

日本では 2021/03/30 に発売が開始された *Rocket Lake(-S)* だが、それから約 2週間経ってオープンソースで開発される C言語コンパイラ GCC とコンパイラバックエンドである LLVM に Intelのソフトウェアエンジニアより、*Rocket Lake* への最適化オプションを追加するパッチが投稿された。  

*Rocket Lake* が対応する x86命令範囲について、GCC へのパッチでは、*Ice Lake (client)* をベースに SGX (Software Guard Extensions) を引いたものだと説明されている。  
SGX については ark.intel.com によると、モバイル向け *Tiger Lake (Family: 0x6, Model: 0x8c)* でもサポートされていない。[^1185g7]  

[^1185g7]: [Intel® Core™ i7-1185G7 Processor (12M Cache, up to 4.80 GHz, with IPU) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/208664/intel-core-i7-1185g7-processor-12m-cache-up-to-4-80-ghz-with-ipu.html)

 * [[PATCH] Add rocketlake to gcc.](https://gcc.gnu.org/pipermail/gcc-patches/2021-April/567894.html)
 * [⚙ D100085 [X86] Support -march=rocketlake](https://reviews.llvm.org/D100085)


## CLWB命令には対応せず {#clwb}

LLVM には同時に *Ice Lake (client)* は `CLWB` 命令をサポートしないとして、それをコンパイラ側にも反映させるパッチが投稿されている。  

`CLWB` は Cache Line Write Back の略であり、任意のキャッシュラインをメモリへ即時に書き戻す (Write Back) 機能が持たされている。これにより、あるコアが読み込みアクセスを行う際、そのアドレスへの変更した (Modified) 状態のキャッシュラインを持ったコアに対して通知を送り、そのコアがキャッシュラインの状態変更や通知への応答を返して、それからメモリに書き戻す、という手間を減らすことができる。  
キャッシュラインをそのまま保持することも可能であり、キャッシュミスが発生する可能性を減らす最適化にも用いられる。  
また、`CLWB` 命令は不揮発性メモリにおけるデータの永続化のためにも使われる。  

不揮発性メモリ (Intel Optane Persistent Memory) をサポートする関係か、Intel CPU では *Skylake (server)* から `CLWB` 命令はサポートされている。  
ただし *Cannon Lake* ではサポートせず、*Ice Lake (client)* がコンシューマ向けとしては初めてサポートするとされ、[Intel® 64 and IA-32 Architectures Optimization Reference Manual (May 2020)](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html) でも *Ice Lake (client)* での新機能として紹介されているが、  
実際にはサポートしていないらしく、それを反映させたのがリンク先のパッチとなる。機能的に持たないのか、それとも無効化されているのかは不明。  

*Ice Lake (client)* の `CLWB` 命令の非サポートについては、GCC には 2020/06 [^gcc-icl-clwb]、Intel が公開している x86命令のエンコーダー/デコーダー [intelxed/xed](https://github.com/intelxed/xed) には 2020/11 にそのことを反映させるパッチが投稿されており[^xed-icl-clwb]、LLVM だけ何故か遅れている状況にあった。  
*Tremont* (Atom系アーキテクチャ) と *Tiger Lake* では正式に `CLWB` 命令をサポートしている。*Ice Lake* という括りでも、サーバー向けである *Ice Lake (server)* では当然サポートされている。  
ただ不思議なことに *Ice Lake (client)* と同じマイクロアーキテクチャのコア (*Sunny Cove*) と *Tremont* を搭載するハイブリッドプロセッサ *Lakefield* でもサポートされているようだ。 (Source: InstLatx64) [^lkf-clwb]  


[^gcc-icl-clwb]: [[PATCH] x96: Remove PTA_CLWB from PTA_ICELAKE_CLIENT](https://gcc.gnu.org/pipermail/gcc-patches/2020-June/548857.html)
[^xed-icl-clwb]: [ICL: update CLWB availability · intelxed/xed@b47f2b4](https://github.com/intelxed/xed/commit/b47f2b480492d0a58595e3a2bd9879ae4f3a6332)
[^lkf-clwb]: [InstLatx64/GenuineIntel00806A1_Lakefield_LC_InstLatX64.txt at e833cd79ce0aab79df0d2879b14e01d4edd359b7 · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/e833cd79ce0aab79df0d2879b14e01d4edd359b7/GenuineIntel/GenuineIntel00806A1_Lakefield_LC_InstLatX64.txt#L8)

 * [[X86] Remove FeatureCLWB from FeaturesICLClient · llvm/llvm-project@5cb47be](https://github.com/llvm/llvm-project/commit/5cb47be4104558b2a69dc3df667dbe046bdcce6d)
 * [x96: Remove PTA_CLWB from PTA_ICELAKE_CLIENT · gcc-mirror/gcc@c422e5f](https://github.com/gcc-mirror/gcc/commit/c422e5f81f42a0fc197f0715f4fcd81f1be90bff)

{{< ref >}}
 * [x86 Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)
 * [x86/asm: Add support for the CLWB instruction · torvalds/linux@d9dc64f](https://github.com/torvalds/linux/commit/d9dc64f30abe42f71bc7e9eb9d38c41006cf39f9)
 * <https://simplecore-gar.intel.com/swdevconf-jp/wp-content/uploads/sites/10/2017/10/Japan-Dev-Con_naoyuki-mori_Track-C_-1610.pdf>
 * [第 2 世代インテル® Xeon® スケーラブル・プロセッサーの技術概要 | iSUS](https://www.isus.jp/others/gen2-xeon-processor-scalable-family-technical-overview/)
{{< /ref >}}


