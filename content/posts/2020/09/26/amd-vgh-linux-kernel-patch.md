---
title: "Linux Kernel に AMD の次世代 APU 「VanGogh」 をサポートするパッチが投稿される"
date: 2020-09-26T05:13:48+09:00
draft: false
tags: [ "VanGogh", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

GPU部が *RDNA 2/GFX10.3* となる AMD の次世代 APU *VanGogh* をサポートするパッチが Linux Kernel(amd-gfx) に投稿された。  
{{< link >}} [[PATCH 00/45] Add support for vangoh](https://lists.freedesktop.org/archives/amd-gfx/2020-September/054216.html) {{< /link >}}

先日には Linux Kernel に先行して Mesa3D(UMD) に *VanGogh* のサポートが追加されている。  
{{< link >}} [新たな AMD RDNA 2 GPU、"Dimgrey Cavefish" & "VanGogh" | Coelacanth's Dream](/posts/2020/09/23/amd-vangogh-dimgrey_cavefish/) {{< /link >}}

## DCN3.01
ディスプレイ部には DCN3.01 を搭載し、*RDNA 2 dGPU* の DCN3.0 よりほんの少し進んだバージョンとなるが、違いもそれに沿って eDP とメモリの対応を追加したくらいではないかと思う。  
*Raven APU* 、*Renoir APU* 同様にディスプレイコントローラーは 4基とされ、DSC の数も *Renoir* から変わらず 3基となっている。[^vgh-dc4]  
{{< link >}} [[PATCH 43/45] drm/amd/display: Add dcn3.01 support to DC](https://lists.freedesktop.org/archives/amd-gfx/2020-September/054258.html) {{< /link >}}

[^vgh-dc4]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/dcn301/dcn301_resource.c?h=amd-staging-drm-next-vangogh#n770>

## VCN3.0 {#vgh-vcn3}
*VanGogh* は *RDNA 2 dGPU* 、*Sienna Cichlid* 、*Navy Flounder* と同じくマルチメディアエンジンには VCN3.0 を搭載する。  
VCN3.0 は AV1 HWデコードに対応するため[^vcn3-av1]、後述の LPDDR5メモリ対応と合わせて、機能面で *Intel Tiger Lake* に追い付いたと言える。  
{{< link >}} [[PATCH 23/45] drm/amdgpu: enable vcn3.0 for van gogh](https://lists.freedesktop.org/archives/amd-gfx/2020-September/054237.html) {{< /link >}}

[^vcn3-av1]: [AMD RDNA 2 世代に搭載される VCN 3 は AV1 HWデコードをサポート | Coelacanth's Dream](/posts/2020/09/16/amd-vcn_3-av1-dec/)

## LPDDR5メモリをサポート {#vgh-ddr5}
*VanGogh* は LPDDR5メモリをサポートするとされる。  
同時に DDR5 のメモリタイプが追加されているが、ドライバー部では LPDDR4 は DDR4 と、LPDDR5 は DDR5 として判定されるため、DDR5メモリまでサポートするかは不明。  
まだテストが行なわれていないだけかもしれないが。  
`vg_clk_mgr.c` ファイルに LPDDR5 と DDR4 のテーブルが用意されているため、DDR4メモリをサポートしている可能性はある。[^vgh-ddr4]  
{{< link >}} [[PATCH 10/45] drm/amdgpu: add uapi to define van gogh memory type](https://lists.freedesktop.org/archives/amd-gfx/2020-September/054225.html) {{< /link >}}
{{< link >}} [[PATCH 13/45] drm/amdgpu: get the correct vram type for van gogh](https://lists.freedesktop.org/archives/amd-gfx/2020-September/054227.html) {{< /link >}}

[^vgh-ddr4]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/clk_mgr/dcn301/vg_clk_mgr.c?h=amd-staging-drm-next-vangogh#n545>


PCI ID は `0x163F` となり、AMDGPUファミリーは新たな *VanGoghファミリー* に属する。  
{{< link >}} [[PATCH 45/45] drm/amdgpu: add van gogh pci id](https://lists.freedesktop.org/archives/amd-gfx/2020-September/054259.html) {{< /link >}}

