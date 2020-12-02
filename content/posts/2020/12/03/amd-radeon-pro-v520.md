---
title: "AMD、Radeon Pro V520 を正式発表 & 仕様公開"
date: 2020-12-03T00:05:33+09:00
draft: false
tags: [ "Navi12", "RDNA"]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AWS が EC2 G4adインスタンスに、第二世代 AMD EPYC (*Rome*) と組み合わせて採用することを発表した **AMD Radeon Pro V520** だが、  
{{< link >}} [AWS、RDNAアーキテクチャ GPU 「Radeon Pro V520」を採用する G4adインスタンスを発表 | Coelacanth's Dream](/posts/2020/12/02/aws-radeon-pro-v520/) {{< /link >}}
少しおくれて AMD も、EC2 G4adインスタンスに関するニュースリリース、そして **Radeon Pro V520** の仕様を公開した。  
{{< link >}} [High-Performance AMD EPYC™ CPUs and Radeon™ Pro GPUs Power New AWS Instance for Graphics Optimized Workloads :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/983/high-performance-amd-epyc-cpus-and-radeon-pro-gpus) {{< /link >}}

**Radeon Pro V520** は *RDNAアーキテクチャ* を採用し、WGP(CU)数は 18(36)基、ピークGPUクロックは 1600MHz、メモリは HBM2 2048-bit、メモリ速度は 1GHz(2Gbps)、ピークメモリ帯域は 512GB/s となる。また HBM2メモリは ECC機能に対応。  
PCIeカードの形で提供され、2スロット厚、PCIe Gen3/Gen4 x16。TBP (Total Board Power) は 225W。  

[Amazon Virtual Workstation (G4ad) | AMD](https://www.amd.com/en/graphics/workstation-virtual-graphics-amazon-g4ad)

同じ [Navi12](/tags/navi12) ベースで、既に Apple MacBook Pro向けに提供されていた **Radeon Pro 5600M** と比較すると、WGP(CU) は 2(4)基が無効化され、ピークGPUクロックは約1.54倍となっている。  
メモリクロックもモバイル向けであるためか **Radeon Pro 5600M** では 0.77GHz(1.54Gbps) に抑えられていたが、**Radeon Pro V520** ではフルの 1GHz(2Gbps) となる。  
消費電力については、**Radeon Pro 5600M** が TGP(Total Graphic Power)、**Radeon Pro V520** が TBP(Total Board Power) であるため、単純な比較はできないが、クロックを上げたため、相応に高いものとなっている。  

| Navi12 | Radeon Pro 5600M | Radeon Pro V520 |
| :--- | :---: | :---: |
| WGP(CU) | 20(40) | 18(36) |
| &emsp;Stream Processors | 2560 | 2304 |
| Peak Engine Clock | 1035 MHz | 1600 MHz |
| Memory Type | HBM2 | HBM2 |
| Memory Interface | 2048-bit | 2048-bit |
| Memory Speed | 1.54 Gbps | 2 Gbps
| Memory Bandwidth | 394 GB/s | 512 GB/s |
| FP16 (TFLOPS) | 10.6 | 14.75 |
| FP32 (TFLOPS) | 5.3 | 7.4 |
| Power | 50 W<br>(TGP) | 225 W<br>(TBP) |

Amazon EC2 G4adインスタンスは、ユースケースにゲーム開発やレンダリング等、リモートワークステーション用途以外にゲームストリーミングを想定している。[^g4ad]  
AMD は 2020/03 の Financial Analyst Day 2020 にて、*RDNAアーキテクチャ* はモバイル向けからクラウドゲーミング向けまでスケールするとしていたが[^rdna-scaling]、今回の **Radeon Pro V520** の発表によりそれが果たされたこととなる。  

[^g4ad]: [Amazon EC2 G4 Instances — Amazon Web Services (AWS)](https://aws.amazon.com/jp/ec2/instance-types/g4/)
[^rdna-scaling]: [Navi12情報近況 (2020-03-09) & 推測 | Coelacanth's Dream](/posts/2020/03/09/navi12-recent-info/)
