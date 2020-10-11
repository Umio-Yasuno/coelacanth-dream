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
