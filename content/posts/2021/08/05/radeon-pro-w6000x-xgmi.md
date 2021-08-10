---
title: "XGMI/Infinity Fabric Link に対応する Radeon Pro W6000X シリーズ"
date: 2021-08-05T14:12:14+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2", "Navi21" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

現地時間 2021/08/03 付に、AMD は Mac Pro 向けに *RDNA 2 アーキテクチャ* を採用した **Radeon Pro W6000X Series** を発表した。  
今回発表された **Radeon Pro W6900X/W6800X/W6800X Duo** の 3製品は *Sienna Cichlid (Navi21)* をベースにしており、同 GPU ASIC をベースとする他製品との違いには *XGMI (Global Memory Interconnect)/Infinity Fabric Link* による高速な GPU間通信に対応している点が挙げられる。  

 * [New AMD Radeon PRO W6000X Series GPUs Bring Groundbreaking High-Performance AMD RDNA 2 Architecture to Mac Pro :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1016/new-amd-radeon-pro-w6000x-series-gpus-bring)
 * [AMD Radeon™ Pro Graphics for Apple | AMD](https://www.amd.com/en/graphics/radeon-apple)

中でも **Radeon Pro W6800X Duo** は、MPX Module 上に 2基の GPU を搭載し、*XGMI/Infinity Fabric Link* で接続する構成を採っており、ピーク FP32 演算性能は 30.2 TFLOPS、TGP (Total Graphics Power) は 400W と、性能的にも消費電力的にも強力ななものとなっている。  

現在は構成選択時に表示されなくなっているが、同様の立ち位置にある **Radeon Pro Vega II** と比較すると、ピーク FP32 演算性能では **W6000X Series** が勝るが、メモリ帯域ではやはり HBM2 を採用していることが大きく、**Radeon Pro Vega II** が 1 TB/s、**W6000X Series** は 512 GB/s とかなりの差がある。  
ただ **W6000X Series** は GPU あたり 128 MiB の *Infinity (L3) Cache* を搭載しており、メモリ帯域が影響しやすい処理については *Infinity Cache* である程度補える可能性がある。  

*XGMI/Infinity Fabric Link* のピーク帯域は 84 GB/s (21 Gbps) と、**Radeon Pro Vega II** と同じ。同様に *Vega20* をベースにするサーバ、データセンター向けの **AMD Instinct MI50/MI60** では 92 GB/s (23 Gbps) となっているため、消費電力や性能の観点からワークステーション向けとデータセンター向けで *XGMI/Infinity Fabric Link* の最適な帯域を決めているのではないかと思われる。  

そして {{< comple >}} これが今回のトピックではあるが {{< /comple >}}、*Sienna Cichlid* が *XGMI/Infinity Fabric Link* に対応していることは、約 1 年前に AMD GPU ドライバーへのパッチにて明かされており、個人的にはようやく実装、有効化された製品 {{< comple >}} Mac Pro 向けとはいえ {{< /comple >}} が登場したという気持ちがある。  
{{< link >}} [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/) {{< /link >}}
同様に明かされてはいたが、実装、有効化された製品が登場するまで時間が空いた機能には ECC 機能があり、これは PCIeカード形状を採るワークステーション向けの **Radeon Pro W6800** で有効化されている。  
{{< link >}} [New AMD Radeon PRO W6000 Series Workstation Graphics with AMD RDNA 2 Architecture and Massive 32GB of Memory to Power Demanding Architectural, Design and Media Workloads :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1008/new-amd-radeon-pro-w6000-series-workstation-graphics-with) {{< /link >}}

1年より前、*Sienna Cichlid* の設計段階からこうした製品展開が計画されていたと思われるが、自分のような外部の人間がそれを知ることができるのはパッチ等が投稿、公開されてからで、実際の製品はそれよりも後に登場することになる。  
自分には気長に、ただ待つ他ない。  
