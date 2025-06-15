---
title: "AMD CDNA 4 アーキテクチャを採用した Instinct MI350 シリーズを正式発表"
date: 2025-06-14T22:51:14+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "CDNA_4", "gfx950" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD は現地時間 6/12 付で開催した AMD Advancing AI 2025 にて、AMD CDNA 4 アーキテクチャの詳細とそれを採用した Instinct MI350 シリーズを正式に発表した。  

 * [AMD Advancing AI 2025 Press Kit](https://www.amd.com/en/newsroom/press-kits/advancing-ai-2025.html)
 * [AMD CDNA™ Architecture](https://www.amd.com/en/technologies/cdna.html)
   * [amd-cdna-4-architecture-whitepaper.pdf](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-4-architecture-whitepaper.pdf)
 * [AMD MI350 and CDNA 4 Architecture Launched with ROCm 7 - ServeTheHome](https://www.servethehome.com/amd-mi350-and-cdna-4-architecture-launched-with-rocm-7/)

## CDNA 4 アーキテクチャ {#cdna_4}
CDNA 4 アーキテクチャは前世代から主に低精度フォーマットの行列演算性能と Compute Unit 内の高速なローカルメモリ Local Data Share (LDS) が強化されている。  
具体的には、FP16/BF16/FP8/INT8/INT4 の行列演算性能はクロックあたりで最大 2倍となり、FP6/FP4 のサポートが追加された。  

ただし、前世代、CDNA 3 アーキテクチャにおける FP8 フォーマットは、複数のハードウェアベンダーが参加している OCP (Open Compute Project) で定義されている FP8 には準拠していなかった。  
CDNA 3 アーキテクチャにおける FP8 フォーマットは FP8-FNUZ と呼ばれ、OCP 準拠の FP8 とは無限大をサポートしない、NaN を表現する形式という点で異なっている。  
今回 CDNA 4 アーキテクチャで OCP 準拠の FP8 をサポートしたことで、よりソフトウェアの相互運用性が高まったと言える。  

TF32 フォーマットをサポートする命令が CDNA 4 アーキテクチャで削除されたが、TF32 フォーマット自体はソフトウェアエミュレートで対応するとされている。  

LDS は前世代の 64KiB から 160KiB と、2倍以上のサイズに強化されている。  
前世代では 32バンク (バンクあたり 32-bits エントリが 512基) で 64KiB という構成だったが、CDNA 4 アーキテクチャでは 64バンクに増えている。バンクあたりのエントリ数は公開されていないが、160KiB のサイズから 640基だと思われる。  
また LDS のリード帯域もクロックあたり 256Byte に増やされたとしている。  
他にも LDS からベクタレジスタへのロードと一緒に転置を実行すると思われる命令も CDNA 4 アーキテクチャでは追加されている。[^transpose]  

[^transpose]: [[MLIR][ROCDL] Add ops for LDS read transpose and global to LDS intrinsics by plognjen · Pull Request #123530 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/123530)

キャッシュ構成は前世代からほとんど変わっていない。各キャッシュ階層のサイズとクロックあたりの帯域は前世代と同じとされている。  
ただ L2キャッシュはコヒーレント向けの最適化が施されており、DRAM からの非コヒーレントデータをキャッシュ可能になり、またダーティデータを DRAM に書き戻した後もキャッシュラインを保持するようになったとされている。  

## MI350 シリーズ {#mi350}
MI350 シリーズとしては、より高速でメモリサイズが増えた HBM3E の採用と通信性能が主な強化点となる。  

前世代の MI325X は同じく HBM3E を採用しているが、ピークメモリ帯域は 6TB/s、メモリサイズは 256GB だったのに対し、MI350 シリーズでは 8TB/s、288GB となっている。  

前世代のチップレット構成は各種 I/O やメモリコントローラー、Infinity Cache を統合した IOD 4基、CU と L2キャッシュを統合した XCD を IOD あたり 2基搭載するものだったが、  
MI350 シリーズでは IOD を巨大化し、IOD 2基、XCD を IOD あたり 4基搭載する構成となった。  
チップレット構成と接続方法がシンプルになり、これはチップレット間帯域の増加とレイテンシの削減に寄与している。  

他ボードへの接続に使われる Infinity Fabric Link も高速化され、前世代の最大 896GB/s に対し、最大 1075GB/s となっている。  

XCD は TSMC N3P プロセスで製造され、XCD あたり CU 36基を搭載する。製品としては 32基を有効化した構成で統一されている。  
前世代の XCD は CU 40基を搭載し、内 38 基を有効化した構成であったため、MI350 シリーズの CU 数は前世代より少ないものとなっている。  
動作クロックが向上しているとはいえ、FP32/FP64 (Vector) に関してはクロックあたりの性能が変わっていないため、TBP が同じ 1000W の MI300X/MI325X (81.7 TFLOPS) と MI350X (72.1 TFLOPS) を比較するとピーク性能は 0.88 倍となっている。  
こうした点からも MI350 シリーズはより推論に重きを置いた製品シリーズと言えるだろう。  
