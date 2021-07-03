---
title: "CDNA 2 アーキテクチャでは Unified Register Files を実装"
date: 2021-07-01T15:19:10+09:00
draft: false
tags: [ "Aldebaran", "CDNA_2", "MI200", "gfx90a" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

*CDNA 2 アーキテクチャ* を採用すると見られ、コンピュート向けに特化した次世代 AMD GPU *Aldebaran* では Unified Register Files が実装されることが明かされた。ArchVGPR (Architectural Vector General-Purpose Register) と AccVGPR (Accumulation VGPR) が統合され、512エントリのプールを共有する。  
ArchVGPR は通常のベクタレジスタ、AccVGPR は機械学習向けの行列演算命令 `MFMA (Matrix-Fused-Multiply-Add)` を処理する SIMDユニット専用のベクタレジスタとなる。  

 * [Refactor vgpr occupancy calculation for Aldebaran's new hardware capability Unified Register Files by jichangjichang · Pull Request #1371 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1371)

2種のレジスタファイルが統合されたことでレジスタの使用率をより最適化することができ、利用可能エントリ数が増えたことでレジスタ溢れによるキャッシュメモリからの読み込みを減らせるなど、性能低下の要因を減らし、実行性能を向上させられると考えられる。  

## MFMA・miSIMD・AccVGPR

*CDNA アーキテクチャ* は 4-way のドット積を処理するための命令 *MFMA (Matrix-Fused-Multiply-Add)* をサポートし、MFMA 命令は専用の SIMDユニット、miSIMD (Machine Intelligence SIMD) で実行される。  
miSIMDユニットは入力値をすべて MFMA命令を実行する際、乗算を行う 2つのソースオペランドには ArchVGPR か AccVGPR を選択できるが、累積のためのソースオペランドは常に専用のレジスタファイル、AccVGPR から読み込まれ、結果も常に AccVGPR に出力される。  
{{< link >}} [AMD、CDNA1/MI100 ISA Reference Guide を公開 | Coelacanth's Dream](/posts/2020/12/15/cdna-isa-doc/) {{< /link >}}

{{< figure src="/image/2020/12/15/cdna-vgpr.webp" caption="<cite>画像出典: [\"AMD Instinct MI100\" Instruction Set Architecture: Reference Guide - CDNA1_Shader_ISA_14December2020.pdf](https://developer.amd.com/wp-content/resources/CDNA1_Shader_ISA_14December2020.pdf)</cite>" >}}

*CDNA アーキテクチャ* の実装では、通常の SIMDユニット (shSIMD)、ArchVGPR と miSIMDユニット、AccVGPR は分離されていた。  
メモリキャッシュや LDS (Local Data Share) へのデータの入出力には ArchVGPR のみが対応し、AccVGPR は対応していないため、miSIMDユニットの実行結果等は ArchVGPR を経由してデータの入出力を行う必要があった。ArchVGPR と AccVGPR 間のデータ移動には `V_ACCVGPR_{READ|WRITE}` 命令を使う。  
SIMDユニットとレジスタファイルの関係についても、miSIMDユニットは一部のソースオペランド先に ArchVGPR を選択することができるが、shSIMDユニットは AccVGPR を扱うことができない。  

ArchVGPR と AccVGPR が統合されたことで、レジスタ使用率の広い最適化が可能になるだけでなく、AccVGPR が ArchVGPR を経由せずにデータの入出力できるよう制限が緩和される可能性がある。  
ただ AccVGPR の使用にはオフセットを指定する形になっており、レジスタの衝突を防ぐためにも、shSIMDユニットが AccVGPR を直接扱えない点、MFMA命令の結果が常に AccVGPR へ出力する点は変わらないのではないかと思われる。  

*CDNA アーキテクチャ* では miSIMD と AccVGPR を増設する形での実装という *GCN アーキテクチャ* をベースとする部分を多く残す構成だったが、*CDNA 2 アーキテクチャ*, *Aldebaran* GPU で 2種のレジスタファイルが統合されたことで、アーキテクチャの進化が本格的に *GCN アーキテクチャ* からコンピュート向けへ分岐したという印象を受ける。  
また、グラフィクス、ゲーミング向けは *RDNA系アーキテクチャ* が担っているが、それと *CDNA系アーキテクチャ* は完全に分かれている訳ではなく、命令のプリフェッチ機能や L2キャッシュラインのサイズなど、*RDNA系アーキテクチャ* から取り入れた新機能が *CDNA 2 アーキテクチャ* では実装されている。  
{{< link >}} [RadeonSI に Aldebaran GPU のサポートを追加するパッチが投稿される | Coelacanth's Dream](/posts/2021/03/04/aldebaran-umd/#inst-prefetch) {{< /link >}}

