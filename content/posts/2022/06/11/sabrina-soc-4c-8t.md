---
title: "最大 4-Core/8-Thread な AMD Sabrina SoC と Mendocino SoC"
date: 2022-06-11T18:09:27+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Coreboot", "Mendocino", "Sabrina" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Felix Held 氏より Coreboot に投稿されたパッチにて、*AMD Sabrina SoC* は最大 4-Core/8-Thread という CPU構成であることが明かされた。  
これまでの情報から、*Sabrina SoC* は GPU部に *RDNA 2 アーキテクチャ* を採用し、メモリは LPDDR5 をサポートしている。  

 * [soc/amd/sabrina: change MAX_CPUS to 8 (I627ed78f) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65068/2)

 > 		The Sabrina APU has a maximum configuration of 4 physical cores with 2
 > 		threads each, so a total of 8 CPU cores.
 >
 > {{< quote >}} [soc/amd/sabrina: change MAX_CPUS to 8 (I627ed78f) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65068/2) {{< /quote >}}

## Mendocino APU {#mendocino}

AMD は COMPUTEX 2022 にて、エントリーレベル向けに新たな APU/SoC *Mendocino* を発表した。  
*Mendocino* は CPUに *Zen 2* 4-Core/8-Thread を備え、GPU は *RDNA 2 アーキテクチャ* を採用、TSMC 6nmプロセスで製造される。  

 * [AMD Showcases Industry-Leading Gaming, Commercial, and Mainstream PC Technologies at COMPUTEX 2022 :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1069/amdshowcases-industry-leading-gaming-commercial-and)
 * [AMD Ryzen 6000 Series Bring Laptop Battery Life th... - AMD Community](https://community.amd.com/t5/business/amd-ryzen-6000-series-bring-laptop-battery-life-the-competition/ba-p/525970/page/2)

*Sabrina SoC* と *Mendocino SoC* は CPU、GPU、それと Chromebookボードが開発するという点も共通することから、そう深く考えずとも両者同じ APU/SoC、シリコンを指しているとするのが自然に思う。  

*Mendocino SoC* については、*Sabrina SoC* に関するパッチが投稿されるより前に、Bao Zheng 氏より PSP部に追加された ISH (Image Slot Header) をサポートするパッチが Coreboot に投稿されていた。[^mendocino-ish]  
{{< link >}} [Coreboot に Family 17h Model A0h APU/SoC のサポートが追加 ―― Sabrina SoC | Coelacanth's Dream](/posts/2022/01/12/fam17h-moda0h-amd-sabrina-soc/#mendocino) {{< /link >}}
*Sabrina SoC* のサポートが進められるようになってから *Mendocino SoC* の名前は Coreboot において出されなくなり、そして *Sabrina SoC* に向けても PSP ISH をサポートするパッチが投稿された。[^sabrina-ish]  
また、*Sabrina SoC* と *Mendocino SoC* の PSP ID は同じものとされ、PSP部においても一致している。[^psp-id]  

[^mendocino-ish]: [amdfwtool: Add ISH header support for A/B recovery layout (I4710df40) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59384/1)
[^sabrina-ish]: [util/amdfwtool: use ISH support for Sabrina SoC (I9e630885) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/63186/3)
[^psp-id]: [util/amdfwtool: add Sabrina SoC type (Ibe52b443) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/63185/3)

*Sabrina* と *Mendocino* という 2つのコードネームがある理由は不明だが、一応 GPU部のコードネームは *VanGogh* だが、SoC に対するコードネームや VBIOS名に *Aerith* が使われるという前例はある。  
個人的には、最初は *Mendocino* というコードネームだったが、Intel の過去 CPU のコードネームと被ることを気にして *Sabrina* というコードネームを新たに付けたのではと考えていたが、COMPUTEX 2022 でコードネーム *Mendocino* が正式発表された。  
Coreboot 内の *Mendocino SoC* に関するコードを *Sabrina SoC* に置き換えるようなパッチは投稿されていないことから、ほとんど同じ構成だが別シリコン、もしくはそれぞれで別プラットフォームを指すコードネームである可能性もある。  

