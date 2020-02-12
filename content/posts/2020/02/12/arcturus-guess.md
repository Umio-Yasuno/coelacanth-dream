---
title: "AMDのGPGPU向け新GPUアーキテクチャ、Arcturus推測"
date: 2020-02-12T04:57:09+09:00
draft: true
tags: [ "Radeon", "GCN", "gfx908", "GFX9", "Arcturus" ]
keywords: [ "Arcturus", "gfx908" ]
categories: [ "Hardware", "GPU", "AMD" ]
noindex: false
---

ある程度自分の中で情報が溜まってきたため、一度記事にして整理することとした。  

### 歩留まりのため8CUを無効、120CU構成か
[前回のArcturusのまとめ](/posts/2019/11/05/arcturus-matome/)の内容と被るが、Arcturusでは最大128CU、8-ShaderEngine(SE)の構成に対応している。  
[[PATCH 099/102] drm/amdkfd: Increase vcrat size for GPU](https://lists.freedesktop.org/archives/amd-gfx/2019-July/036848.html)  
[drm/amdkfd: Extend CU mask to 8 SEs (v3)](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/include/v9_structs.h?h=amd-staging-drm-next&id=5145d57ec5f5cf7dadaa6ccd9c9f1e4dae82570b)  

しかしArcturusベースの製品で128CUすべてを有効にするとは考えづらい。  
Arcturusはその構成から巨大なダイになるとされ、自然と歩留まりは厳しくなる。  
そしてGPGPUとしての色が強い製品は、需要の多くがスパコンやサーバーであり、そこではまとまった数を導入する。  
要は数が必要で、歩留まりを少しでも向上させるため、複数あるユニットの一部を無効化や動作クロックを抑えるのが常だ。  

過去の例で言えば、かなりのビックダイである (815mm<sup>2</sup>) NVIDIA GV100ベースのTesla V100は4SM(256CUDA Core)を無効化していた。  

 > 参考: [［GTC 2017］西川善司の3DGE：Volta世代のGPU「GV100」は，これまでと大きく異なるプロセッサだ――いったい何が？ - 4Gamer.net](https://www.4gamer.net/games/208/G020859/20170512111/)  

そして過去のArcturus(gfx908)に関するソフトウェアへの更新には以下の文面があった。  

 > \* add 120CU support of perfdb and find-db

 > 引用元: [MI100 merge (#1951) · ROCmSoftwarePlatform/MIOpen@bc7d47b](https://github.com/ROCmSoftwarePlatform/MIOpen/commit/bc7d47bd0d65b331667da74ba3cdb04893a77998)  

AMDGPUはSEが対称となるように構成され、8SEのArcturusがCUを一部無効するならば8刻みとなり、120CUはそれと一致する。  

### 動作クロックは 1600MHz前後？
この前初めて知ったのだけれど、AMDはServer Accelerator向けであるInstinctシリーズの製品名にて、MIの後に数字を置いているが、  
これは複数のデータ精度の中で最もピーク性能が高い数字を採用している。（見栄えが良い数字とも言える。）  
ただ *大体* の数字であり、上から二桁目は四捨五入で丸み込みがなされている。  
（MI60はINT8で59 TOPs、MI50はINT8で53 TOPs）  
[Professional Graphics Specifications | AMD](https://www.amd.com/en/products/specifications/professional-graphics/4476)  

このことと、120CUという推測を合わせると、逆算して動作クロックは1628MHzとなる。  

 > ( 100 [TOPs] / ( 120 [CU] * 64 [SP] * 2 [FLOPS] * 4 [INT8のPacked実行] )  
 > = 0.0016276 [THz] = 1.628 [GHz] = 1628 [MHz] )

大体で決まるため、実際は1600MHz前後あたりだろうか。ギリギリまで下げると1550MHz程。  

### Arcturusでは行列演算に対応する？

