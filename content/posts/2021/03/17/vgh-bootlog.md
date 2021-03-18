---
title: "AMD VanGogh APU ブートログ"
date: 2021-03-17T08:06:16+09:00
draft: false
tags: [ "VanGogh", "Zen_2", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

2021/03/11 に、Alex Deucher 氏による不具合報告の中で *VanGogh APU* の Linux Kernel ブートログが投稿されていた。  
Alex Deucher 氏は AMD のソフトエンジニアであり、AMD GPU の Linux Kernelドライバーの開発を担当している。  

 * [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html)

使用しているボードは *AMD Chachani-VN/Chachani-VN* 。この名前は過去にも *VanGogh* 関連のパッチの中で登場していた。[^chachani-vn]  
*Chachani* はプラットフォーム、開発用のリファレンスボードに対するコードネームで、*VN* は *VanGogh* の略だろう。  
プラットフォーム・コードネームには、モバイル向けリファレンスボード (FP6パッケージ) に *Celadon* または *Majolica* [^majolica] 、AM4ソケットを採用するデスクトップ向けに *Mytle* [^myrtle]、(恐らくは) デスクトップ向け APU に *Artic* 等が使われてきている。  

[^myrtle]: <https://drivers.amd.com/relnotes/amd_nvme_sata_raid_quick_start_guide_for_windows_operating_systems.pdf>
[^majolica]: [Lucienne/Cezanne APU を搭載する Chromebookボード　―― Guybrush、Mancomb | Coelacanth's Dream](/posts/2020/11/22/lcn-czn-fp6-chromebook-board/)


 >        [  135.183257] Hardware name: AMD Chachani-VN/Chachani-VN, BIOS WCH1304N 03/04/2021
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

[^chachani-vn]: [[PATCH 1/2] drm/amdgpu/display: fix the NULL pointer reference on dmucb on dcn301](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054813.html)

{{< pindex >}}
 * [Zen 2](#zen_2)
 * [怪しいメモリ](#mem)
 * [lspci -vv](#lspci)
    * [ISP (Image Signal Processor)](#isp)
    * [PCI/DeviceID](#pci_id)
 * [VanGogh APU Spec](#spec)
{{< /pindex >}}

## Zen 2 {#zen_2}

プロセッサ名は **AMD Eng Sample: 100-000000405-03_35/24_N** 、その名の通りサンプリング品とされる。  
CPU の判別等に使われる Family, Model は `family: 0x17 (23), model: 0x90 (144)` 。  

 >        [    0.353153] smpboot: CPU0: AMD Eng Sample: 100-000000405-03_35/24_N (family: 0x17, model: 0x90, stepping: 0x1)
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

*VanGogh* の TLBエントリ数は以下のようになっており、*Zen 2 アーキテクチャ* のそれと一致する。`family: 0x17` ということとも矛盾しない。  

 >        [    0.230926] Last level iTLB entries: 4KB 1024, 2MB 1024, 4MB 512
 >        [    0.231010] Last level dTLB entries: 4KB 2048, 2MB 2048, 4MB 1024, 1GB 0
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

CPUスレッド数は 8-Thread 認識されており、以前 Linux Kernel (amd-gfx) に投稿されたパッチのコメントから[^vgh-power]、SMTを有効にした、4-Core/8-Thread 構成だと考えられる。その時投稿されたパッチは、GPUドライバー側でサポートする電力管理機能で、GPU だけでなく CPU も管理するよう対応を拡張する、というものだった。  
*Zen 2 アーキテクチャ* を採用する APU には [Renoir](/tags/renoir), [Lucienne](/tags/lucienne) がいるが、両者は 8-Core/16-Thread (2-CCX) の構成を取っている。対し、*VanGogh* は 4-Core/8-Thread (1-CCX) の構成と考えられ、CCX間の通信によるレイテンシが発生しない、コンパクトな設計となる。  
それと、今回の *VanGogh* はベースクロック 2.4 GHz (2400 MHz) で動作している。ブーストクロックはプロセッサ名とその法則から 3.4 GHz と思われる。  

[^vgh-power]: [VanGogh APU がサポートする "Fine Grain Clock Gating" は CPU にも適用　―― 最大 4コアか | Coelacanth's Dream](/posts/2021/01/08/amd-vgh-cpu-fgcg/) / [[PATCH 7/7] drm/amd/pm: implement processor fine grain feature for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058001.html)

 >        [    0.583839] x86: Booting SMP configuration:
 >        [    0.584209] .... node  #0, CPUs:        #1  #2  #3  #4  #5  #6  #7
 >        [    0.773093] smp: Brought up 1 node, 8 CPUs
 >        [    0.773952] smpboot: Max logical packages: 2
 >        [    0.774378] smpboot: Total of 8 processors activated (38401.47 BogoMIPS)
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}


## 怪しいメモリ {#mem}

このブートログでは GPU部も有効化されており、そこからも *VanGogh APU* であることが確認できる。  

 >        [   99.930720] [drm] initializing kernel modesetting (VANGOGH 0x1002:0x163F 0x1002:0x0123 0xAE).
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

メモリバス幅とメモリタイプも出力されているが、これは明らかに怪しい情報である。  

 >        [   99.984978] [drm] Detected VRAM RAM=1024M, BAR=1024M
 >        [   99.984981] [drm] RAM width 256bits DDR5
 >        [   99.985223] [drm] amdgpu: 1024M of VRAM memory ready
 >        [   99.985233] [drm] amdgpu: 3072M of GTT memory ready.
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

DDR5 という点は、Linux Kernel (amd-gfx) に投稿された *VanGogh* 関連のパッチから、*VanGogh* は LPDDR5メモリをサポートすることが分かっている。そして Kernelドライバー部では LPDDR5 も DDR5 も、メモリタイプとしては両方 DDR5 として認識されるため、搭載しているのがどちらかはともかく、表示自体はおかしくない。これは誤検出などではなく、仕様である。[^vgh-ddr5]  

 >        		case Ddr4MemType:
 >        		case LpDdr4MemType:
 >        			vram_type = AMDGPU_VRAM_TYPE_DDR4;
 >        			break;
 >        		case Ddr5MemType:
 >        		case LpDdr5MemType:
 >        			vram_type = AMDGPU_VRAM_TYPE_DDR5;
 >        			break;
 >
 > {{< quote >}} [linux/amdgpu_atomfirmware.c at 12f2df72205fe348481d941c3e593e8068d2d23d · torvalds/linux](https://github.com/torvalds/linux/blob/12f2df72205fe348481d941c3e593e8068d2d23d/drivers/gpu/drm/amd/amdgpu/amdgpu_atomfirmware.c) {{< /quote >}}

[^vgh-ddr5]: [drm/amdgpu: get the correct vram type for van gogh · torvalds/linux@15c90a1](https://github.com/torvalds/linux/commit/15c90a1fbcb1621be0ba492bc5d7b88edf3d2e22)

怪しいのは 256-bit幅というので、ブートログから搭載されているメモリサイズは 7200304K {{< comple >}} 約 8 GB/7.45 GiB、予約分等があるため若干少なく表示される {{< /comple >}} と読み取れるのに、メモリバス幅が 256-bit とやけに広いのは不自然だ。  
メモリバス幅は VBIOS から読み取った値を表示するため、VBIOS が開発途中で、間違った値が表示されている可能性がある。他のブートログでは GPUドライバーが無効化されているため、開発途中である可能性は高い。GPU部の ShaderEngine、ShareArray、CU の構成情報等、出力されていない情報がいくつかあることもそれを裏付けている。  
一応、もっともらしいメモリ構成を考えるならば、LPDDR5 64-bit幅 (64Gb/8GB) とかだろうか。  

また、VBIOS に *AMD Aerith* というコードネームが使われているが、*VanGogh* の名を使わなかった理由は不明。通常はシリアルナンバーのようなものか、その APU/GPU のコードネームが入ったものが使われる。  

 >        [   99.983800] amdgpu 0000:04:00.0: amdgpu: Fetched VBIOS from VFCT
 >        [   99.983833] amdgpu: ATOM BIOS: 113-AMDAerith-004
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

## lspci -vv {#lspci}

関連スレッドには他のブートログもアップロードされており、そこでは GPUドライバーは無効化されていたが、`lspci`, `lsusb` の実行結果も含んだものがあった。  

 * <https://lists.freedesktop.org/archives/amd-gfx/2021-March/060720.html> 

<!--
 * <https://lists.freedesktop.org/archives/amd-gfx/2021-March/060616.html>
-->

### ISP (Image Signal Processor) {#isp}

`lspci` の実行結果中に、*Signal processing controller* というデバイスがあった。  
これはイメージセンサーから信号を取り込み、画像処理を行う ISP (Image Signal Processor) と考えられ、GPUドライバーが電力管理機能を適用する各種 IP にも ISP はあった。  
実行結果中からは確認できなかったが、*VanGogh* に内蔵される IP には、それ以外に CVIP というのがあり、これは *Computer Vison Image Processing/Processor* ではないかと思われる。  
CVIP では、ISP が画像処理したデータを元に、スケーリングやフィルターの適用、推論によるオブジェクト検出等を行う。[^cvip]  

これら ISP、CVIP を搭載した目的として、*VanGogh* は組み込みやエッジAI用途が想定されているのだろう。  
4-Core/8-Thread という現行の *Renoir/Lucienne/Cezanne APU* と比較して小さい規模から、 *VanGogh* をベースに、**Ryzen Embedded R1000シリーズ** の次世代、**R2000シリーズ** を構築することも考えられる。  
現 **R1000シリーズ** は 14nmプロセスで製造される [Raven2 APU](/tags/raven2) で構築されている。*Raven2* は、CPU Zen 2-Core/4-Thread、GPU Vega 3CU と規模が控えめの SoC であり、そうした特徴は *VanGogh* に近いものがある。  

また、*VanGogh* は GPU部に *RDNA 2 アーキテクチャ* を採用しており、*RDNA 2* では INT4、INT8 といった低精度でのドット積を処理する命令がサポートされている。それらを推論処理に活用することも可能だろう。  

[^cvip]: [CVIP | Computer Vision and Image Processing](http://cvip.computing.dundee.ac.uk/) / [Combining an ISP and Vision Processor to Implement Computer Vision - Edge AI and Vision Alliance](https://www.edge-ai-vision.com/2019/01/combining-an-isp-and-vision-processor-to-implement-computer-vision/)

### PCI/DeviceID {#pci_id}

`lspci` 実行結果から読み取れたデバイスとその PCI/DeviceID をまとめた表が以下。  
認識されているデバイスはいくつか *Renoir APU* と同じものが見受けられ、また各デバイスのリンク情報は PCIe Gen3 となっていた。  

| PCI/DeviceID | VanGogh |
| :-- | :--:         |
| 145a | PCIe Dummy Function |
| 15e3 | Audio Processor – HD Audio Controller (Standalone AZ) (Renoir) |
| 1632 | PCIe® Dummy Host Bridge (Renoir) |
| 163a | USB controller |
| 163b | USB controller |
| 163f<br>(VendorID: 1002) | Internal GPU (GFX) |
| 1640<br>(VendorID: 1002) | Audio device /<br>Display HD Audio Controller (GFXAZ) |
| 1645 | Host bridge                   |
| 1646 | IOMMU                         |
| 1647 | PCI bridge                    |
| 1648 | VanGogh Root Complex          |
| 1649 | Encryption controller/<br>VanGogh PSP/CCP |
| 164a | Signal processing controller  |
| 1660 | Host bridge /<br>Data Fabric: Device 18h; Function 0 |
| 1661 | Host bridge /<br>Data Fabric: Device 18h; Function 1 |
| 1662 | Host bridge /<br>Data Fabric: Device 18h; Function 2 | 
| 1663 | Host bridge /<br>Data Fabric: Device 18h; Function 3 |
| 1664 | Host bridge /<br>Data Fabric: Device 18h; Function 4 | 
| 1665 | Host bridge /<br>Data Fabric: Device 18h; Function 5 | 
| 1666 | Host bridge /<br>Data Fabric: Device 18h; Function 6 |
| 1667 | Host bridge /<br>Data Fabric: Device 18h; Function 7 |

## VanGogh APU Spec {#spec}

|                   | AMD VanGogh |
| :--               | :--: |
| Platform Codename | Chachani<br>(Aerith?) |
| CPU Arch | Zen 2 |
| CPU Core/Thread   | 4/8? |
| CPU Clock Range<br>(default) | 1.4GHz - 3.5GHz? |
| GPU Arch | RDNA 2/GFX10.3<br>(gfx1033?) |
| Media Engine | VCN 3.0 |
| Display | DCN 3.01 <br> 4 Display controller |
| Memory | DDR4/LPDDR4?/LPDDR5<br>(DDR5?) |
| Power | ~29W? |


{{< ref >}}
 * [CVIP | Computer Vision and Image Processing](http://cvip.computing.dundee.ac.uk/)
 * [Combining an ISP and Vision Processor to Implement Computer Vision - Edge AI and Vision Alliance](https://www.edge-ai-vision.com/2019/01/combining-an-isp-and-vision-processor-to-implement-computer-vision/)
{{< /ref >}}

