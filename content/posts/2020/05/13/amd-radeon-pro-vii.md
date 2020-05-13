---
title: "AMD、Infinity Fabric 接続のマルチGPUに対応した Radeon Pro VII を発表"
date: 2020-05-13T23:21:50+09:00
draft: false
tags: [ "Vega20", "gfx906" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMD は 2020/05/13 付でプロフェッショナル向けハイエンドGPUカード、**Radeon Pro VII** を発表した。  
{{< link >}}[AMD Expands Professional Offerings with AMD Radeon Pro VII Workstation Graphics Card and AMD Radeon Pro Software Updates | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-expands-professional-offerings-amd-radeon-pro-vii){{< /link >}}

ベースとなるGPUは[Vega20](/tags/vega20)、総CU数 60基、GPUメモリ HBM2 16GB (4096-bit)という点はコンシューマ向けの **Radeon VII** と同じだが、制限されていた *Vega20* の機能を **Radeon Pro VII** では解放している。  

具体的には、倍精度演算性能を **Radeon VII** では 1/4レートに制限していたのを、フル 1/2レートにすることで性能を向上。  
PCIe も Gen4 に対応した。これはデスクトップ/ワークステーションで、PCIe Gen4対応のプラットフォームを出していることが大きいのではないかと思う。  
**Radeon VII** 登場時は CPU側が第2世代Ryzen /Threadripperであり、PCIe Gen3までの対応だった。  
HBM2 が ECCに対応したが、容量、帯域は変わらない。  
動作クロックは **Radeon VII** の最大 1800MHzから 1705MHzに抑えられており、その分TBPが300Wから250Wとなっている。  

そして、2GPUまでの Infinity Fabric ブリッジを用いた接続に対応している。帯域は双方向合わせて 168GB/s。  
Infinity Fabric 接続により、2GPUのメモリは共有化され、GPU間のデータ移動を広帯域、低レイテンシで行なえる。  

個人的に気になるのは、HEVC 8Kデコードに対応しているという点で、*Vega20* は 4Kまでの対応であるはずだ。  
[RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeonuvdunifiedvideodecoderhardware)ではバージョンこそ *Vega10* から進んだ UVD 7.2 だが、最大解像度は 4Kとなっている。  
RadeonFeature は 公式に 8Kデコードに対応しているとされる VCN 2.0 も、対応最大解像度は 4Kとなっているため、根拠には微妙だが、  
オープンソースドライバーのソースコードを見ても、8Kに対応するのは Radeon Family における [Renoir](/tags/renoir)からとされている。(Renoir、Arcturus、Navi10、Navi12、Navi14が対象)  
固定ハードウェアによる対応ではなく、シェーダーを用いての対応ということなのだろうか？  

| Radeon Pro | <span style="color:skyblue">W5700</span> | <span style="color:skyblue">W5700X</span> | <span style="color:skyblue">VII</span> |
| :--- | :--: | :--: | :--: |
| CUs | 36 | 40 | 60 |
| SPs | 2304 | 2560 | 3840 |
| TMUs | 144 | 160 | 240 |
| ROPs | 64 | 64 | 64? |
||
| Boost Clock | 1929 MHz | 1855 MHz | 1705 MHz |
||
| Max Memory Size | 8 GB | 16 GB | 16 GB |
| Memory Bandwidth | 448 GB/s | 448 GB/s | 1024 GB/s |
||
| FP16 (TFLOPS) | 17.78 | 19.0 | 26.2 |
| FP32 (TFLOPS) | 8.89 | 9.5 | 13.1 |
| FP64 (TFLOPS) | 0.56 | 0.59 | 6.55  |
| Typical Board Power | 190 W<br>205 W (+USB) | ? | 250 W |
| Price | $799[^2] | $1,000[^1] | $1,899 |

[^1]: [Radeon Pro W5700X MPX Module - Apple](https://www.apple.com/shop/product/MW662AM/A/radeon-pro-w5700x-mpx-module)
[^2]: [AMD Launches World’s First 7nm Professional PC Workstation Graphics Card for 3D Designers, Architects and Engineers | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-launches-worlds-first-7nm-professional-pc-workstation)
