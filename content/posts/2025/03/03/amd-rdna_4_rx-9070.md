---
title: "AMD、RDNA 4 アーキテクチャの詳細を発表、そのメモ"
date: 2025-03-03T09:17:14+09:00
draft: false
categories: [ "AMD", "GPU", "Hardware" ]
tags: [ "GFX12", "RDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---
2025/02/28 付で、AMD は *RDNA 4 アーキテクチャ* の詳細と Radeon RX 9000 シリーズを正式発表した。  
発表内容の紹介はメディア記事に任せるとして、個人的に気になった部分をピックアップしていく。  

 * [AMD Unveils Next-Generation AMD RDNA™ 4 Architecture with the Launch of AMD Radeon™ RX 9000 Series Graphics Cards](https://www.amd.com/en/newsroom/press-releases/2025-2-28-amd-unveils-next-generation-amd-rdna-4-architectu.html)
 * [レイトレ性能が2倍になった「Radeon RX 9070」正式発表！ - PC Watch](https://pc.watch.impress.co.jp/docs/news/1666581.html)
 * [AMD Radeon RX 9070 Series Technical Deep Dive - Complete Slide Deck | TechPowerUp](https://www.techpowerup.com/review/amd-radeon-rx-9070-series-technical-deep-dive/7.html)
 * [RDNA 4世代初のGPU「Radeon RX 9070」シリーズは3月6日に世界市場で発売。4KゲームではRX 7900 GREより20～42％も上回る](https://www.4gamer.net/games/869/G086962/20250228040/)

## Out Of Order Memory {#ooom}
*RDNA 3* ではメモリーリクエストに対して順番通りに処理して結果を返していたが、この方式だとキャッシュミス等によるレイテンシの長いリクエストによって他のリクエストが遅れる場合があった。  
*RDNA 4* ではメモリーリクエストを順不同で処理可能にしたことでこの問題を解決した。  
この変更により多くのワークロードで性能が向上したとしている。  

## Dynamic register allocation {#dyn-reg-alloc}
*RDNA 3* とそれ以前ではベクタレジスタの割り当ては静的であり、使用する最大量も合わせて割り当てていた。  
*RDNA 4* では部分的にベクタレジスタの割り当てと解放が可能になったとされている。割り当ての最適化により稼働させられる Wave 数も増えたとしている。  

割り当てを待つ条件についてはソフトウェア管理だとしているが、具体的な方法 (データの依存性を示す命令を挿入する等) で動的割り当てが機能するかは不明。  

## Cache {#cache}
RX 9070 /XT では 64MB の Infinity Cache を持ち、メモリチャネルあたり 4MB という点は *RDNA 3* と変わらない。  
3rd Gen と *RDNA 3* から世代の進んだ Infinity Cache を採用しているが、進化点などは不明。  
実効メモリ帯域 (Effective Memory Bandwidth) も執筆時点で公開されていない。  
L2キャッシュは全体で 8MB 持つ。メモリチャネルあたり 512KB と、*RDNA 2/3* のメモリチャネルあたり 256KB から増加している。  
ブロックダイアグラムから GL1キャッシュが消えており、LLVM のドキュメント等で公開されていた通りバッファへと変わったものと思われる。しかし、バッファ化の理由や利点などは明かされていない。  

 * [GL1キャッシュがバッファへと変わる AMD RDNA 4 アーキテクチャ | Coelacanth's Dream](/posts/2024/12/16/rdna_4-gl1-buffer/)

## WMMA {#wmma}
*RDNA 4* ではついに行列演算専用ユニットが追加された。  
*RDNA 3* でも行列演算用の命令 `WMMA (Wave Matrix Multiply-accumulate)` をサポートしているが、内部的には複数のドット積命令に分解して発行する方式だった。  
これにより 16-bit 以下のデータフォーマットのピーク演算性能が *RDNA 3* から大幅に向上しており、CU あたり fp16/bf16 は 2倍、int8/int4 は 4倍となっている。  
疎行列もサポートしており、その場合はさらに 2倍のピーク演算性能となる。  
また、OCP 準拠の fp8/bf8 も新たにサポートした。  

