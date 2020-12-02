---
title: "AMD、ツール面を強化した ROCm v3.10 をリリース"
date: 2020-12-02T20:45:41+09:00
draft: false
tags: [ "ROCm", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

今回はツール面の強化が主であり、RDC(ROCm Data Center) Tool が Python から使用可能となった。  
ROCM-SMI(System Management Information) ではプロセスあたりの SDMA(System DMA) の使用率が確認可能となった。また、GPUトポロジ内のネットワーク構成、リンク情報の取得、表示も可能となった。SDMAはマルチGPU間の通信に用いられるため、そうした構成を取るノードに向けた改良がメインと取れる。  
それ以外には、計算ライブラリの API 追加、OpenMP の GPUオフロードを行なう AOMP の機能強化が行なわれている。  

今回もアップグレードではなく、クリーンインストールが推奨されている。  
前バージョンからアップグレードをサポートしないともあるため、ほぼクリーンインストールが必須と言える。  

その他詳細は以下から。  

 * [RadeonOpenCompute/ROCm at roc-3.10.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.10.x)


**AMD Instinct MI100** 発表時、性能テストに ROCm v3.10 が用いられていたため[^mi100-rocm]、既にサポートされているものと思われるが、正式には追加されていない。  

[^mi100-rocm]: [AMD EPYC™ Processors and New AMD Instinct™ MI100 Accelerator Redefine Performance for HPC and Scientific Research :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/980/amd-epyc-processors-and-new-amd-instinct-mi100)


