---
title: "ROCmのコードにMI100、Arielが追加される"
date: 2019-12-20T07:19:56+09:00
draft: false
tags: [ "Radeon", "GCN", "RDNA", "Arcturus", "Navi10", "GFX9", "GFX10", "gfx908", "gfx1010", ]
keywords: [ "Radeon", "Navi10", "Arcturus" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

ROCm-OpenCL-Runtimeのコミットにおいて、MI100、Arielの名が確認された。  
2つのGFX IDは、MI100が**gfx908**、Arielが**gfx1000**となっている。  
[rocdefs.hpp#L75 - RadeonOpenCompute/ROCm-OpenCL-Runtime](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/blob/c0567561d1c2a07c9aa63edf940f8589d2ee1dc6/runtime/device/rocm/rocdefs.hpp#L75)  
[rocdevice.cpp#L98 - RadeonOpenCompute/ROCm-OpenCL-Runtime](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/blob/c0567561d1c2a07c9aa63edf940f8589d2ee1dc6/runtime/device/rocm/rocdevice.cpp#L98)  

#### MI100
gfx908というのはチップコードネーム Arcturus に割り振られたGFX IDだが、これまでのROCm関連のコードではArcturusという名が使われることはなく、GFX908そのままが使われていた。  
今回のコミットでちゃんとした名に変わったのだけれど、何故かArcturusではなくMI100となっている。  
MI100自体は過去にもgfx908に対して使われており、関連がない訳ではない。  
[MI100 merge (#1951) - ROCmSoftwarePlatform/MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen/commit/bc7d47bd0d65b331667da74ba3cdb04893a77998)  
ただ、いささか製品的な気が過ぎるように思う。  
Arcturusには現在4つのDeviceIDが確認されており、（内1つはVirtual Function[MxGPU]用）  
[drm/amdgpu: add pci DID for Arcturus GL-XL.  - linux/amd-staging-drm-next](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=48c69cda452f4817d040908d743a818187767f17)  
[amd/amdgpu: add Arcturus vf DID support - linux/amd-staging-drm-next](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=ea207b29ae7718a8717de6ea784e99d41826e642)  
それらすべての製品名がMI100となるということなのだろうか？  

Arcturusは最大8 ShaderEngine、128CUという構成になると見られ、そうなるとダイサイズも巨大化するため、歩留まり向上としてMI100という枠内で複数のグレードに分けているのかもしれない。  
GPGPU全振りなアーキテクチャであり、Server Acceleratorsジャンルでしか出せないというものあるだろう。  

#### Ariel
あんまりよくわかってない存在。  
gfx1000というGFX IDからNavi10（gfx1010）より早くに開発されたRDNAアーキテクチャと読める。  
聞いた話では、Ariel（gfx1000）こそが次世代PSに使われるGPUらしい。  

SonyはPS4のOSにてFreeBSDをベースに採用しており、そういった理由もあってコンパイラにLLVM Clangを開発に採用していた。  
[PlayStation 4、開発にはLLVM Clang - マイナビニュース](https://news.mynavi.jp/article/20131225-a269/)  
[LLVM Clang、PlayStation 4用コードを統合開始 - マイナビニュース](https://news.mynavi.jp/article/20150129-a018/)  
今回確認されたコードはHSA関連と見られ、  
そしてそのHSAのバックエンドにはLLVMが使われているため、その関係でArielも追加されたのだろう。  
同時に、Sonyは次世代PSでも引き続きFreeBSDベースのOS、コンパイラにLLVM Clangを採用すると見られる。PS4との互換性を予定しているらしいため、当然と言えば当然かもしれないが。  

#### おまけ
Navi1xも追加されている。  
ROCmにてNavi1xのOpenCLがサポートされる日は近いのかもしれない。  
現在進行系で一部騒がれている、Windows版ドライバのOpenCLランタイムのバグも修正されることを願う。  
<span style="color:darkslategray">相変わらずAMD GPUの新製品はソフトウェア面が不遇……</span>
