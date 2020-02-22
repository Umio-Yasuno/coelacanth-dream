---
title: "Arcturus情報近況 (2020/02/22)"
date: 2020-02-22T08:54:20+09:00
draft: false
tags: [ "Radeon", "GCN", "gfx908", "GFX9", "Arcturus" ]
keywords: [ "Arcturus", "gfx908" ]
categories: [ "Hardware", "GPU", "AMD" ]
noindex: false
---

新情報は特に無く、推測が9割。  

## Arcturusでは行列演算に対応
結構今更な情報。  
Arcturusでは AGPR (Accumulate? General-Purpose Register) が増設され、INT8 /FP16 /FP32 /BF16 32x32までの行列演算に対応すると思われる。  
NVIDIA Tensor CoreはVolta、TuringではFP16までの対応であり、BF16には対応してないため、学習精度という点では有利だが、NVIDIAもAmpereで対応しそうである。  
また、専用ハードウェア追加ではなく、CU側での対応という汎用的なものであるから、性能がどこまで出るか。その分割かなければならないダイエリアも少なく済むが。  

 > 参考:  
 > * [llvm-project/AMDGPUAsmGFX908.rst at master · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/master/llvm/docs/AMDGPU/AMDGPUAsmGFX908.rst)  
 > * [MI 32x32x2x1 F32 (#800) · ROCmSoftwarePlatform/Tensile@9dfd621](https://github.com/ROCmSoftwarePlatform/Tensile/commit/9dfd6212f888cf83ff334e675c823239831c4aa8)  
 > * [Initial version of Store code clean-up for mi100 · ROCmSoftwarePlatform/Tensile@1ae540b](https://github.com/ROCmSoftwarePlatform/Tensile/commit/1ae540b7170ef971abdecd1ab0b4fc3e5f927dc1)  
 > * [Fix total number of Agprs to account for multiple instructions, add d… · ROCmSoftwarePlatform/Tensile@756d5b3](https://github.com/ROCmSoftwarePlatform/Tensile/commit/756d5b3e435fea4aeeea74bd17d206f6fc01c305)  
 > * [GTC Japan 2017 - NVIDIAのVoltaを読み解く(2) V100で新設されたTensorコア | マイナビニュース](https://news.mynavi.jp/article/gtcjapan2017_v100-2/)

## 歩留まりのため8CUを無効、120CU構成か
[前回のArcturusのまとめ](/posts/2019/11/05/arcturus-matome/)の内容と被るが、Arcturusでは最大128CU、8-ShaderEngine(SE)の構成に対応している。  
[[PATCH 099/102] drm/amdkfd: Increase vcrat size for GPU](https://lists.freedesktop.org/archives/amd-gfx/2019-July/036848.html)  
[drm/amdkfd: Extend CU mask to 8 SEs (v3)](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/include/v9_structs.h?h=amd-staging-drm-next&id=5145d57ec5f5cf7dadaa6ccd9c9f1e4dae82570b)  

しかしArcturusベースの製品で128CUすべてを有効にするとは考えづらい。  
Arcturusはその構成から巨大なダイになるとされ、自然と歩留まりは厳しくなる。  
そしてGPGPUとしての色が強い製品は、需要の多くがスパコンやサーバーであり、そこではまとまった数を導入する。  
要は数が必要で、歩留まりを少しでも向上させるため、複数あるユニットの一部を無効化や動作クロックを抑えるのが常だ。  

過去の例で言えば、かなりのビックダイである (815mm<sup>2</sup>) NVIDIA GV100ベースのTesla V100は4SM(256CUDA Core)を無効化していた。  

 > 参考: [［GTC 2017］西川善司の3DGE：Volta世代のGPU「GV100」は，これまでと大きく異なるプロセッサだ――いったい何が？ - 4Gamer.net](https://www.4gamer.net/games/208/G020859/20170512111/)  

そして過去のArcturusに関するソフトウェアへの更新には以下の文面があった。  

 > \* add 120CU support of perfdb and find-db

 > 引用元: [MI100 merge (#1951) · ROCmSoftwarePlatform/MIOpen@bc7d47b](https://github.com/ROCmSoftwarePlatform/MIOpen/commit/bc7d47bd0d65b331667da74ba3cdb04893a77998)  

AMDGPUはSEが対称となるように構成され、8-SEのArcturusがCUを一部無効するならば8刻みとなり、120CUはそれと一致する。  

## 動作クロックは 1600MHz前後？
この前初めて知ったのだけれど、AMDはServer Accelerator向けであるInstinctシリーズの製品名にて、MIの後に数字を置いているが、  
これは複数のデータ精度の中で最もピーク性能が高い数字を採用している。（見栄えが良い数字とも言える。）  
ただ *大体* の数字であり、上から二桁目は四捨五入で丸み込みがなされている。  
（MI60はINT8で59 TOPs、MI50はINT8で53 TOPs）  
[Professional Graphics Specifications | AMD](https://www.amd.com/en/products/specifications/professional-graphics/4476)  

このことと、120CUという推測を合わせると、逆算して動作クロックは1628MHzとなる。  

 > ( 100 [TOPs] / ( 120 [CU] * 64 [SP] * 2 [FLOPS] * 4 [INT8のPacked実行] )  
 > = 0.0016276 [THz] = 1.628 [GHz] = 1628 [MHz] )

大体で決まるため、実際は1600MHz前後あたりだろうか。ギリギリまで下げると1550MHz程。  

## MI100 / Arcturus 登場時期予想

まず、ROCmのArcturusへの対応はVer 3.2で行われることが予想される。  
下記URL先を見ると、Arcturusでの新機能、行列演算やBF16対応が含まれているからだ。  
[merge develop into master for ROCm 3.2 Feature complete by amcamd · Pull Request #983 · ROCmSoftwarePlatform/rocBLAS](https://github.com/ROCmSoftwarePlatform/rocBLAS/pull/983)  
そしてROCmは基本、2、3ヶ月の間隔でリリースされている。  
[Releases · RadeonOpenCompute/ROCm](https://github.com/RadeonOpenCompute/ROCm/releases)  
そのことからROCm 3.2は 2020/04 〜 2020/06 の間にリリースされると予想され、近い時期のイベントには、  

 * [COMPTEX TAIPEI](https://www.computextaipei.com.tw/) (2020/06/02 〜 2020/06/06)
 * [ISC High Performance 2020](https://www.isc-hpc.com/) (2020/06/21 〜 2020/06/25)

がある。  
そういうことで、そのどっちかで発表するのではないか、と考えている。イベント関係なく発表する可能性もあるが。  
ただ、ソフトウェアが無ければせっかくハードウェアの発表をしても霞んでしまうため、ROCmのリリースには合わせてくるはずだ。  

### おまけ 8GPUシステムのトポロジー概要？

 >	const char *model_descriptions[] = {  
  "4 nodes with 8 GPUs PCIe 1 NIC",  
  "4 nodes with 8 GPUs PCIe 2 NIC",  
  "2 nodes VEGA20 4P1H",  
  "4 nodes with 8 VEGA20 GPUs XGMI 4P2H 1 NIC",  
  "single node gfx908 4P3L",  
  "single node gfx908 8P6L",  
  "single node gfx908 8P6L Alt. Connection",  
  "single node 8 GPUs PCIe on Rome",  
  "4 nodes 8 GPUs PCIe 2 NICs on Rome",  
  "3 nodes 8 GPUs PCIe + 1 Rome 8 GPUs PCIe + 2 nodes gfx908 4P3L",  

 > 引用元: [Add topology explorer by wenkaidu · Pull Request #172 · ROCmSoftwarePlatform/rccl](https://github.com/ROCmSoftwarePlatform/rccl/pull/172/files#diff-78fedcf9e11f148f1840999ae1b3438e)

他のファイルを見るに、8P6LのPはGPU数、Lはリンクを表すと思われる。  
[rccl/model.h at 55f8e2dec75077773ed414c1ef2f5a48afef4a6f · wenkaidu/rccl](https://github.com/wenkaidu/rccl/blob/55f8e2dec75077773ed414c1ef2f5a48afef4a6f/tools/topo_expl/include/model.h)
model.h下の表は、あるGPUから見た他のGPUへのリンク数を表しているのだろうか？  

| | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 |
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
| 6 |
| 7 |

といったように。  
具体的にどのようなネットワークになってるかは自信が全くないため、やめておく。  
