---
title: "AMD、ROCm v5.2 と rocWMMA ライブラリをリリース"
date: 2022-06-29T19:13:43+09:00
draft: false
categories: [ "Software", "AMD", "GPU" ]
tags: [ "ROCm", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD より ROCm v5.2 がリリースされた。  
リリースノートは v5.1 から [AMD Documentation - Portal](https://docs.amd.com/) のみに公開されるようになり、また最近の更新内容からリリースノートは vX.Y の更新時に公開されるようになった。  

 * [ROCm™ v5.2](https://docs.amd.com/category/ROCm_v5.2)
    * [What’s New in This Release](https://docs.amd.com/bundle/ROCm-Release-Notes-v5.2/page/What%E2%80%99s_New_in_This_Release.html)

主な更新内容は HIP (Heterogeneous-Compute Interface for Portability) のメモリ管理に関する API の更新、GPU Device側で実行される kernel 内での malloc (device-side malloc) のサポート、ライブラリの更新となる。  
そして *CDNA 系アーキテクチャ* でサポートされている `MFMA (Matrix-Fused-Multiply-Add)` 命令を使いやすくするためのライブラリ、rocWMMA がリリースに含まれるようになった。  

 * [ROCmSoftwarePlatform/rocWMMA: rocWMMA](https://github.com/ROCmSoftwarePlatform/rocWMMA)

## rocWMMA {#rocwmma}
rocWMMA は `MFMA` 命令を使いやすくするのと同時に、CUDA WMMA を用いたコードとのポータビリティを強化するためのライブラリとなる。  

 > 		/**
 > 		 * \mainpage
 > 		 *
 > 		 * ROCWMMA is a C++ library for facilitating GEMM, or GEMM-like 2D matrix multiplications
 > 		 * leveraging AMD's GPU hardware matrix cores through HIP.
 > 		 * Specifically, the library enhances the portability of CUDA WMMA code to
 > 		 * AMD's heterogeneous platform and provides an interface to use underlying
 > 		 * hardware matrix multiplication (MFMA) units.
 > 		 * The ROCWMMA API exposes memory and MMA (Matrix Multiply Accumulate) functions
 > 		 * that operate on blocks, or 'fragments' of data appropriately sized for
 > 		 * warp (thread block) execution.
 > 		 * ROCWMMA code is templated for componentization and for providing ability to
 > 		 * make compile-time optimizations based on available meta-data.
 > 		 * This library is an ongoing Work-In-Progress (WIP).
 >
 > {{< quote >}} [rocWMMA/rocwmma.hpp at rocm-5.2.0 · ROCmSoftwarePlatform/rocWMMA](https://github.com/ROCmSoftwarePlatform/rocWMMA/blob/rocm-5.2.0/library/include/rocwmma/rocwmma.hpp) {{< /quote >}}

現状では *CDNA 系アーキテクチャ*、*CDNA 1/gfx908/MI100/Arcturus* と *CDNA 2/gfx90a/MI200/Aldebaran* のみをサポートしているが、LLVM へのパッチから *GFX11/RDNA 3* は `MFMA` とは別の行列積和演算命令 (`WMMA (Wave Matrix Multiply-accumulate)`) をサポートすることが公開されており、将来的に *GFX11/RDNA 3* も rocWMMA にサポートが追加されるのではないかと思われる。  
{{< link >}} [GFX11/RDNA 3 では行列積和演算命令、WMMA (Wave Matrix Multiply-accumulate) をサポート | Coelacanth's Dream](/posts/2022/06/29/gfx11-wmma-inst/) {{< /link >}}
`MFMA (CDNA)` と `WMMA (GFX11/RDNA 3)` との違いには、主に対応するデータフォーマットと行列レイアウト (*GFX11/RDNA 3* は 16x16x16 で固定) が挙げられる。  
