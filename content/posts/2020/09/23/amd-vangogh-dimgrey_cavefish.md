---
title: "新たな AMD RDNA 2 GPU、\"Dimgrey Cavefish\" & \"VanGogh\""
date: 2020-09-23T01:22:25+09:00
draft: false
tags: [ "RDNA_2", "Dimgrey_Cavefish", "VanGogh", "NGG", "Navi23" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Mesa3D に、新たな *RDNA 2 /GFX10.3* GPU、*Dimgrey Cavefish* と *VanGogh* のサポートを追加されたマージリクエストが投稿された。  
OSS にこれらチップの名前が追加されたのは初であり、珍しく UMD(User Mode Driver) である Mesa3D が先行した。Linux Kernel(amd-gfx) に、サポートするためのパッチが投稿される日も近いはずだ。  

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
 > {{< quote >}} [pal/amdgpu_asic_addr.h at 39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0/src/core/imported/addrlib/src/amdgpu_asic_addr.h#L115) {{< /quote >}}

*Dimgrey Cavefish* 、*VanGogh* は、[Sienna Cichlid](/tags/sienna_cichlid)、[Navy Flounder](/tags/navy_flounder) 同様 `gfx1030` に関連付けられ、*RDNA 2 /GFX10.3* GPU とされる。  
*VanGogh* は画家系のコードネームであることから APU と考えられ、聞こえ伝わってくる話もそのようである。  
NGG(Next Generation Geometry) が使用可能かの判定に専用VRAMを持っているかが追加されており、現時点で *VanGogh* で NGG が使えないようになっている。今後 NGG のサポートが追加されるかは不明。  
また、GPU部が *Vega /GFX9* 世代から進んだことで新たに *VanGoghファミリー* が作られたが、`chipRevision` の範囲が `0x01, 0xFF` と既に埋まっており、そのファミリーに属するのは *VanGogh* のみとなる可能性が考えられる。  

*RDNA 2/GFX10.3* 世代の dGPU には他に、[Sienna Cichlid](/tags/sienna_cichlid) 、[Navy Flounder](/tags/navy_flounder) がおり、*色 + 魚* のコードネームとなっている。  
*Cichlid* はカワスズメ科の硬骨魚、*Flounder* は砂底に生息する硬骨魚<del>ヒラメ</del>の英名である。  

{{< ins >}}

*Flounder* はカレイの意でした。大変申し訳ありません。  

{{< /ins >}}

そして *Cavefish* は、洞窟の生活に適応したドウクツギョ科の英名らしく、硬骨魚の *Cichlid* や *Flounder* とは大きく異なっている。果たしてこれが意味する所は。いや特に意味は無い、というのも十分ありうるけれども。  

ちなみに *Dimgrey* は <span style="color: dimgrey;">こんな感じの色</span>。  
*Sienna* は <span style="color: #882D17">こんな感じ</span>。  
*Navy* は <span style="color: #000080">こんな感じ</span>。  
