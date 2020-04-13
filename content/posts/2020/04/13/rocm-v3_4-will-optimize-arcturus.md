---
title: "ROCm v3.4 は Arcturus への最適化がメインか"
date: 2020-04-13T09:14:36+09:00
draft: false
tags: [ "Radeon", "ROCm", "Arcturus", "gfx908" ]
keywords: [ "", ]
categories: [ "Software", ]
noindex: false
---

[Tensile](https://github.com/ROCmSoftwarePlatform/Tensile)へのプルリクエストから ROCm v3.4 での更新内容が見えてきた。[^1]  
Tensileとは、行列演算やN次元のテンソルの縮約、2つの多次元オブジェクトの乗算といった演算のベンチマークに使われるバックエンドライブラリを作成するためのツール。

[^1]: [ROCm 3.4 milestone merge develop into master by zaliu · Pull Request #922 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/922)

内容としては、行列演算用の命令 MFMA、データ精度 BF16、その他もろもろへの最適化、修正が見られ、  
先2つから ROCm v3.4 は[Arcturus](/tags/arcturus)への対応をより進め、性能を引き出すことがメインと考えられる。  

*Arcturus* での新機能に対応する ROCm v3.2 を待たず、ROCm v3.3 が不意を打つ形でリリースされたことでリリース形体が分かれた。[^2]  

[^2]: [AMD、ROCm v3.3.0 リリース | Coelacanth's Dream](/posts/2020/04/02/amd-rocm-v330-release/)

 * ROCm v3.2 &rarr; ROCm v3.4
 * ROCm v3.3 &rarr; ROCm v3.5?

と *Arcuturs* 向けのリリースと一般ユーザー向けのリリースで分けるのか、  
それとも ROCm v3.4 で合流するか。  
ROCm のパッケージには一般ユーザーもそれなりに使うOpenCLドライバも含まれるため、リリース形体を分けることには一応の意味がある。  
また、*Zen 4 EPYC Genoa* CPU と、時期から *CDNA 2* GPUを採用すると思われるスーパーコンピューター **El Capitan** ではソフトウェアスタックに ROCm の拡張バージョンを使用するとしており、今後開発を進める上で分けることはありえるはずだ。  

 > An enhanced version of the open source ROCm heterogenous programming environment, being developed to tap into the combined performance of AMD CPUs and GPUs, unlocking maximum performance.

 > 引用元: <cite>[Next-Generation AMD EPYC™ CPUs and Radeon™ Instinct GPUs Enable El Capitan Supercomputer at Lawrence Livermore National Laboratory to Break 2 Exaflops Barrier | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/next-generation-amd-epyctm-cpus-and-radeontm-instinct-gpus)</cite>
