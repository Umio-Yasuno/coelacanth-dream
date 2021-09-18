---
title: "やっぱり Aldebaran/MI200 はトータル 110CU ?"
date: 2021-09-17T01:47:03+09:00
draft: false
tags: [ "Aldebaran", "gfx90a", "CDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

以前、[MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen) へのプルリクエストに付けられたコメントから *Aldebaran/MI200* GPU が 110CU ではないかと読み取ったが、もっと直接的な記述が MIOpen のコード中に存在していた。  
{{< link >}} [Aldebaran/MI200 はダイあたり 55CU、トータル 110CU か? | Coelacanth's Dream](/posts/2021/09/01/aldebaran-gfx90a-cu/) {{< /link >}}

 > 		        else if(arch == "gfx908")
 > 		        {
 > 		            assert(num_cu == 120);
 > 		        }
 > 		        else if(arch == "gfx1030")
 > 		        {
 > 		            assert(num_cu == 72 || num_cu == 36);
 > 		        }
 > 		        else if(arch == "gfx90a")
 > 		        {
 > 		            assert(num_cu == 110);
 > 		        }
 > 		        else
 > 		            throw std::runtime_error("Invalid Arch Name");
 >
 > {{< quote >}} [MIOpen/conv_fin.hpp at 55fdfad32e7001acd07033e334d95941fcb44f4e · ROCmSoftwarePlatform/MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen/blob/55fdfad32e7001acd07033e334d95941fcb44f4e/fin/src/include/conv_fin.hpp#L103) {{< /quote >}}

上記コード部は GPUデバイスの情報を検証する部分に使われており、アーキテクチャを示す GPUID (`gfxXXX`) とその CU数が一致しない GPU は無効なデバイスとして実行時エラーが発生するようになっている。  

*Aldebaran/MI200* の ShaderEngine 構成等は不明だが、AMD はチップレット技術を GPU に投入する上で単一の GPU と認識できるようにする特許を投稿していること、  
{{< link >}} [プライマリーダイとセカンダリーダイで構成される Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/06/09/aldebaran-primary-secondary/) {{< /link >}}
FP64演算のフルレート実行と FP32演算のパックド実行に対応しているため、合計 CU が *Arcturus/MI100 (120 CU)* より 10基少ない 110CU だとしても、大幅に性能が向上すること、クロックあたりの性能に対するメモリ帯域を維持できる点で、自分は *Aldebaran/MI200* はトータル 110CU (2x 50CU) ではないかと考えている。  



| CDNA | Arcturus/MI100 | Aldebaran/MI200 |
| :-- | :--: | :--: |
| Active CUs | 120 CU | 110 CU?<br> (2x 55 CU?) |
| FP32:FP64 Rate <br> (Arcturus FP32 = 1) | 1 : 0.5 | 2 : 1 ? |
| FP32 Ops/clk<br>([CU] * [SP per CU] * [FP32 Rate] * 2 [Ops]) | 15360 | 28160 ?<br>(2x 14080) ? |
| FP64 Ops/clk<br>([CU] * [SP per CU] * [FP64 Rate] * 2 [Ops]) | 7680 | 14080 ?<br>(2x 7040) ? |
| Memory | HBM2 2.4Gbps  | HBM2e |
| Memory Bus width | 4096-bit | 8192-bit? <br> (2x 4096-bit) |
| Memory Size | 32 GB | 128GB ? <br> (2x 64GB ?) |


#### Arcturus ( gfx908 ) {#arcturus-gfx908}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x7388 | | | |
| 0x738C | | MI100[^5] | |
| 0x738E | | | |
| 0x7390 | | | (Arcturus VF) |

[^5]:<https://github.com/ROCm-Developer-Tools/aomp-extras/blob/f3d316fe64347e697a9789f0f2499fec50024db1/utils/bin/gputable.txt#L1867>

#### Aldebaran ( gfx90a ) {#aldebaran-gfx90a}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x7408 | | | |
| 0x740C | | | |
| 0x740F | | | |
| 0x7410 | | | (Aldebaran VF)[^aldebaran-vf] |

[^aldebaran-vf]: [[PATCH 1/1] drm/amdgpu: Add a new device ID for Aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-April/062817.html)
