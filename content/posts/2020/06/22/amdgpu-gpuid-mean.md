---
title: "AMD GPU の GPU ID は何を意味するか"
date: 2020-06-22T23:12:53+09:00
draft: false
tags: [ "Radeon", "LLVM" ]
keywords: [ "", ]
categories: [ "Hardware", "GPU", "AMD", "Database" ]
noindex: false
---

AMD GPU を話題にすると時々出てくる *gfx908* だとか *gfx1012* といった文字列。  
これが何を意味するかと言うと、AMD GPUのマシンタイプ、アーキテクチャであり、そのアーキテクチャが対応する命令や機能、つまりは命令セット、ISA であり、  
また存在するバグを示すために使われる。  

ちなみに *gfx908* は [Arcturus](/tags/arcturus)、*gfx1012* は [Navi14](/tags/navi14) に採用されているアーキテクチャを示す。  

{{< pindex >}}

 * [Machine Type / GPU ID / GFX ID](#gpuid)
 * [GPU ID から読み取れること](#read-from-gpuid)
   * [規模は読み取れない](#cant-read-scale)
   * [HBCC / XNACK について](#hbcc-xnack)

{{< /pindex >}}

## Machine Type / GPU ID / GFX ID {#gpuid}
そして gfx から始まる文字列は **Machine Type / GPU ID / GFX ID** といった名で使われる。  
どれも同じ意味であるため、名前で特に使い分けはされておらず、コードを書く人次第なのではないかと思う。  
この記事では **GPU ID** とする。  

そして gfx より後ろの数字にはそれぞれ意味があり、  
最初がその GPU の主要なバージョン、世代を、(`pGfxIpMajorVer`)  
その次が補助的なバージョンを、(`pGfxIpMinorVer`)  
最後がステッピングとなる。(`pGfxIpStepping`)[^1]  

[^1]: [pal/palPipelineAbiProcessorImpl.h at e1b2dde021a2efd34da6593994f87317a803b065 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/e1b2dde021a2efd34da6593994f87317a803b065/inc/core/palPipelineAbiProcessorImpl.h#L640)

Navi からは世代が *GFX10* と二桁になったが、LLVM 等では `_` で区切りを付けて記述されているため、問題はない。[^2]  
GFX1世代とされる AMD GPU はないため、間違うこともそう無いと思うが。  

[^2]: [llvm-project/AMDGPU.td at master · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/master/llvm/lib/Target/AMDGPU/AMDGPU.td)

また、GFX10 までは補助的なバージョンを示す数字が使われることはあまり無く、*gfx810* である *Stoney* APU くらいで、ほとんど `0` となっていた。  
GFX10 からは *RDNA /Navi1x* であることを示すためには `1` (gfx101x) 、*RDNA 2/Navi2x* には `3` (gfx103x) が充てられている。  
`2` (gfx102x) が飛ばされているが、存在はしており、*Navi21_Lite/nv21_lite* が *gfx1020* とされている。[^3]*gfx1000* の *Navi10_Lite/nv10_lite* もあり、そちらは *Ariel* というコードネームが付けられている。[^4]  
自分が知っているのはそれだけで、詳しくは知らない。<span class="hide">知りたいなら探偵様にでも聞いてくれ。</span>  

[^3]: [P4 to Git Change 1917655 by vsytchen@vsytchen-remote-ocl-win10 on 201… · ROCm-Developer-Tools/ROCclr@3d26650](https://github.com/ROCm-Developer-Tools/ROCclr/commit/3d2665034250cf93bc88b409e67c86453f568bd4)
[^4]: <https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/blob/c0567561d1c2a07c9aa63edf940f8589d2ee1dc6/runtime/device/rocm/rocdefs.hpp#L75>

## GPU ID から読み取れること {#read-from-gpuid}
その **GPU ID** を [llvm-project/AMDGPU.td](https://github.com/llvm/llvm-project/blob/master/llvm/lib/Target/AMDGPU/AMDGPU.td) と照らし合わせることで、その AMD GPUアーキテクチャが対応している命令、機能、存在するバグについて知ることができる。  

これまでに登場してきた AMD GPU の **GPU ID** は[記事下部の表](#amdgpu-codename-gpuid)にて。  

読み方としては例えば、*gfx1011 (Navi12)* 、*gfx1012 (Navi14)* には `FeatureDot1Insts` 、`FeatureDot2Insts`、`FeatureDot5Insts`、`FeatureDot6Insts` があるけど[^5]、*gfx1010 (Navi10)* には無い[^6]、  
*gfx1012 (Navi14)* にはバグに `FeatureLdsMisalignedBug` があるけど[^7]、*gfx1011 (Navi12)* には無い (修正されている?)[^7]、という風に比較することが基本となる。  

[^5]: <https://github.com/llvm/llvm-project/blob/9ee272f13d88f090817235ef4f91e56bb2a153d6/llvm/lib/Target/AMDGPU/AMDGPU.td#L900>
[^6]: <https://github.com/llvm/llvm-project/blob/9ee272f13d88f090817235ef4f91e56bb2a153d6/llvm/lib/Target/AMDGPU/AMDGPU.td#L881>
[^7]: <https://github.com/llvm/llvm-project/blob/9ee272f13d88f090817235ef4f91e56bb2a153d6/llvm/lib/Target/AMDGPU/AMDGPU.td#L934>


### 規模は読み取れない {#cant-read-scale}
下の表を見るとわかるが、GPU ID は普通に被る。  
規模の異なる *Fiji /Polaris10 /Polaris11 /Polaris12* が一緒の *gfx803* になってたり、  
最近の AMD GPU でも、*Raven2 /Renoir* が一緒の *gfx909* だった。  
だから、GPU ID と [llvm-project/AMDGPU.td](https://github.com/llvm/llvm-project/blob/master/llvm/lib/Target/AMDGPU/AMDGPU.td) と組み合わせで、ある AMD GPU の規模を知ることは不可能と言っていい。  
GPU ID だけとなるともっと無理。  

倍精度(FP64)演算を、単精度(FP32)演算の半分のレートで実行できることを示す `HalfRate64Ops` があれば、その AMD GPUアーキテクチャはサーバやデータセンター向けで、それを採用する GPUもそういった製品に使われる、といったことは推測できるが、あくまで規模が読み取れる訳ではない。  

対応している命令、機能、存在するバグといった情報は、GPUの中で低い階層に位置するため、それだけで全体を見渡すことはできない。  

また GPU ID がアーキテクチャを示すと言っても、[llvm-project/AMDGPU.td](https://github.com/llvm/llvm-project/blob/master/llvm/lib/Target/AMDGPU/AMDGPU.td) を見るに、  
GFX10世代が対応する命令、機能をまとめた `FeatureGFX10` に、Wave内のスレッド数が 32スレッドである `FeatureWavefrontSize32` が無く、*gfx1010 /gfx1011 /gfx1012 /gfx1030* それぞれに記述されていることから、  
gfx10xx としても、厳密に言えば RDNA系列でない可能性がある、というのが自分の考えだ。  
gfx10xx すべてが RDNA系列となるなら `FeatureGFX10` に記述すれば楽に済む。  

メジャーバージョン、世代で Wavefront のサイズを判定するコードもどっかにあるだろうから、わざわざややこしく、わかりにくいことをするとも考えにくいが、可能性としてはわずかに存在する。  

### HBCC / XNACK について {#hbcc-xnack}
HBCC は CPU側の一部 DRAM も追加で VRAM として扱う *Vega* の目玉機能だが、これを有効にすると GPU ID が変更されていた。[^8]  
*Vega10 (HBCC無効)* が *gfx900* 、*Vega10 (HBCC有効)* が *gfx901* となる。  

[^8]: [ROCm-OpenCL-Runtime/rocdefs.hpp at c0567561d1c2a07c9aa63edf940f8589d2ee1dc6 · RadeonOpenCompute/ROCm-OpenCL-Runtime](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/blob/c0567561d1c2a07c9aa63edf940f8589d2ee1dc6/runtime/device/rocm/rocdefs.hpp#L71)

何で変更する必要があるのかは、  
まず HBCC機能の中身としては、メモリのデマンドページングやページ移行に使用される `XNACK` という機能を有効にしている。[^9][^10] LLVM から *gfx901* 等は消されているが、消された時の内容から *gfx901* に `FeatureXNACK` がある。[^11]

`XNACK` を有効にした場合に生成したコードを、無効にされている GPU で実行すると、正しく実行されない、またはパフォーマンスが低下するといったことが発生するため、GPU ID で判別する必要があるのだと思われる。  

[^9]: [User Guide for AMDGPU Backend — LLVM 8 documentation](https://prereleases.llvm.org/8.0.0/rc5/docs/AMDGPUUsage.html#target-features)
[^10]: [pal/palDevice.h at 82390674c40402c81dbf311a5ee488fb972894f5 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/82390674c40402c81dbf311a5ee488fb972894f5/inc/core/palDevice.h#L1020)
[^11]: [AMDGPU/GCN: Bring processors in sync with AMDGPUUsage · llvm/llvm-project@c40d9f2](https://github.com/llvm/llvm-project/commit/c40d9f2e5df4482eb036a6130e8f3ae30294f3b4#diff-9b0b58a9b5e6244681ae14d7079f3704)

しかし HBCC/XNACK はもう dGPU で有効にすることは想定されておらず、dGPU には `FeatureDoesNotSupportXNACK` が記述され、また *Vega20 (gfx906)* 以降は判別するための GPU ID も確保されていない。  
元々 `XNACK` はメモリを CPU と GPU で共有する APU 向けの機能だったらしく、元の位置に収まったとも言える。  

## AMD GPU Code Name / GPU ID {#amdgpu-codename-gpuid}

| Code Name | Tahiti | Pitcairn / Capeverde / Oland / Hainan |
| :-- | :--: | :--: |
| GPU ID | gfx600 | gfx601 |

| Code Name | Spectre / Spooky | HawaiiPro | Hawaii | Kalindi / Godavari | Banaire |
| :-- | :--: | :--: | :--: | :--: | :--: |
| GPU ID | gfx700 | gfx701 | gfx702 | gfx703 | gfx704 |

| Code Name | Carrizo / Bristol | Iceland / Tonga | Fiji / Polaris10 / Polaris11 / Polaris12 | Stoney |
| :-- | :--: | :--: | :--: | :--: |
| GPU ID | gfx801 | gfx802 | gfx803 | gfx810 |

| Code Name | Vega10 | Vega10 (HBCC) | Raven / Picasso | Vega12 | Vega20 | Arcturus | Raven2 / Renoir |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| GPU ID | gfx900 | gfx901 | gfx902 | gfx904 | gfx906 | gfx908 | gfx909 |

| Code Name | Navi10 | Navi12 | Navi14 | Navi21 |
| :-- | :--: | :--: | :--: | :--: |
| GPU ID | gfx1010 | gfx1011 | gfx1012 | gfx1030 |

(参考: <cite><https://github.com/GPUOpen-Drivers/pal/blob/e1b2dde021a2efd34da6593994f87317a803b065/src/core/os/nullDevice/ndDevice.cpp#L106></cite>)

