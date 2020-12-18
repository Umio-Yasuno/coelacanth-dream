---
title: "AMD、GPGPU開発者向けの資料を公開するサイト 「ROCm Learning Center」を開設"
date: 2020-12-18T12:26:38+09:00
draft: false
tags: [ "ROCm", ]
# keywords: [ "", ]
categories: [ "AMD", "Software", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMD は現地時間 2020/12/16 に、GPU を活用するコンピューティング、HPC、機械学習の開発者に向けた各種資料を公開するサイト [ROCm™ Learning Center - AMD](https://developer.amd.com/resources/rocm-resources/rocm-learning-center/) を開設した。  
資料は動画と PDFファイル両方の形式で提供されている。  

[CDNA1 アーキテクチャ](/tags/cdna) 採用 GPU **AMD Instinct™ MI100** のリリース、それを正式にサポートすると同時に、**ROCm** ソフトウェアスタックとして HPC、機械学習を満たすソリューションとなる **ROCm 4.0** のリリースが間近に迫っており、それに合わせてのものと見られる。  

現在公開されている資料には、**ROCm** のインストール方法、HIP (Heterogeneous-Compute Interface for Portability ) の基本的なプログラミング、CUDAソースコードを HIP に変換する [HIPIFY](https://github.com/ROCm-Developer-Tools/HIPIFY)、**ROCm** を活用した機械学習、マルチGPUプログラミングとそのコミュニケーションライブラリの解説等がある。  

**ROCm** はマルチGPUのコミュニケーションライブラリに、MPI (Maeesage Passing Interface) と RCCL (ROCm Communication Collectives Library) をサポートしている。  
RCCL は NCCL (NVIDIA Collective Communications Library) API の機能が実装されており、NCCL を用いたコードは書き換えずにそのまま RCCL を使うことができる。  
2つのライブラリの使い分けとして、RCCL は深層学習で多く使われる `AllReduce` 操作に対応しているため、学習では RCCL が向いているとし、  
MPI は HPC は長年 HPCアプリケーションで用いられており、それらアプリケーションの保守性では優れていると説明されている。  

マルチGPUシステムの例として、機械学習、物理シミュレーション、そして Bitcoinマイニングが挙げられていた。Bitcoinマイニングに高価な **Instinct** ブランドの GPU は合わないように思えるが、映像出力の無いマイニング向け Radeon カードの売りの 1つとして **ROCm** がサポートされていることがあったりするため、一応 AMD もターゲットとして見ているのだろう。[^gpu-mining]  
2020/10 には Linux Kernel にブロックチェーン/マイニング向けの [Navi10](/tags/navi10)、[Navi14](/tags/navi14) SKU のサポートが追加されている。  
{{< link >}} [ブロックチェーン向けの Navi10 SKU とそこから読める文脈 | Coelacanth's Dream](/posts/2020/10/21/navi10-sku-for-blockchain/) {{< /link >}}
{{< link >}} [Navi14 にもブロックチェーン/マイニング向け SKU が追加される | Coelacanth's Dream](/posts/2020/10/28/navi14-sku-for-blockchain/) {{< /link >}}

[^gpu-mining]: [SAPPHIRE RADEON RX 570 16GB HDMI Blockchain Graphics Card - SAPPHIRE GPU Blockchain Systems](https://gpuminer.sapphiretech.com/Radeon-RX-570-16GB-Blockchain/)
