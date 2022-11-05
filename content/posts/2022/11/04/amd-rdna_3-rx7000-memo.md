---
title: "AMD、RDNA 3 アーキテクチャを採用した RX 7900 Series を発表、そのメモ"
date: 2022-11-04T20:24:20+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD は 2022/11/03 付で、事前に予告していたように *RDNA 3 アーキテクチャ* とチップレットデザインを採用した **RX 7900 Series** を発表した。  

 * [AMD Unveils World’s Most Advanced Gaming Graphics Cards, Built on Groundbreaking AMD RDNA 3 Architecture with Chiplet Design :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1099/amd-unveils-worlds-most-advanced-gaming-graphics-cards)
 * {{< youtube id="kN_hDw7GrYA" title="AMD Presents: together we advance_gaming" >}}
 * [AMD Radeon™ RX 7900 XTX | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-7900xtx)
 * [AMD Radeon™ RX 7900 XT | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-7900xt)

発表内容については、ニュースリリースか各種ニュースサイトを読むのが良いと考えているため、ここでは個人的に気になった部分のメモや補足に留める。  

 * [AMD，新世代GPU「Radeon RX 7000」シリーズを発表。第1弾製品はRadeon RX 7900 XTXとRadeon RX 7900 XTで12月13日発売](https://www.4gamer.net/games/660/G066019/20221104001/)
 * [AMD、前世代比2.7倍の性能となった「Radeon RX 7000」 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1452983.html)

## チップレットデザイン {#chiplet}
**RX 7900 Series** はゲーミング向けとして世界初のチップレットデザインを採用した GPU となる。  
1x GCD (Graphics Compute Die) と 6x MCD (Memory Cache Die) を同パッケージに搭載しており、チップレット全体での帯域は 5.3TB/s にもなる。  
GCD-MCD 間が 883.3GB/s としても、**AMD MI250X (CDNA 2)** の GCD-GCD 間が 400GB/s だったのを考えるとチップレット間の帯域は大幅に向上している。  

GCD は 5nmプロセスで製造され、ダイサイズは 306mm2。MCD は 6nmプロセス、37.5mm2 となっている。  
*Navi21* は 7nmプロセス 519mm2、*Navi22* からの計測だが Infinity Cache (4MiB) が約 2.47mm2 だったため、Infinity Cache を MCD に分離したことによる効果は大きいように思う。  

## Unified Compute Unit {#cu}
CU (Compute Unit) は Unified Compute Unit となり、従来から 1.5倍のベクタレジスタ、64 Dual-issue SP、AI Accelerator 2基、第 2世代 Ray Tracing Accelerator で構成される。  

ベクタレジスタの増量については LLVM へのパッチという形ですでに公開されていたが、*RDNA 3 アーキテクチャ* 全体で共通する特徴ではなく、*Navi31/gfx1100* と *Navi32/gfx1101* のみ備える。  
64 Dual-issue SP は詳細が不明で、Wave Controller や Scalar ALU、SIMD Unit がどうなっているかは気になる所である。  
AI Accelerator は行列演算命令の専用ユニットと思われる。  
第 2世代 Ray Tracing Accelerator は CU あたりの性能が 1.5倍になったとしているが、具体的にどう強化されたのか、サイクルあたりの性能や新機能の詳細、活用方法は明らかにされていない。  

## 2.7X AI 性能 {#ai-perf}
*RDNA 3 アーキテクチャ* では *RDNA 2 アーキテクチャ* から AI 命令が追加され、AI スループット性能は 2.7倍になったとしている。  
2.7倍の内訳だが、ニュースリリースの脚注を確認すると **RX 7900 XTX (2.505GHz, Dual-issue 96CU)** と **RX 6900 XT (2.25GHz, 80CU)** をデータフォーマット Bfloat16 (BF16) の計算処理で比較した結果となっている。  
Dual-issue 96CU を 192CU 相当とすると、CU 数の増加とクロックの向上で 2.7倍の性能を達成でき、追加された AI 命令の効果が見えない。  
AI 命令は *RDNA 3 アーキテクチャ* でサポートしているドット積命令と行列積和演算命令 (`WMMA (Wave Matrix Multiply-accumulate`) を指していると思われる。それら命令では *RDNA 2 アーキテクチャ* ではサポートしていなかった BF16 フォーマットに対応した命令が含まれている。  
命令レベルで BF16 をサポートしている *RDNA 3 アーキテクチャ* ならば、FP32 として処理しなければならない *RDNA 2 アーキテクチャ* と比べて高い性能が出せると思うのだが、何故発表では 2.7倍というピーク FP32 演算性能での比較にも当てはまる数字を使ったのかは不明である。  
あるいは *CDNA 系アーキテクチャ* のように、データフォーマットごとのピーク演算性能が出されればはっきりするのだが。  

`WMMA` 命令については、ROCm ライブラリの [ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile) と [ROCmSoftwarePlatform/rocWMMA](https://github.com/ROCmSoftwarePlatform/rocWMMA) においてサポートが進められており、*RDNA 3 アーキテクチャ* のサポートと性能に期待が持てる。  

## Infinity Cache {#ic}
Infinity Cache /MALL (MALL (Memory Access at Last Level) /L3キャッシュは、**RX 7900 Series** ではメモリチャネルあたり 4MiB と、*Navi21/Navi22* よりも規模を減らしており、*Navi23/24* と同じバランスとなっている。  

**RX 7900 XTX** の実効メモリ帯域 (Effective Memory Bandwidth) は Up to 3500GB/s。  
Infinity Cache を含めた実効メモリ帯域については以前に取り上げたが、AMD が内部で計測したキャッシュヒット率が関係してくるため、マーケティング的な都合が入り込むことは否めない。  
キャッシュヒット率にはターゲットとするゲームの解像度も影響してくる。  

`([number of memory channel] x [cache bus width] x [hit rate] x [clock]) + ([memory bw])`  

Infinity Cache 全体のサイズが *Navi21* から縮小されつつも、**RX 6950 XT** の実効メモリ帯域 1793.5GB/s から大きく向上している。  
だが **RX 7900 XTX** の実効メモリ帯域がどの解像度を想定したものなのかは公開されていない。  

## Dual Media Engine, AV1 エンコード {#av1-enc}
**RX 7900 Series** では Dual Media Engine を搭載し、8K60 HEVC の同時デコード/エンコードと AV1 のエンコードにも対応する。  
AV1 エンコードには、Intel GPU は *DG2/Alchemist* で、NVIDIA GPU は *Ada Lovelace* で対応しており、AMD GPU も **RX 7900 Series** で並んだ形となる。  

*Navi21/RDNA 2* でも Media Engine を 2基搭載しており、片方がデコードを、もう片方がエンコードをサポートする非対称構成だった。  
**RX 7900 Series** の Dual Media Engine がデコードとエンコードを同時に処理可能ということならば、同様に非対称構成を採っていることが考えられる。  
Media Engine の非対称構成の利点には、役割を明確に分けることによりパワーゲーティングといった省電力機構を適用しやすくやり、電力効率が向上することが挙げられている。[^navi21-vcn]  

[^navi21-vcn]: [[PATCH 38/42] drm/amd/powerplay: set VCN1 pg only for sienna_cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051564.html)

Media Engine 自体の動作クロックは *RDNA 2 アーキテクチャ* から 1.8倍に向上しているとし、それによって処理性能も向上している。  

一部否定的な書き方になってしまったが、言い換えれば *RDNA 3 アーキテクチャ* と **RX 7900 Series** は前世代から進化しながらも、まだ詳細が公開されていない部分が多くある。  
特に、構成を変更し Dual-issue を実装した Unified Compute Unit、第 2世代となった Ray Tracing Accelerator は、ホワイトペーパーの公開が待たれる。  

{{< ref >}}
 * [追加のベクタレジスタを持つ Navi31/GFX1100 と Navi32/GFX1101 | Coelacanth's Dream](/posts/2022/09/23/some-gfx11-extra-vgpr/)
 * [RDNA 3/GFX11 では行列積和演算命令、WMMA (Wave Matrix Multiply-accumulate) をサポート | Coelacanth's Dream](/posts/2022/06/29/gfx11-wmma-inst/)
 * [GFX11 のサポートに向けた LLVM へのさらなるパッチ ―― True16Bit, Dot8, Dual issue wave32 | Coelacanth's Dream](/posts/2022/05/10/llvm-gfx11-dual-issue/)
 * [Navy Flounder/Navi22 のダイ観察 | Coelacanth's Dream](/posts/2021/12/02/navy_flounder-dieshot/)
 * [RX 6500 XT の実効メモリ帯域 | Coelacanth's Dream](/posts/2022/01/13/rx-6500-xt-effective-bw/)
{{< /ref >}}
