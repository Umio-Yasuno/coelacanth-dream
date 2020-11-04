---
title: "Linux Kernel に \"Fine Grain Clock Gating\" のサポートが追加される"
date: 2020-11-04T17:22:25+09:00
draft: false
tags: [ "RDNA_2", "VanGogh" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU", "AMD" ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel (amd-gfx) に `FGCG (Fine Grain Clock Gating)` のサポートを追加するパッチが投稿された。  
{{< link >}} [[PATCH 1/3] drm/amdgpu: Add GFX Fine Grain Clock Gating flag](https://lists.freedesktop.org/archives/amd-gfx/2020-November/055539.html) {{< /link >}}  
クロックゲーティングは動作させる必要のない回路へのクロック供給を止める (最小にする) ことでスイッチング動作を減らし、ダイナミック消費電力を減らす省電力技術。  
きめ細かい (Fine Grain) クロックゲーティングというのは、*RDNA 2 / GFX10.3* GPU 発表時にアピールされていた点でもある。[^rdna_2-youtube]  

[^rdna_2-youtube]: <https://youtu.be/oHpgu-cTjyM?t=373>

FGCG は *RDNA / GFX10* の機能としているが、現在有効のフラグは *VanGogh APU (RDNA 2)* のみに追加されている。  
{{< link >}} [[PATCH 3/3] drm/amdgpu: Enable FGCG for Vangogh](https://lists.freedesktop.org/archives/amd-gfx/2020-November/055541.html) {{< /link >}}
*RDNA / GFX10* GPU *Navi10 / Navi12 / Navi14* にも、アピールされていた *RDNA 2 / GFX10.3* GPU [Sienna Cichlid](/tags/sienna_cichlid)、[Navy Flounder](/tags/navy_flounder)、 [Dimgrey Cavefish](/tags/dimgrey_cavefish) にも有効のフラグは追加されていない。  

*RDNA 2 / GFX10.3* 世代で追加された省電力機能には、Shader Array が非対称である場合の効率を上げるというものがある。  
{{< link >}} [AMD RDNA 2 ノート　―― 非対称な ShaderArray、gpu_info、XSS 【2020/09/23】 | Coelacanth's Dream](/posts/2020/09/23/rdna_2-note-2020-09-23/#shader-array) {{< /link >}} 
Shader Engine 内の Shader Array 2基それぞれの CU数が非対称である場合、Wave の発行を少ない方に合わせることで、稼働させる CU数を同じに、Shader Array を実質対称とすることで効率を上げられる。  
その CU 分の消費電力は減らせ、他の CU のクロック向上に回すこともできる。  
FGCG はこうした機能とも組み合わせられ、より活用できるのではないかと思われる。  

また、現時点で発表されている *RDNA 2 / GFX10.3* GPU で、Shader Array が非対称となり、上記の省電力機能が活用できるものには **RX 6800 XT** がおり、   
**RX 6800 XT** の WGP数は総36基、4 Shader Engineそれぞれの WGP数は 9基、そして Shader Engine あたりの Shader Array は 2基であるため、WGP が 4基と 5基の Shader Array に分かれることとなる。  
