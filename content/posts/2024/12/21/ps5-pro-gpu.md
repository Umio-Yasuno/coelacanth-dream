---
title: "PS5 Pro GPU アーキテクチャの詳細が公開"
date: 2024-12-21T22:28:45+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
# tags: [ "", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

2024/12/19 に PS5 と PS5 Pro のリードアーキテクトである Mark Cerny 氏による、PS5 Pro の GPU アーキテクチャの解説動画が公開された。  
ゲーム機向けのカスタム APU としては珍しく、かなり詳細に触れた動画であったため紹介したい。  

 * [PS5 Pro Technical Seminar reveals new in-depth details on console – PlayStation.Blog](https://blog.playstation.com/2024/12/18/ps5-pro-technical-seminar-reveals-new-in-depth-details-on-console/)
 * [PS5 Pro Technical Seminar at SIE HQ - YouTube](https://www.youtube.com/watch?v=lXMwXJsMfIQ)

## RDNA 2.x {#rdna2_x}
PS5 Pro では RDNA 2.x と呼ぶ、RDNA 2 ISA との互換性を重視しつつ RDNA 3 の機能を取り入れたアーキテクチャを採用している。  
実際、RDNA 3 では RDNA 2 から一部命令のサポートが削除されており (`V_{MAC,MADMK,MADAK}_F32` や `{V_MAD_F32}, V_MAD_LEGACY_F32`)、キャッシュ制御ポリシーを示す DLC, SLC, GLC ビットの意味も変わっている。  
RDNA 3 での大きな変更であった、2x SIMD32 ユニットと VOPD (Dual issue wave32) 命令は取り入れられていない。このことは PS5 Pro の PS5 から見た FP32 ピーク理論性能の増加が WGP (CU) 数の増加率と一致することからも分かる。  
また、ジオメトリパイプにも RDNA 3 の要素が取り入れられている。  

RDNA 3 では WGP (CU) あたりの FP32 理論性能が倍になり、それに合わせてか L0ベクタデータキャッシュや GL1キャッシュ、ベクタレジスタサイズを増やしている。  
既に互換性という理由が語られてはいるが、そうした変更によるダイサイズの増加を嫌った可能性も考えられる。  
2x SIMD32 ユニットによって、処理に適した Wave サイズというのも RDNA 3 では変わっているため、ゲーム機のよるに低レイヤーで GPU に触れられるプログラミングモデルでは導入コストが高かったことも考えられる。  

## レイトレーシング {#rt}
PS5 Pro では PS5 からレイトレーシング機能が大幅に強化された。  
RDNA 3 の BVH (Bounding Volume Hierarchy) 構造体のスタックを管理する機能を取り入れただけでなく、将来の RDNA アーキテクチャで採用される、BVH8 構造体のサポートと交差エンジンの倍化 (8x Ray/Box, 2x Ray/Triangle) が導入されている。  

## 専用命令 {inst}
PS5 Pro では小規模な CNN 向けの 3x3 の畳み込みを行う専用命令が実装されており、8-bit (Int8?) では 300TOPS、16-bit (FP16?) では 67TFLOPS のピーク理論性能となる。  
FP32 ピーク理論性能との比率から、RDNA 3 の WMMA (Wave Matrix Multiply Accumulate) 命令のように内部で複数のドット積命令に分解して発行する方式ではなく、CDNA 系の MFMA (Matrix fused-multiply-add) 命令のように CU 内に専用のユニットを持つ方式に近いと考えられる。  

ベクタレジスタへの柔軟なアクセス可能にする命令と、CNN に必要な計算を処理するための命令は合計で 44個追加されている。  

ROCm でのコンシューマ向け GPU のサポートが進んでいることもあり、将来の RDNA アーキテクチャで同様の小規模な CNN に特化した命令をわざわざ実装する可能性は低いだろう。  
ここは動画内でも "Custom RDNA" と強調されているように、かなりゲーム機向けのアーキテクチャらしいカスタム色が強い部分だと言える。  
