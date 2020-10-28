---
title: "AMD RDNA 2 GPU 正式発表　―― 各スペック、Infinity Cache、プロセス"
date: 2020-10-29T01:14:51+09:00
draft: false
tags: [ "RDNA_2", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

AMD は現地時間 2020/10/28 付で、*RDNA 2 /GFX10.3* 世代の GPU、**Radeon RX 6000 Series** を正式発表した。  


 * [AMD Unveils Next-Generation PC Gaming with AMD Radeon™ RX 6000 Series – Bringing Leadership 4K Resolution Performance to AAA Gaming :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/978/amd-unveils-next-generation-pc-gaming-with-amd-radeon-rx)
 * [Where Gaming Begins: Ep. 2 | AMD Radeon™ RX 6000 Series Graphics Cards - YouTube](https://www.youtube.com/watch?v=oHpgu-cTjyM)

*RDNA 2 アーキテクチャ* は *RDNA アーキテクチャ* と比較して、電力比性能が 54% 向上しているとし、これは高い動作クロックに向けた設計、全体的な設計の最適化、新設された Infinity Cache による IPC 向上によって実現されている。  
また、CU (Compute Unit) あたりの電力効率は最大 30% 向上し、高い動作クロックに向けた設計によって、同じ消費電力で最大 30% 高いピーククロックを実現したとしている。  

{{< pindex >}}

 * [RX 6000 Series 各スペック](#spec)
 * [Infinity Cache](#if-cache)
 * [プロセス](#process)

{{< /pindex >}}

## RX 6000 Series 各スペック {#spec}

 * [AMD Radeon RX 6800 Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6800#product-specs)
 * [AMD Radeon RX 6800 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6800-xt#product-specs)
 * [AMD Radeon RX 6900 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6900-xt#product-specs)

AMD は **Radeon RX 6000 Series** のダイショット {{< comple >}} 機密保持のため加工されている、あくまでそれっぽい画像と思われるが {{< /comple >}} を公開しており、  
{{< link >}} [AMD Radeon™ RX 6000 Series die shot](https://www.globenewswire.com/NewsRoom/AttachmentNg/56b9f51f-b313-41a3-9fc1-0f1bf766c3d4/en) {{< /link >}}
それを *Navi10* のダイショットと見比べるに[^navi10-dieshot]、Shader Array あたりの WGP(CU) 数は変えずに、Shader Engine の数を 4基に増やしたものと考えられる。  

[^navi10-dieshot]: [AMD Navi10のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/01/22/navi10-dieshot-and-guess/)

**RX 6800** は他 2製品よりも ROP 数が 32基少ないものとなるが、ROP/RB 周りの仕様が *RDNA アーキテクチャ* と同じであるならば無効化する必要は無いはずで、  
製品の差別化のため意図的に無効化しているか、4 Shader Engine の内 1 Shader Engine を無効化して製品化しているのではないかと思われる。  
4 Shader Engine のまま 30WGP を実現するには Shader Engine を非対称としなければならず、これは従来の AMD GPU の構成から外れたものになり、スケジューリングに問題が出ると思われる。そのため、1 Shader Engine を無効化している可能性が高い。  
ただ、それならば 32WGP(64CU)、Shader Engine あたり 8WGP(16CU) とした方が製品化において余裕があり、歩留まり的には有利なように思う。  
30WGP(60CU) という構成では、3 Shader Engine 内の WGP(CU) がすべて有効な必要がある。  

*Ray Accelarator* は CU数と同数であり、レイトレーシングユニットに関しては **Xbox Series X** と同様の構成を取っているものと思われる。  

以下は、今回発表された **Radeon RX 6000 Series**  3製品を比較した表。  

| AMD RDNA 2 | RX 6800 | RX 6800 XT | RX 6900 XT
| :-- | :--: | :--: | :--: |
| WGP(CU) | 30(60) | 36(72) | 40(80) |
| SP | 3840 | 4608 | 5120 |
| Ray Accelerator | 60 | 72 | 80 |
| TMU | 240 | 288 | 320 |
| ROP | 96 | 128 | 128 |
| Game Clock | 1815 MHz | 2015 MHz | 2015 MHz |
| Boost Clock | 2105 MHz | 2250 MHz | 2250 MHz |
| |
| Memory Bus Width | 256-bit | 256-bit | 256-bit |
| Max Memory Size | 16 GB | 16 GB | 16 GB |
| Memory Bandwidth | 512 GB/s | 512 GB/s | 512 GB/s |
| Infinith Cache | 128 MB | 128 MB | 128 MB |
||
| Peak FP16 (TFLOPS) | 32.34 | 41.47 | 46.08 |
| Peak FP32 (TFLOPS) | 16.17 | 20.74 | 23.04 |
||
| Typical GPU Power | 250 W | 300 W | 300 W |

## Infinity Cache {#if-cache}

*Infinity Cache* は今回発表された 3製品には 128MB 搭載されている。  
*Infinity Cache* には 64Byte単位でアクセスでき、アクセスチャネルは 16chとなる。  
16ch というのは GDDR6 256-bit のチャネル数と一致し (GDDR6 はチップあたり 16-bit のメモリチャネルを持つ)、ニュースリリースの *last-level data cache* の文言から、メモリキャッシュとして動作するものと思われる。  
AMD GPU におけるデータキャッシュ階層は、*RDNA アーキテクチャ* では CU内のプライベートキャッシュとなる L0キャッシュ 16KB、その下が Shader Array ごとに持つ L1キャッシュ 128KB、全体でアクセス可能な L2キャッシュ (メモリチャネルあたり 256KB、*Navi10* では計 4MB、*Navi14* では計 2MB) で構成されていたが、  
*RDNA 2 アーキテクチャ* ではそこに *Infinity Cache / L3キャッシュ* 、メモリチャネルあたり 8MB が新設された、ということだろう。  

上述した **Radeon RX 6000 Series** のダイショットでは、動画内で示されていた *Infinity Cache* とされる部分はそれ程大きく占めてはいないように見える。  
AMD によると、*Zen 3 アーキテクチャ* の L3キャッシュデザインをベースにしているとのこと。  

*Infinity Cache* により、GDDR6メモリのみで構成された *RDNA アーキテクチャ* ベースの GPU と比較して、電力あたりのメモリ帯域は最大 2.4倍になるとしている。  
ただこの数値は、AMD が *Infinity Cache* 128MB、GDDR6 256-bit の GPUで 4Kゲーミング実行時に、キャッシュヒートレートを 59% とした場合の値である。  
それも最大の値であり、平均的なヒットレートや性能への影響は不明。  


## プロセス {#process}

**Radeon RX 6000 Series** のベースとなる GPUチップは、**RX 5000 Series** と同じ 7nmプロセスで製造されるとしている。  
それを鵜呑みにするならば、*Navi10* の 10.3B の 2倍どころか *Vega20* の 2倍以上のトランジスタ数をつぎ込み、CU数、動作クロックも向上した GPU をフルスペック、同程度の電力枠で製品とすることに成功していることとなり、かなり高度に設計されていることが窺える  
言い換えれば、3世代同じプロセスを使って GPU を設計したことにより、今世代の **Radeon RX 6000 Series** の高性能、優れた電力比性能を実現できたのかもしれない。  


| | Vega20<br>*Radeon VII* | Navi10<br>*RX 5700 XT 50th* | Navi2<br>*RX 6900 XT* |
| :-- | :--: | :--: | :--: |
| CU | 60(64) | 40 | 80 |
| Peak Clock | 1800 MHz | 1980 MHz | 2250 MHz |
| Power | 300 W | 235 W | 300 W |
| Transistor Count | 13.2B | 10.3B | 26.8B |




