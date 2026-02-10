---
title: "RDNA4/GFX12 と同じ WMMA 命令をサポートする RDNA4m/GFX11.7 APU"
date: 2026-02-11T00:26:46+09:00
draft: false
categories: [ "AMD", "APU" ]
tags: [ "GFX1170", "LLVM", "GFX11", "GFX12" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Mirko Brkusanin 氏により RDNA4m/GFX1170 APU に関する新たなパッチが投稿され、RDNA4/GFX12 と同じ WMMA (Wave Matrix Multiply Accumulate) 命令をサポートすることが判明した。  

 * [[AMDGPU] Add WMMA and SWMMAC instructions for gfx1170 by mbrkusanin · Pull Request #180731 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180731)

WMMA 命令は RDNA3/GFX11 と RDNA4/GFX12 で入力データの配置に関して異なる点があり、RDNA3/GFX11 では入力データを複製して配置するため、追加のベクタレジスタを必要とする仕様だった。  
それが RDNA4/GFX12 では必要ではなくなり、効率的な仕様となった。  
そして RDNA4m/GFX1170 がサポートするのは RDNA4/GFX12 と同じ仕様となる。  

 >         +// Some of these are very similar to their base GFX11 counterparts, but they
 >         +// don't require replication of the A,B matrices, so they use fewer vector
 >         +// elements. Therefore, we add an "_gfx12" suffix to distinguish them from the
 >         +// existing builtins.
 >
 > {{< quote >}} [[AMDGPU] Add WMMA and SWMMAC instructions for gfx1170 by mbrkusanin · Pull Request #180731 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180731) {{< /quote >}}

RDNA4m/GFX1170 がサポートする WMMA/SWMMAC (Sparse WMMA) 命令も RDNA4/GFX12 と同じであることから、専用の行列積和演算ユニットとそのスループットは RDNA4/GFX12 と同一である可能性が高い。  

そうなると気になるのは、RDNA4m/GFX1170 APU が RDNA4/GFX12 ではない理由だ。  
同じ機能と命令をサポートするなら、最初から RDNA4/GFX12 系列の GPU ID を割り当てればいい話であり、追加のソフトウェア対応も最低限で済む。  
実装面積を削減するために RDNA4/GFX12 から機能か性能を削ったとして、後考えられるのはレイトレーシングユニットくらいだろうか？  
RDNA4/GFX12 では CU に BVH4 に対応したレイトレーシングユニットを 2基搭載することで、RDNA3/GFX11 から倍のレイトレーシング性能の向上を図っている。  
レイトレーシングユニットの内部を倍の規模にした訳ではないため、モジュール的にレイトレーシングユニット 1基だけにした設計も可能ではあるはずだ。  
