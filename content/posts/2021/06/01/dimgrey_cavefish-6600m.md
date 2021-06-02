---
title: "AMD、Dimgrey Cavefish/Navi23ベースの Radeon RX 6600M を発表"
date: 2021-06-01T12:17:32+09:00
draft: false
tags: [ "Dimgrey_Cavefish", "Navi23", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD は現地時間 2021/05/31、Computex 2021 における基調講演の中で、モバイル向け *RDNA 2* GPU **Radeon RX 6600M** を発表した。  
他にも **Radeon RX 6800M/6700M** が発表されているが、**Radeon RX 6600M** に注目する理由は、開発コードネーム *Dimgrey Cavefish/Navi23* をベースにした初の製品とされるからだ。  

 * [AMD Unveils RDNA 2-Based Mobile Graphics, New AMD Advantage Laptops, Broadly Compatible Upscaling Technology and More at Computex 2021 :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1006/amd-unveils-rdna-2-based-mobile-graphics-new-amd-advantage)


{{< pindex >}}
 * [Dimgrey Cavefish/Navi23](#dimgrey_cavefish)
 * [RDNA 2 GPU のトランジスタ数](#transistor)
{{< /pindex >}}

## Dimgrey Cavefish/Navi23 {#dimgrey_cavefish}

**Radeon RX 6600M** は WGP(CU)数 14(28)基、RenderBackend数 8基 (ROP 64基相当)、メモリバス幅 128-bit、Infinity Cache 32MiB という仕様。  

 * [AMD Radeon™ RX 6600M Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6600m#product-specs)

メモリバス幅と Infinity Cache の容量については、過去に Linux Kernel (amd-gfx) へのパッチの中で AMD のソフトウェアエンジニアより公開されてきた *Dimgrey Cavefish/Navi23* の情報と一致する。  
{{< link >}} [Dimgrey Cavefish/Navi23 の Infinity Cache はメモリチャネルあたり 4MiB | Coelacanth's Dream](/posts/2021/03/04/dimgrey_cavefish-4mb-mall-per-ch/#mall-size) {{< /link >}}
{{< link >}} [AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/#dimgrey_cavefish) {{< /link >}}

キャッシュ構成情報から、*Dimgrey Cavefish* は ShaderArray あたり WGP 4基 (CU 8基) を持つと思われるが、**Radeon RX 6600M** は WGP 14基 (CU 28基) という 8の倍数ではない。  
恐らく各 ShaderEngine の WGP 1基をそれぞれ無効化しているのだろう。 *Dimgrey Cavefish* のフルスペックは ShaderEngine (SE) 2基、SE あたりの ShaderArray (SA) 2基、SA あたりの WGP (CU) 数は 4 (8) 基で、総 WGP (CU) 数は最大 16 (32) 基と考えられる。  

*RDNA/2 アーキテクチャ* では ShaderEngine あたりに ShaderArray 2基を持ち、WGP 3基 (CU 6基) の ShaderArray と WGP 4基 (CU 8基) の ShaderArray を組み合わせる非対称な ShaderEngine 内部構成となるが、こうした構成が *RDNA/2 アーキテクチャ* では許容されている。  
*RDNA 2 アーキテクチャ* では ShaderEngine 内部の ShaderArray が非対称となる場合の性能効率を向上させる改善が加えられており、[RadeonSI](/tags/radeonsi) ドライバーの例では、Pixel Shader を処理する際、ShaderArray が非対称な構成では規模の小さい方に処理速度が引っ張られて効率が悪化してしまう。  
そこで小さい方の ShaderArray が持つ WGP (CU) 数に合わせて Wave (スレッドの塊) を発行、実質対称的な ShaderArray として扱うことで効率の悪化を防ぐ。  
あぶれる WGP (CU) が最低でも 1基出ることになるが、その分消費電力をセーブすることができ、それをクロック向上に回すことも可能だ。  
{{< link >}} [AMD RDNA 2 ノート　―― 非対称な ShaderArray、gpu_info、XSS 【2020/09/23】 | Coelacanth's Dream](/posts/2020/09/23/rdna_2-note-2020-09-23/#shader-array) {{< /link >}}

## RDNA 2 GPU のトランジスタ数 {#transistor}

**Radeon RX 6600M** の製品ページにはトランジスタ数も記載されており、これまでに登場した *Sienna Cichlid/Navi21* 、*Navy Flounder/Navi22* と比較した表が以下。  

| RDNA 2 | Sienna Cichlid/Navi21 | Navy Flounder/Navi22 | Dimgrey Cavefish/Navi23 |
| :-- | :--: | :--: | :--: |
| Shader Engine | 4 | 2 | 2? |
| WGP (CU) | 40 (80) | 20 (40) | 16? (32?) |
| RB+ | 16 | 8 | 8 |
| Memory Bus | 256-bit | 192-bit | 128-bit |
| Infinity Cache | 128 MiB | 96 MiB | 32 MiB | 
| Transistor | 26.8 B | 17.2 B | 11.1 B |

*Dimgrey Cavefish* は *Navy Flounder* の約8割という WGP (CU) 数に対し、トランジスタ数は約6割となっている。  
トランジスタ数のスケール具合で言えば、メモリバス幅 32-bit とそれに対応した Infinity Cache を減らし、WGP (CU) は *Sienna Cichlid* から半分にまで減らした *Navy Flounder* と同程度だ。  
トランジスタを多く使用する SRAM である Infinity Cache を、*Dimgrey Cavefish* ではメモリチャネルあたり 4 MiB に抑え、メモリバス幅も同時に減らしたことで総合的には 32 MiB と他に比べて小さい容量になったことが影響していると思われる。  

*Dimgrey Cavefish* と *Navy Flounder* を比較すると、ShaderEngine数と RenderBackend (RB+) は同数であり、*Dimgrey Cavefish* はメモリ性能を抑える代わりに WGP (CU)、RenderBackend は多めに搭載してピーク性能を高めているとも捉えられる。  
*RDNA 2* GPU がサポートしているハードウェアレイトレーシング機能は、内部の命令では BVH (Bounding VolumeHierarchy) データをメモリからフェッチする仕様になっており、*Dimgrey Cavefish* のそうした向きはレイトレーシング性能に影響が大きそうな気もするが、レイトレーシングはまだハイエンド GPU でなければ性能的に厳しいと考えていることの現れかもしれない。  
{{< link >}} [AMD、RDNA 2 ISA Reference Guide を公開 | Coelacanth's Dream](/posts/2020/12/08/rdna_2-isa-doc/) {{< /link >}}

