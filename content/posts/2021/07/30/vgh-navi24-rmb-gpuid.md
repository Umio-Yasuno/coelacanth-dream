---
title: "VanGogh は gfx1033、Navi24 は gfx1034、Rembrandt は gfx1035……"
date: 2021-07-30T00:11:09+09:00
draft: false
tags: [ "VanGogh", "Rembrandt", "Navi24", "gfx1035" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
noindex: false
# summary: ""
---

AMD がオープンソースで開発する [ROCclr (Radeon Open Compute Common Language Runtime)](https://github.com/ROCm-Developer-Tools/ROCclr) と [ROCm-OpenCL-Runtime](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime) レポジトリの developブランチが更新され、そこに *VanGogh, Navi24, Rembrandt* の GPUID を追加するコミットが含まれていた。  
そのメモ書き的なもの。  

 > 		Subject: [PATCH] SWDEV-290306 - [LNX][Navi24][mainline]clinfo test failed on
 > 		 Navi24
 > 		
 > 		Add Navi 24 support
 >      ---
 >
 > 		     {"gfx1030",                "gfx1030",   true,  true,  false,              10, 3,  0,    NONE,   NONE, 2,    32,   1,    256,    64 * Ki, 32},
 > 		     {"gfx1031",                "gfx1031",   true,  true,  false,              10, 3,  1,    NONE,   NONE, 2,    32,   1,    256,    64 * Ki, 32},
 > 		     {"gfx1032",                "gfx1032",   true,  true,  false,              10, 3,  2,    NONE,   NONE, 2,    32,   1,    256,    64 * Ki, 32},
 > 		+    {"gfx1034",                "gfx1034",   true,  false,  false,             10, 3,  4,    NONE,   NONE, 2,    32,   1,    256,    64 * Ki, 32}
 >
 >  {{< quote >}} [SWDEV-290306 - [LNX][Navi24][mainline]clinfo test failed on Navi24 · ROCm-Developer-Tools/ROCclr@600bb35](https://github.com/ROCm-Developer-Tools/ROCclr/commit/600bb356421a3d39d50be1dd149d373506878663) {{< /quote >}}

 > 		Subject: [PATCH] SWDEV-267762 - add RMB(gfx1035) to deviceQuery
 > 		
 > 		---
 > 		+    /* Navi21 */   { "gfx1030",     "gfx1030",     4, 32, 1, 256, 64 * Ki, 32, 10, 3 },
 > 		+    /* Navi22 */   { "gfx1031",     "gfx1031",     4, 32, 1, 256, 64 * Ki, 32, 10, 3 },
 > 		+    /* Navi23 */   { "gfx1032",     "gfx1032",     4, 32, 1, 256, 64 * Ki, 32, 10, 3 },
 > 		+    /* Van Gogh */ { "gfx1033",     "gfx1033",     4, 32, 1, 256, 64 * Ki, 32, 10, 3 },
 > 		+    /* Rembrandt */{ "gfx1035",     "gfx1035",     4, 32, 1, 256, 64 * Ki, 32, 10, 3 },
 >
 > {{< quote >}} [SWDEV-267762 - add RMB(gfx1035) to deviceQuery · RadeonOpenCompute/ROCm-OpenCL-Runtime@bc916ca](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/commit/bc916cac70a5ef0215c602877c7db6dce203de7e) {{< /quote >}}

*Navi24* と *Rembrandt APU* については、オープンソースドライバー (Mesa, Linux Kernel) には出てきていないコードネームではあるが、  
タイミングからして *Beige Goby* に *GPUID: gfx1034* 、*Yellow Carp* に *GPUID: gfx1035* が該当するのではないかと推測した以上、*Beige Goby == Navi24* 、*Yellow Carp == Rembrandt* と考えない訳にはいかないだろう。  
両方の Device (PCI) ID か、ASICごとに割り振られる `external_rev_id` が判明すればそれが合っているかを確かめられるが。  
{{< link >}}[既に 2つの DeviceID が用意されている Yellow Carp APU ファミリー | Coelacanth's Dream](/posts/2021/07/26/yc-apu-two-did/){{< /link >}}

| GPUID | Arch | Codename |
| :-- | :--: | :--: |
| *gfx908* | CDNA | Arcturus/MI100 |
| *gfx90a* | CDNA 2 | Aldebaran/MI200 |
| *gfx940* [^gfx940] | ? | ? |
| *gfx90c* | Vega | Renoir/Lucienne/Cezanne (Green Sardine)[^green_sardine]
| *gfx1010* | RDNA | Navi10 |
| *gfx1011* | RDNA | Navi12 |
| *gfx1012* | RDNA | Navi14 |
| *gfx1013* | RDNA | Cyan Skilfish ?<br>(APU, Navi10-base, Support RayTracing Inst)[^gfx1013] |
| *gfx1030* | RDNA 2 | Sienna Cichlid/Navi21 |
| *gfx1031* | RDNA 2 | Navy Flounder/Navi22 |
| *gfx1032* | RDNA 2 | Dimgrey Cavefish/Navi23 |
| *gfx1033* | RDNA 2 | VanGogh (APU) |
| *gfx1034* | RDNA 2 | Beige Goby?/Navi24 |
| *gfx1035* | RDNA 2 | Yellow Carp?/Rembrandt (APU) |

[^gfx940]: [SWDEV-292904 - Extend HIP coherency tests to gfx940 · ROCm-Developer-Tools/HIP@9fbd19a](https://github.com/ROCm-Developer-Tools/HIP/commit/9fbd19a6759b0ed091562ad286a790783998b88a)
[^green_sardine]: [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/)
[^gfx1013]: [gfx1013 | Coelacanth's Dream](/tags/gfx1013/)
