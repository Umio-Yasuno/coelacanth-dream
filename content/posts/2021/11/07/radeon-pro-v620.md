---
title: "AMD、Radeon Pro V620 を発表"
date: 2021-11-07T17:33:51+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD は現地時間 2021/11/04 付で、*Sienna Cichlid/Navi21* ベースのサーバ向け GPU、**Radeon Pro V620** が発表された。  


 * [AMD Radeon™ PRO V620 GPU Delivers Powerful, Multi-Purpose Data Center Visual Performance for Today’s Demanding Cloud Workloads :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1030/amd-radeon-pro-v620-gpu-delivers-powerful-multi-purpose)
 * [AMD Radeon™ PRO V620 Graphics | AMD](https://www.amd.com/en/products/server-accelerators/amd-radeon-pro-v620)

前モデルにあたる *Navi12* ベースの **Radeon Pro V520** は [AWS re:Invent](https://reinvent.awsevents.com/) に合わせ、AWS EC2 G4adインスタンスへの採用と同時に発表されていた。  
{{< link >}} [AMD、Radeon Pro V520 を正式発表 & 仕様公開 | Coelacanth's Dream](/posts/2020/12/03/amd-radeon-pro-v520/) {{< /link >}}
今年の AWS re:Invent が 2021/11/29 から開催されるため、もしかしたらそこで **Radeon Pro V620** 採用 EC2 インスタンスが発表されるかもしれない。  

**Radeon Pro V520** は HBM2 8GB (2048-bit, 2スタック) という構成でピークメモリ帯域は 512GB/s となっていたが、**Pro V620** は GDDR6 32GB (256-bit) で同じピークメモリ帯域 512 GB/s を達成している。  
**Pro V620** は WGP (CU) を **Pro V520** の倍となる 36 (72) 基持ち、加えてピーククロックも 600 MHz 向上しているため、ピーク FP32演算性能は **Pro V520** 比で 2.74倍となっている。  
ただ TBP (Total Board Power) も増えており、**Pro V520** の 225 W に対し、**Pro V620** は 300W となる。  

| | Pro V520 | Pro V620 |
| :-- | :--: | :--: |
| WGP (CU) | 18 (36) | 36 (72) |
| Peak Clock | 1600 MHz | 2200 MHz |
| Memory Type | HBM2 | GDDR6 |
| Peak Memory BW | 512 GB/s | 512 GB/s |
| Infinity Cache | N | 128 MB |
| Peak FP32 | 7.4 TF | 20.28 TF |
| Total Board Power | 225 W | 300 W |

**Pro V620** では *XGMI/Infinity Fabric* はサポートされていないが、サーバ・クラウドゲーミング向けであり、コンピューティングに特化した性能ではないからだろうか。  
*Sienna Cichlid/Navi21* の *XGMI/Infinity Fabric* が有効化、実装されている製品は現時点では Apple Mac Pro向けの **Radeon Pro W6000X シリーズ** のみとなっている。  

*RDNA 2 アーキテクチャ* としては初めて ROCmソフトウェアに対応することがアピールされている。  
先日リリースされた ROCm v4.5 では対応 GPU に同じく *Navi21/Sienna Cichlid* ベースの **Radeon Pro W6800** が追加されていたが、本来サポート対象に想定していたのは **Pro V620** だと思われる。一部 ROCmソフトウェアでは **Pro W6800** とは異なる `DeviceID: 0x73A2` を対象としており、これが **Pro V620** に割り当てられているのかもしれない。  
{{< link >}} [ROCm v4.5 がリリース ―― CPU+GPU の統合メモリをサポート、RDNA系のサポートは v5.0 に？ | Coelacanth's Dream](/posts/2021/10/30/rocm-4_5-release/) {{< /link >}}

**Radeon Pro V620** では *Sienna Cichlid/Navi21* がマルチメディアエンジン VCN (Video Core Next) を 2基持つ点に触れられている。  
このことは 2020/06 に投稿された Linux Kernel の AMD GPUドライバーへのパッチでも明かされていたが、2基の VCN は非対称であり、片方がデコードのみを、もう片方がエンコードのみを担当する。  
{{< link >}} [AMD の次世代GPU、Sienna Cichlid の Linux Kernelパッチが投稿される | Coelacanth's Dream](/posts/2020/06/02/amd-sienna_cichlid/#vcn3) {{< /link >}}
2基をまとめて、デコード/エンコードを行う 1基の VCN としては構成されはしないが、役割を明確に分けることでパワーゲーティングを設定しやすくなり、より優れた電力効率を得ることができるとされている。[^vcn3-pg]  
デコード/エンコード性能の詳細については不明だが、*Navy Flounder* などの統合された VCN 1基を搭載する場合よりも引き上げられている可能性はある。  
また、*Beige Goby/Navi24* では対応コーディックに違いこそあるが、デコードのみに対応する VCN を 1基搭載しており、元々 VCN 3 ではデコードとエンコードにそれぞれ分割が可能なように設計されていると考えられる。  
{{< link >}} [他 RDNA 2 GPU とは対応コーディックが異なる Yellow Carp APU と Beige Goby GPU | Coelacanth's Dream](/posts/2021/07/14/yc-bg-vcn/) {{< /link >}}

[^vcn3-pg]: [[PATCH 38/42] drm/amd/powerplay: set VCN1 pg only for sienna_cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051564.html)
