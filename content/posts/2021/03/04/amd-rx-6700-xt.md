---
title: "AMD、RDNA 2 GPU 「RX 6700 XT」 を正式発表"
date: 2021-03-04T01:22:18+09:00
draft: false
tags: [ "RDNA_2", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMD は現地時間 2021/03/03 付で、1440p の解像度をターゲットとした *RDNA 2* GPU、**RX 6700 XT** を正式発表した。  

 * [AMD Unveils AMD Radeon RX 6700 XT Graphics Card, Delivering Exceptional 1440p PC Gaming Experiences :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/990/amd-unveils-amd-radeon-rx-6700-xt-graphics)
 * [AMD Radeon RX 6700 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6700-xt#product-specs)

WGP(CU) 数は 20(40) 基。  
*Sienna Cichlid/Navi21* と同様に、SE (Shader Engine) あたりの SA (Shader Array) 2基、SA あたりの WGP(CU) 5(10)基、SA あたりの RB (RenderBackend) 2基の構成を取っていると考えられ、  
大雑把に言えば、*Sienna Cichlid/Navi21* の GPUコア部を半分にカットしたような規模、構成となる。  
メモリバス幅に関しては半分までは減らされておらず、3/4 の 192-bit で、ピークメモリ帯域は 384 GB/s、最大メモリサイズは 12GB。  
Infinity Cache のサイズは 96MiB。メモリチャネルあたり 8MiB (96 [MiB] / 12 [ch]) であり、*Sienna Cichlid/Navi21* と同じバランスとなっている。  

クロックは **Radeon RX 6800/6900 シリーズ** からさらに引き上げられ、Boost Clock は 2581MHz にも達す。  
TGP (Typical Graphics Power) で想定されている、ゲーム実行時のクロック (Game Clock) は 2424MHz。  

GPUチップのトランジスタ数は公表されていない。  
同じく WGP(CU) 20(40)基 である *Navi10* のトランジスタ数は 10.3B (103億個)、倍近い規模の *Sienna Cichlid/Navi21* が 26.8B であることから 14B 近くではないかと思われる。  

| AMD RDNA 2 | RX 6700 XT | RX 6800 | RX 6800 XT | RX 6900 XT 
| :-- | :--: | :--: | :--: | :--: |
| WGP(CU) | 20(40) | 30(60) | 36(72) | 40(80) |
| SP | 2560 | | 3840 | 4608 | 5120 |
| Ray Accelerator | 40 | 60 | 72 | 80 |
| TMU | 160 | 240 | 288 | 320 |
| ROP | 64 | 96 | 128 | 128 |
| Game Clock | 2424 MHz | 1815 MHz | 2015 MHz | 2015 MHz |
| Boost Clock | 2581 MHz | 2105 MHz | 2250 MHz | 2250 MHz |
| |
| Memory Bus Width | 196-bit | 256-bit | 256-bit | 256-bit |
| Max Memory Size | 12 GB | 16 GB | 16 GB | 16 GB |
| Memory Bandwidth | 384 GB/s | 512 GB/s | 512 GB/s | 512 GB/s |
| Infinity Cache | 96MB | 128 MB | 128 MB | 128 MB |
||
| Peak FP16 (TFLOPS) | 26.43 | 32.34 | 41.47 | 46.08 |
| Peak FP32 (TFLOPS) | 13.21 | 16.17 | 20.74 | 23.04 |
||
| Typical GPU Power | 230 W | 250 W | 300 W | 300 W |

