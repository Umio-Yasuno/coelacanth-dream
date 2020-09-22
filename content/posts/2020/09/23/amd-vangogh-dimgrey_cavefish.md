---
title: "新たな AMD RDNA 2 GPU、\"Dimgrey Cavefish\" & \"VanGogh\""
date: 2020-09-23T01:22:25+09:00
draft: false
tags: [ "RDNA_2", "Dimgrey Cavefish", "Van_Gogh" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Mesa3D に、新たな *RDNA 2 /GFX10.3* GPU、*Dimgrey Cavefish* と *Van Gogh* のサポートを追加されたマージリクエストが投稿された。  

 * [amd: add support for more chips - Dimgrey Cavefish and VanGogh (!6820) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6820)

*Dimgrey Cavefish* は `chipRevision` の範囲が、以前 [GPUOpen-Drivers/pal: Platform Abstraction Library](https://github.com/GPUOpen-Drivers/pal) に投稿された *Navi23* と一致するため、内部的なコードネームは *Navi23* ではないかと思われる。  

 >       #define AMDGPU_DIMGREY_CAVEFISH_RANGE   0x3C, 0x46
 >
 >       #define AMDGPU_VANGOGH_RANGE    0x01, 0xFF
 >
 > {{< quote >}} [amd: add support for more chips - Dimgrey Cavefish and VanGogh (!6820) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6820/diffs){{< /quote >}}

 >       #if ADDR_NAVI23_BUILD
 >       #define AMDGPU_NAVI23_RANGE     0x3C, 0x46
 >       #endif
 >
 > {{< quote >}} [pal/amdgpu_asic_addr.h at 39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0/src/core/imported/addrlib/src/amdgpu_asic_addr.h#L111) {{< /quote >}}

*Dimgrey Cavefish* 、*VanGogh* は、[Sienna Cichlid](/tags/sienna_cichlid)、[Navy Flounder](/tags/navy_flounder) 同様 `gfx1030` に関連付けられ、*RDNA 2 /GFX10.3* GPU と考えられる。  
*VanGogh* は画家系のコードネームであり APU と考えられ、伝え聞こえてくる話もそのようになっている。  
NGG(Next Generation Geometory) が使用可能かの判定に専用VRAMを持っているかが追加されており、現時点で *VanGogh* で NGG が使えないようになっている。今後 NGG のサポートが追加されるかは不明。  

*Cavefish* は、洞窟の生活に適応したドウクツギョ科の英名であり、硬骨魚の *Cichlid* や *Flounder* とは大きく異なっている。これが意味する所は。  
ちなみに *Dimgrey* は <span style="color: dimgrey;">こんな感じの色</span>。  
*Sienna* は <span style="color: #882D17">こんな感じ</span>。  
*Navy* は <span style="color: #000080">こんな感じ</span>。  
