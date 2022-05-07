---
title: "AMD RDNA 2 世代に搭載される VCN 3 は AV1 HWデコードをサポート"
date: 2020-09-16T04:24:28+09:00
draft: false
tags: [ "RDNA_2", "Sienna_Cichlid" , "Navy_Flounder"]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD *RDNA 2* 世代の GPU、*Sienna Cichlid* 、*Navy Flounder* がマルチメディアエンジンとして備える VCN 3 は AV1コーデックのハードウェアデコードをサポートする。  
そのことが Linux Kernel (amd-gfx) に投稿されたパッチから判明した。  
{{< link >}} [[PATCH 2/4] drm/amdgpu: add VCN 3.0 AV1 registers](https://lists.freedesktop.org/archives/amd-gfx/2020-September/053779.html) {{< /link >}}
{{< link >}} [[PATCH 3/4] drm/amdgpu: use the AV1 defines for VCN 3.0](https://lists.freedesktop.org/archives/amd-gfx/2020-September/053780.html) {{< /link >}}

AV1 のHWデコードと次世代 GPUに関しては、以前より Intel の次世代 GPUアーキテクチャ、[Gen12](/tags/gen12) はサポートすることが分かっており[^gen12-av1-dec]、NVIDIA もまた先日発表した、Ampereアーキテクチャの RTX 3000シリーズでサポートすることを明らかにしている。  
そのため、AMD の次世代 GPU、*RDNA 2* もそれに続いて AV1 HWデコードをサポートするか、または遅れるかが注目されていた。  
だが、今回のパッチでサポートすることが判明し、不安は払拭された。  
VCN 3 は *RDNA 2* でないと搭載できない、といった理由は無いため、今後登場する AMD の次世代 APU に搭載される可能性も考えられる。  

[^gen12-av1-dec]: [Intel Gen12 GPU は AV1コーデックのHWデコードをサポート | Coelacanth's Dream](/posts/2020/07/09/intel-gen12-av1-decode/)

また、今回のパッチは Linux Kernel側の対応で、実際に AV1 HWデコードを実行するには libdrm や UMD(User Mode Driver) である [RadeonSI](/tags/radeonsi) の対応も必要となる。  
VRS(Variable Rate Shader) の対応を示すレジスタ情報も追加されているが[^gfx103-vrs]、こちらも同様に UMD側の対応が必要である。  

[^gfx103-vrs]: [[PATCH 1/4] drm/amdgpu: add the GC 10.3 VRS registers](https://lists.freedesktop.org/archives/amd-gfx/2020-September/053778.html)

*Sienna Cichlid* の DeviceID もようやく? 追加されており、以下 6個がそうなる。  
実験的なサポートを意味するフラグ `AMD_EXP_HW_SUPPORT` が無いため、Linux Kernel (amd-gfx) への DeviceID 追加は意図的に遅らせたと考えられるが、まあ個人の邪推である。  
素直に次世代 GPU のサポートが進んでおり、登場が近いとも捉えるべきだろう。  
あとは、発売してすぐに Linux で問題なく動作することを願う。  

 >       +	/* Sienna_Cichlid */
 >       +	{0x1002, 0x73A0, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_SIENNA_CICHLID},
 >       +	{0x1002, 0x73A2, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_SIENNA_CICHLID},
 >       +	{0x1002, 0x73A3, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_SIENNA_CICHLID},
 >       +	{0x1002, 0x73AB, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_SIENNA_CICHLID},
 >       +	{0x1002, 0x73AE, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_SIENNA_CICHLID},
 >       +	{0x1002, 0x73BF, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_SIENNA_CICHLID},
 >
 > {{< quote >}} [[PATCH 4/4] drm/amdgpu: add device ID for sienna_cichlid (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-September/053781.html) {{< /quote >}}

