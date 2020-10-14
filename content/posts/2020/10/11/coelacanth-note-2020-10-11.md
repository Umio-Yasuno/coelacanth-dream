---
title: "【シーラカンスノート】 【2020/10/11】"
date: 2020-10-11T04:38:04+09:00
draft: true
tags: [ "Green_Sardine", ]
# keywords: [ "", ]
categories: [ "Note", ]
noindex: false
# summary: ""
toc: false
---

## Green Sardine DCN2.1

大体が *Renoir APU* と同じとされる *Green Sardine APU* のディスプレイ部、DC(Display Core) をサポートするパッチが投稿された。  
{{< link >}} [[PATCH 1/2] drm/amd/display: Add green_sardine support to DC](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054696.html) {{< /link >}}

ディスプレイ部も *Renoir* と同じであり、バージョンは DCN2.1 となる。  

## Navy Flounder GDDR6

メモリの各種タイミングを調節するトレーニング機能において、*Navy Flounder* は GDDR6 をサポートするとされている。  
{{< link >}} [drm/amdgpu: enable GDDR6 save-restore support for navy_flounder](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=ac0b5ce6684089cacc3968ce8647a87df01e9d85) {{< /link >}}

だが、*RDNA 2 /GFX10.3* GPU のメモリに関してははっきりしない所があり[^sienna_cichlid-mem]、また現時点ではエミュレーションモードで dGPU をテストする場合、メモリタイプに GDDR6 をセットしているため、これだけで *Navy Flounder* は GDDR6 をサポートとは言い切れない。  

[^sienna_cichlid-mem]: [Navi21 /Sienna Cichlid のメモリは GDDR6 か HBM2 か、新たに出てきたパッチを読む | Coelacanth's Dream](/posts/2020/07/27/what-sienna_cichlid-hbm2/)

## Radeon Pro 5600M

LLVM のドキュメント内にて gfx1011 を採用している製品に **Radeon Pro 5600M** が記載されていたのだが、それを gfx1010 に移すパッチが投稿され、組み込まれている。  

[[AMDGPU] Correct processor names for gfx1010 and gfx1011 · llvm/llvm-project@fe145b6](https://github.com/llvm/llvm-project/commit/fe145b66ecfd98769feef496d47e49781efd6a2e)

まず Mesa3D から、*Navi12* は *gfx1011* とされている。  
<https://gitlab.freedesktop.org/mesa/mesa/-/blob/283686ad6762182037b708f1b5187129aff0a5dd/src/amd/llvm/ac_llvm_util.c#L173>  

そして、Geekbench の結果から **Radeon Pro 5600M** は *gfx1011* と認識されている。  
[Apple Inc. MacBookPro16,4 - Geekbench Browser](https://browser.geekbench.com/v5/compute/1599856)  

パッチの意味する所は不明。  

