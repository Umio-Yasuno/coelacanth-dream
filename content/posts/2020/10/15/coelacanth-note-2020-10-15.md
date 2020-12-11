---
title: "【シーラカンスノート】 Green Sardine DCN2.1、gfx1011、RADV/ACO NGG は一旦無効化 【2020/10/15】"
date: 2020-10-15T10:17:44+09:00
draft: false
tags: [ "Green_Sardine", "ACO", "NGG" ]
# keywords: [ "", ]
categories: [ "Note", ]
noindex: false
# summary: ""
toc: false
---

## Green Sardine DCN2.1

大体が *Renoir APU* と同じとされる *Green Sardine APU* のディスプレイ部、DC(Display Core) をサポートするパッチが投稿された。  
{{< link >}} [[PATCH 1/2] drm/amd/display: Add green_sardine support to DC](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054696.html) {{< /link >}}
*Green Sardine APU* のディスプレイ部は DCN2.1 となり、*Renoir APU* と同じとなる。またそれ以外の GFXコア、マルチメディアエンジン等も *Renoir APU* と同じではないかと思われる。[^green_sardine]  

[^green_sardine]: [AMD Renoir の兄弟？ Linux Kernel に "Green Sardine" APU をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2020/10/03/amd-apu-green_sardine/)

## Navy Flounder と Dimgrey Cavefish も GDDR6 のメモリトレーニング機能に対応

メモリの各種タイミングを調節するトレーニング機能において、*Navy Flounder* と *Dimgrey Cavefish* は GDDR6 をサポートするとされている。  
{{< link >}} [drm/amdgpu: enable GDDR6 save-restore support for navy_flounder](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=ac0b5ce6684089cacc3968ce8647a87df01e9d85) {{< /link >}}
{{< link >}} [drm/amdgpu: enable GDDR6 save-restore support for dimgrey_cavefish](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=5d08b18420755f501781fd625b2f50e0322af1ba) {{< /link >}}

ただ、*RDNA 2 /GFX10.3* GPU のメモリに関してははっきりしない所があり[^sienna_cichlid-mem]、また現時点ではエミュレーションモードで dGPU をテストする場合、メモリタイプに GDDR6 が設定されている。  

[^sienna_cichlid-mem]: [Navi21 /Sienna Cichlid のメモリは GDDR6 か HBM2 か、新たに出てきたパッチを読む | Coelacanth's Dream](/posts/2020/07/27/what-sienna_cichlid-hbm2/)

## Radeon Pro 5600M は Navi12 で gfx1011 という話

だったのだけれど。  
これまではその通り、LLVM のドキュメント内にて *gfx1011* を採用している製品に **Radeon Pro 5600M** が記載されていたのだが、それを *gfx1010* に移すパッチが投稿され、組み込まれている。  
{{< link >}} [[AMDGPU] Correct processor names for gfx1010 and gfx1011 · llvm/llvm-project@fe145b6](https://github.com/llvm/llvm-project/commit/fe145b66ecfd98769feef496d47e49781efd6a2e) {{< /link >}}

簡潔に言うと、Geekbench の結果から **Radeon Pro 5600M** は *gfx1011* と認識されており、何かズレが生じているように思える。[^gb5-radeon-pro-5600m]  
ついでに言うと、*Navi12* は *gfx1011* とされており[^navi12-gfx1011]、**Radeon Pro 5600M** は *Navi12* をベースにしていると考えている。  

[^gb5-radeon-pro-5600m]: [Apple Inc. MacBookPro16,4 - Geekbench Browser](https://browser.geekbench.com/v5/compute/1599856)  
[^navi12-gfx1011]: <https://gitlab.freedesktop.org/mesa/mesa/-/blob/283686ad6762182037b708f1b5187129aff0a5dd/src/amd/llvm/ac_llvm_util.c#L173>  

パッチの意味する所は不明。  
今回のパッチが影響するのはドキュメントのみであるため、特にそれといった問題は無いが。  

## ACOバックエンドでの NGG サポートは一旦無効化

この前 [ACOバックエンドでも NGG をサポートする動き | Coelacanth's Dream](/posts/2020/10/04/aco-ngg-gfx10/) という記事を書き、その時に触れたマージリクエストは無事組み込まれたのだが、Vulkan CTS(Conformance Tests) を実行するとランダムで動作が停止するらしく、一旦無効化されることになった。  
{{< link >}} [radv/aco: disable NGG GS support because it randomly hangs the GPU (!7108) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7108) {{< /link >}}

ランダムということは停止する原因が不明ということであり、デフォルトで NGG が有効化されるのはまだ時間が掛かりそうだ。  


