---
title: "RadeonSI、RADV が AMD Navy Flounder をサポート"
date: 2020-07-29T01:56:12+09:00
draft: false
tags: [ "RDNA_2", "Navy_Flounder", "Navi22" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMDGPU向けオープンソース・ドライバー、**RadeonSI (OpenGL)** と **RADV(Vulkan)** に AMD の次世代 *RDNA 2* GPU、*Navy Flounder* をサポートするパッチ/マージリクエストが投稿された。  
{{< link >}} [amd: Navy Flounder support, enable displayable DCC on gfx10.3 (!6100) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6100/diffs?commit_id=3e1090e53ddc9881f4c0f9c4a89f1ad0b213b289) {{< /link >}}

しかし、GFX10.3世代での主な追加点、変更点は [Sienna Cichlid](/tags/sienna_cichlid) サポートの時点で作業が行なわれたためか、今回のパッチから読み取れることはそう多くない。  

{{< pindex >}}

 * [やっぱり Navi22](#navi22)
 * [gfx1030 に関連付けられる](#gfx1030)
 * [余談](#digression)
    * [RenderBackend でも他と揃わない Navi10](#navi10-rb)

{{< /pindex >}}

## やっぱり Navi22 {#navi22}
[Linux Kernel へのパッチが投稿された際](/posts/2020/07/15/amd-next-gen-gpu-navy_flounder/)にも書いたが、`chipRevision` の範囲が、以前 [GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal) 一時的に組み込まれた *Navi22* のコードのものと一致するため[^pal-navi22]、やはり `Navy Flounder == Navi22` であると考えられる。  

 >       #define AMDGPU_NAVY_FLOUNDER_RANGE      0x32, 0x3C
 >
 > 引用元: <cite>[amd: Navy Flounder support, enable displayable DCC on gfx10.3 (!6100) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6100/diffs?commit_id=3e1090e53ddc9881f4c0f9c4a89f1ad0b213b289#diff-content-c1f3b41aade2b5733c9e6855b5288f64c3048688)</cite>
 >
 >       #define AMDGPU_NAVI22_RANGE     0x32, 0x3C
 >
 > 引用元: <cite>[pal/amdgpu_asic_addr.h at 39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0/src/core/imported/addrlib/src/amdgpu_asic_addr.h#L111)</cite>


しかし、AMD GPU のオープンソース・ドライバーは Kernel側も User側も *Navi2x* は使わず、基本「色 + 硬骨魚」のコードネームで通しており、[llvm/llvm-project](https://github.com/llvm/llvm-project)のようなコンパイラバックエンドは ASIC ごとのサポートではなくマイクロアーキテクチャのサポートであるため、GPUID で示される。  
{{< link >}} [AMD GPU の GPU ID は何を意味するか | Coelacanth's Dream](/posts/2020/06/22/amdgpu-gpuid-mean/) {{< /link >}}
そのため、自分の範囲では *Navi21* とか *Navi22* といったコードネームに拘りを持つ必要は無く、支障が無い限り、コード中で使われているコードネーム「色 + 硬骨魚」を用いていくつもりでいる。  
コードネームから製品展開を推測する気はないし。  

[^pal-navi22]: [pal/amdgpu_asic_addr.h at 39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0/src/core/imported/addrlib/src/amdgpu_asic_addr.h#L111)

## gfx1030 に関連付けられる {#gfx1030}
*Navy Flounder* の GPUID だが、*Sienna Cichlid* と同じ *gfx1030* とされている。[^navy_flounder-gfx1030]  
だが、自分が確認できる情報[^navi22-navi23-vangogh-gpuid]では *Navi22* は *gfx1031* か *gfx1032* (恐らく前者)、インターネット上で言われている情報では *gfx1031* とされている。  

[^navy_flounder-gfx1030]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6100/diffs?commit_id=3e1090e53ddc9881f4c0f9c4a89f1ad0b213b289#diff-content-fbbe09bc487101e3f2f96c0b8fe543c915fa99bf>
[^navi22-navi23-vangogh-gpuid]: [P4 to Git Change 2008166 by vsytchen@vsytchen-remote-ocl-win10 on 201… · ROCm-Developer-Tools/ROCclr@df2e0b9](https://github.com/ROCm-Developer-Tools/ROCclr/commit/df2e0b9ae27f43ba8e23a3afa185c16dd3bb5ebc)

ここで考えられるのは、LLVM 側の対応がそこまで追い付いていないため、仮のコードとして記述した、  
もしくは、*Sienna Cichlid* と *Navy Flounder* の間にマイクロアーキテクチャレベルの対応命令やバグに違いは無く、両方を *gfx1030* としても問題ないか。  
別チップ/ASIC であってもマイクロアーキテクチャは同一、ということは普通にある。  

## 余談 {#digression}
### RenderBackend でも他と揃わない Navi10 {#navi10-rb}
同じ *Navi1x /GFX10.1* 世代であっても、命令の対応範囲が *Navi10* と *Navi12 /Navi14* で異なり、*Navi10* は一部命令に対応していないことは、これまで何度か触れたことがあったが、  
{{< link >}} [Navi12情報近況 (2020-03-09) & 推測 | Coelacanth's Dream](/posts/2020/03/09/navi12-recent-info/) {{< /link >}}
RenderBackendの機能にも差異があるらしく、DCC(Delta Color Compression) 機能の一部を *Navi12 /Navi14* はサポートし、*Navi10* はサポートしないとなっている。[^navi12-14-dcc]  

[^navi12-14-dcc]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6100/diffs?commit_id=2ba4fe2f1f1d7bcfc1222d3141b4ce981455d498>

その機能についてコードを漁ってみると、独立した 64Byte のブロックで DCC を行なうものらしいことがわかる。[^dcc-64b-block]  
それと、*RDNA 2 /GFX10.3* 世代、*Sienna Cichlid* と *Navy Flounder* ではそのブロックサイズの対応が拡張され、128B のブロックもサポートするとなっている。  

[^dcc-64b-block]: [src/amd/common/ac_surface.c · 87ecfdfbf0a8448d1475e6da15175e68bdeb933b · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/87ecfdfbf0a8448d1475e6da15175e68bdeb933b/src/amd/common/ac_surface.c#L1991)
