---
title: "AMD Navy Flounder は gfx1031 という話"
date: 2020-09-25T15:27:24+09:00
draft: false
tags: [ "RDNA_2", "Navy_Flounder" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

それだけの話なんだけど、ヒラメをカレイと間違えるようなうっかりではない方向に話が行ったりする。  

ROCm のデバッグツール [ROCm-Developer-Tools/ROCgdb](https://github.com/ROCm-Developer-Tools/ROCgdb)、デバッガーAPI [ROCm-Developer-Tools/ROCdbgapi](https://github.com/ROCm-Developer-Tools/ROCdbgapi) に GPU(GFX) ID に `gfx1030`, `gfx1031` のサポートが追加された。  
{{< link >}} [Add support for gfx1030/gfx1031 · ROCm-Developer-Tools/ROCdbgapi@deb6d8a](https://github.com/ROCm-Developer-Tools/ROCdbgapi/commit/deb6d8ab8d62072bdc6707d157fd226da3a95691) {{< /link >}}
{{< link >}} [Add support for gfx1030/gfx1031 · ROCm-Developer-Tools/ROCgdb@0f5f27a](https://github.com/ROCm-Developer-Tools/ROCgdb/commit/0f5f27ac6622e2ff6a71a6245644f704f50d85b7) {{< /link >}}

その中に *Sienna Cichlid* は `gfx1030` 、*Navy Flounder* は `gfx1031` とする記述がある。  

 >        { "sienna_cichlid", EF_AMDGPU_MACH_AMDGCN_GFX1030, 0 },
 >        { "navy_flounder", EF_AMDGPU_MACH_AMDGCN_GFX1031, 0 } };
 >
 > {{< quote >}} [ROCdbgapi/os_driver.cpp at deb6d8ab8d62072bdc6707d157fd226da3a95691 · ROCm-Developer-Tools/ROCdbgapi](https://github.com/ROCm-Developer-Tools/ROCdbgapi/blob/deb6d8ab8d62072bdc6707d157fd226da3a95691/src/os_driver.cpp) {{< /quote >}}

また、*Navi21* は `gfx1030` 、*Navi22* は `gfx1031` という記述もあったが、OSS においては *Navi2x* ではなく 色+魚 なコードネームを使う方向性でいるのだからそれで統一してほしい、とは思う。  


 >       @c ``Navi 21'' which is displayed as @samp{sienna_cichlid} by @value{GDBN} and
 >       @c denoted as @samp{gfx1030} by the compiler.
 >       @c
 >       @c @item
 >       @c ``Navi 22'' which is displayed as @samp{navy_flounder} by @value{GDBN} and
 >       @c denoted as @samp{gfx1031} by the compiler.
 >
 > {{< quote >}} [Add support for gfx1030/gfx1031 · ROCm-Developer-Tools/ROCgdb@0f5f27a](https://github.com/ROCm-Developer-Tools/ROCgdb/commit/0f5f27ac6622e2ff6a71a6245644f704f50d85b7#diff-ddd44f7e2bba0c3ed8b4c9be4feda726) {{< /quote >}}

 それだけのお話。  
