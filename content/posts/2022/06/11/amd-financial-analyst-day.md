---
title: "AMD Financial Analyst Day 個人的まとめ"
date: 2022-06-11T04:18:05+09:00
draft: false
categories: [ "Hardware", "AMD", "CPU", "GPU", "APU" ]
tags: [ "CDNA_3", "gfx940", "Zen_4", "Zen_5", "GFX11" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD Financial Analyst Day で発表された 2024年までのロードマップ、それと将来の CPU/GPU/APU の一部詳細のまとめ。  

 * [AMD Details Strategy to Drive Next Phase of Growth Across $300 Billion Market for High-Performance and Adaptive Computing Solutions :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1078/amd-details-strategy-to-drive-next-phase-of-growth-across)
 * [Financial Analyst Day :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/financial-analyst-day)

{{< pindex >}}
 * [CDNA 3](#cdna_3)
    * [Infinity Cache](#infinity-cache)
    * [4th Gen Infinity Architecture](#4th-gen-if)
    * [Unified Memory APU Architecture](#datacenter-apu)
    * [New Math Format](#tf32-fp8)
 * [RDNA 3](#rdna_3)
 * [Zen 4](#zen_4)
{{< /pindex >}}

## CDNA 3 {#cdna_3}
*CDNA 3 アーキテクチャ* では 5nmプロセス、3Dチップレットパッケージング技術、次世代 Infinity Cache、4th Gen Infinity Architecture、Unified Memory APU Architecture、New Math Formats が導入される。  

### Infinity Cache {#infinity-cache}
Intel *Ponte Vecchio* では Base Tile に含む L2キャッシュ 144MB に加え RAMBO Cache Tile を搭載し、NVIDIA *H100* では L2キャッシュ 50MB を搭載するなど、他社のデータセンター向け GPU は巨大なメモリキャッシュを搭載してきているのに対し、*CDNA 2/MI200* は *CDNA 1/MI100* と同じ L2キャッシュ 8MB を保っていた。  
*CDNA 3* での Infinity Cache 導入によって、データセンター向け GPU として同様の方向に進むと同時に、キャッシュサイズで追い付くと思われる。  

### 4th Gen Infinity Architecture {#4th-gen-if}
2th Gen Infinity Architecture では GPU間の XGMI (Infinity Fabric) 接続によるマルチGPU性能の向上、3th Gen では CPU-GPU間の XGMI接続により統合メモリ空間を実現した。  
4th Gen ではチップレットアーキテクチャを推し進め、CPU/GPU/IO、それ以外の機能を持つチップレットもパッケージレベルで統合が可能になる。  

*Instinct MI300* では CPU/GPU/Infinity Cache/HBM が 3Dチップレットパッケージング技術により統合される。  

### Unified Memory APU Architecture {#datacenter-apu}
Unified Memory APU Architecture は、パッケージレベルで統合された CPU と GPU がメモリを共有することを指している。  
*CDNA 2 アーキテクチャ* と 3th Gen Infinity Architecture ではメモリ空間は統合されているが、CPU側は DDR4メモリを、GPU側は HBM2メモリをそれぞれ持つという構成だった。  
*Instinct MI300* では CPU と GPU がパッケージ内の HBM メモリ (恐らくは HBM3) を実際に共有する。また、CPU側には *Zen 4 アーキテクチャ* が採用される。  

Unified Memory APU Architecture により CPU-GPU間の余分なメモリコピーを減らすことができ、また GPU を PCIeカードや OAM (OCP Accelerator Module) への実装するのと比べて省電力となる。  

Intel も CPU と GPU を 1つのパッケージ (ソケット) に統合する *Falcon Shores* の存在を発表している。  
*Falcon Shores* では CPU/GPU Tile が最大 4基、また CPU/GPU どちらかのみの構成が可能であることが示されているが、*Instinct MI300* も同様かは今回明かされていない。  
{{< link >}} [Intel、次世代の HPC 向け GPU となる Rialto Bridge を公開 | Coelacanth's Dream](/posts/2022/06/02/intel-rialto-bridge/#falcon-shores) {{< /link >}}
*Intel Falcon Shores* は 2024年を目標としているのに対し、*AMD Instinct MI300* は 2023年の出荷を計画しており、*the world’s first data center APUs* を謳っている。  

### New Math Format {#tf32-fp8}
New Math Format の詳細について発表内では明かされていないが、少なくとも FP8フォーマットと TF32フォーマットのことを指しているのではないかと思われる。  

*CDNA 2/Instinct MI250X* と比較して 8倍の AI学習性能という点において、*CDNA 2/Instinct MI250X* は FP16フォーマットを用い、*CDNA 3/Instinct MI300* は FP8フォーマットを用いた時の性能としており、このことから FP8フォーマットに対応していると考えられる。  

TF32フォーマットへの対応については、LLVM への *gfx940* 対応追加において明かされており、新たな行列演算命令の一部が TF32 (XF32) ファーマットを用いる。  
{{< link >}} [gfx940 で新たにサポートされる命令と XF32フォーマット | Coelacanth's Dream](/posts/2022/03/19/amd-gfx90a-gfx940-diff/) {{< /link >}}

## RDNA 3 {#rdna_3}
*RDNA 3 アーキテクチャ* では 5nmプロセス、チップレットアーキテクチャ、CU の再設計、グラフィクスパイプラインの最適化、次世代 Infinity Cache が導入され、*RDNA 2 アーキテクチャ* からワットパフォーマンスが 50%以上向上するとしている。  

## Zen 4 {#zen_4}
*Zen 4 アーキテクチャ* では 5nmプロセスが採用され、L2キャッシュは 1MB となり、AI向け命令と AVX512命令への対応が行われ、IPC は 8-10% 向上、シングルスレッド性能は >15% 向上、ワットパフォーマンスは >25% 向上するとしている。  

2024年を計画している *Zen 5 アーキテクチャ* ではフロントエンド部の再設計に加え、wide issue、バックエンド部の改良、推論と学習処理への最適化が行われるとしている。  
*AMD Zen 系 アーキテクチャ* は *Zen 3 アーキテクチャ* までフロントエンド部、バックエンド部は基本的な構成は変えず、キャッシュ構成や各所に改良を重ねることで IPC を向上させてきた。*Zen 4 アーキテクチャ* の詳細は発表されておらず、コンパイラに最適化情報もまだ追加されていないが、IPC の向上幅から *Zen 4 アーキテクチャ* も同様の延長線上にあるのではないかと思われる。  
*Zen 5 アーキテクチャ* ではその流れが変わり、大幅な変更が施されていることが期待できる。  

## コードネーム {#codename}

 * Phoenix Point
    * Zen 4 + RDNA 3 + AIE (AI Engine, XDNA)
    * 4nm
    * Mobile
    * 2023
 * Strix Point
    * Zen 5 + RDNA 3+ + AIE
    * Adovanced Node
    * Mobile
    * 2024
 * Granite Ridge
    * Zen 5
    * Adovanced Node
    * Desktop
    * 2024

{{< ref >}}
 * [AMD Showcases Industry-Leading Gaming, Commercial, and Mainstream PC Technologies at COMPUTEX 2022 :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1069/amdshowcases-industry-leading-gaming-commercial-and)
{{< /ref >}}

