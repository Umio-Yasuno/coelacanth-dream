---
title: "AMDGPU Database: Device ID/ Revision ID/ Product Name"
date: 2019-12-30T10:59:25+09:00
draft: false
tags: [ "Radeon", "Database" ]
keywords: [ "Radeon", "AMDGPU" , "Device ID", "Revision ID", "Product Name" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

## Table of Content

 * [APU](#apu)
	* [Raven](#raven-gfx902)
	* [Picasso](#picasso-gfx902)
	* [Raven2](#raven2-gfx909)
		* [Dali](#dali-gfx909)
		* [Pollock](#pollock-gfx909)
	* [Renoir](#renoir-gfx909)
 * [Discrete GPU](#dgpu)
	* [Vega10](#vega10-gfx900)
	* [Vega12](#vega12-gfx904)
	* [Vega20](#vega20-gfx906)
	* [Arcturus](#arcturus-gfx908)
	* [Navi10](#navi10-gfx1010)
	* [Navi14](#navi14-gfx1012)
	* [Navi12](#navi12-gfx1011)

[参考リンク](#ref-title)

## APU {#apu}
<!-- FAMILY_RV -->
### Raven ( gfx902 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15DD &darr; | 81 | V1807B | (45W FP5 Vega 11)
|  | 82 | V1756B | (45W FP5 Vega 8)
| | 83 | V1605B | (15W FP5 Vega 8)
| | 84 | iTemp EMBEDDED | (15W FP5 Vega 6)
| | 85 | V1202B | (15W FP5 Vega 3)
| | 86 |  | (Embedded 65W AM4)
| | 87 |  | (Embedded 65W AM4)
| | 88 |  | (Embedded 65W AM4 Vega 8)
|  | C1 | 2800H? | (45W FP5 Vega 11) |
| | C2 | 2600H? | (45W FP5 Vega 8) |
| | C3 | 2700U | (15W FP5 Vega 10) |
| | C4 | 2500U | (15W FP5 Vega 8) |
| | C5 | 2200U | (15W FP5 Vega 3) |
| | C6 | 2400G | (65W AM4 Vega 11) |
| | C7 | | (B8 65W AM4) |
| | C8 | 2200G | (65W AM4 Vega 8) |
| | C9 | 2400GE? | (35W AM4 Vega 11) |
| | CA | 2200GE? | (35W AM4 Vega 8) |
| | CB | 200GE /220GE /240GE | (35W AM4 Vega 3) |
| | CC | 2300U | (15W FP5 Vega 6) |
| | CE | 2200U? | (15W FP5 Vega 3) |
| | D0 | PRO 2700U? | (15W FP5 Vega 10) |
| | D1 | PRO 2500U | (15W FP5 Vega 8) |
| | D2 | | (B4 15W FP5) |
| | D3 | PRO 2400G? | (65W AM4 Vega 11) |
| | D4 | | (B8 65W AM4) |
| | D5 | PRO 2200G? | (65W AM4 Vega 8) |
| | D6 | PRO 2400GE? | (35W AM4 Vega 11) |
| | D7 | PRO 2200GE? | (35W AM4 Vega 8) |
| | D8 | | (35W AM4 Vega 3) |
| | D9 | | (15W FP5 Vega 6) |
| | ? | V1404I | (15W FP5 Vega 8) |
[Page Top](#table-of-content)

### Picasso ( gfx902 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15D8 &darr; | 00 | | Winston (Vega 8 WS) |
| | 93 | | (Vega 1 Raven2? /Dali? /Pollock?) |
| | A1 | | (Vega 10) |
| | A2 | | (Vega 8) |
| | A3 | | (Vega 6) |
| | A4 | | (Vega 3) |
| | B1 | | (Vega 10) |
| | B2 | | (Vega 8) |
| | B3 | | (Vega 6) |
| | B4 | | (Vega 3) |
| | C1 | 3700C/U[^7] /3750H | (Vega 10) |
| | C2 | 3500C/U[^7] /3550H | (Vega 8) |
| | C3 | | (Vega 6) |
| | C8 | 3400G | (AM4 Vega 11) |
| | C9 | 3200G | (AM4 Vega 8) |
| | CA | | (AM4 Vega 11) |
| | CB | | (AM4 Vega 8) |
| | CD | | (AM4 Vega 2) |
| | D1 | PRO 3700U | (Vega 10) |
| | D2 | PRO 3500U | (Vega 8) |
| | D3 | (PRO) 3300U | (Vega 6) |
| | D4 | | (Vega 3) |
| | D8 | | (AM4 Vega 11) |
| | D9 | | (AM4 Vega 8) |
| | DA | | (AM4 Vega 11) |
| | DB | | (AM4 Vega 8) |
| | DC | | (AM4 Vega 3) |
| | E1 | 3780U | Winston (Surface Edition Vega 11) |
| | E2 | 3580U | Winston (Surface Edition Vega 8) |
| | ? | 300UGE | (65W AM4 Vega 3) |
[Page Top](#table-of-content)

### Raven2 ( gfx909 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15DD &darr; | E1 | | (15W FP5 Vega 3) |
| | E2 | | (35W AM4 Vega 3) |
| 15D8 &darr; | 91[^10] | R1605G | Embedded |
| | 92 | R1505G? | Embedded |
| | C4 | 3200U | (FP5 15W Vega 3), == 3250C? |
| | C5 | 300U | (Vega 3) |
| | CC | 3000G | (AM4 Vega 3) |
[Page Top](#table-of-content)

[^10]: [Core i7並みのRyzen搭載で、4万円台＆片手サイズ⁉ 〝Ryzen Embedded”搭載の超小型PCベアボーン「4×4 BOX」が超お得 (3/3) | AMD HEROES](https://amd-heroes.jp/article/2020/03/0364/3/)

#### Dali ( gfx909 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15D8 &darr; | C4 | 3250C[^7] | (Vega 3) |
| | CD | 3050C[^7] | (Vega 2) |
| | CE | 3150C[^7] | (Vega 3) |
| | CF | R1305G | (6W[^8]) |
| | DE[^9] | | |
| | DF[^9] | | |
| | E3 | | (6W[^8] Vega 3) |
| | E4 | R1102G[^11] | (6W[^8] Vega 3) |
[Page Top](#table-of-content)

[^7]: [2040455: Rework map_oprom_vendev to add revision check and mapping —   Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/3/src/soc/amd/picasso/northbridge.c#332)
[^8]: [[Dali] Raven 2 detection Patch](https://lists.freedesktop.org/archives/amd-gfx/2020-February/045579.html) <br> <https://lists.freedesktop.org/archives/amd-gfx/attachments/20200205/f22ebc46/attachment.obj>
[^9]: [[PATCH 28/35] drm/amd/display: Fix RV2 Variant Detection](https://lists.freedesktop.org/archives/amd-gfx/2020-February/046322.html)
[^11]: amdgpu.ids [R-Series R1000 with Radeon™ Vega Graphics Drivers & Support | AMD](https://www.amd.com/en/support/embedded/amd-ryzen-embedded-r-series-processors/r-series-r1000-radeon-vega-graphics)

### Pollock ( gfx909 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15D8 &darr; | 94 | | |
| | 95 | | |
| | E9 | | (Samples FT5)[^7] |
| | EA | | (Production FT5)[^7] |
| | EB | | |
[Page Top](#table-of-content)

 > Source:<cite>
[drm/amd/display: add Pollock IDs, fix Pollock & Dali clk mgr construct - amd-staging-drm-next](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/display/include/dal_asic_id.h?h=amd-staging-drm-next&id=04d8707ef5549065c10d5a2e20d2c61089807997)</cite>

<!--
| 15D9 &darr; | 91 | | **?** |
| | 92 | | |
| | C1 | | |
| | C2 | | |
| | C3 | | |
-->

<!--
### FireFlight
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15FF | 00 | Fenghuang (Zhongshan Subor Z+) | Radeon Vega 28 Mobile |
-->

## Renoir ( gfx909 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 1636 &darr; | 00 | | (BringUp FP6) |
| | C1 | 4800U | (B12 15W FP6) |
| | C2 | | (B10 15W FP6) |
| | C3 | 4500U | (B8 15W FP6) |
| | C4 | | (B6 15W FP6) |
| | C5 | 4900HS | (B12 45W FP6) |
| | C6 | 4800HS | (B10 45W FP6) |
| | C7 | | (B8 45W FP6) |
| | C8 | | (B10 65W AM4) |
| | C9 | | (B8 65W AM4) |
| | CA | | (B6 65W AM4) |
| | CB | | (B10 35W AM4) |
| | CC | | (B8 35W AM4) |
| | CD | | (B6 35W AM4) |
| | CE | | (B4 35W AM4) |
| | D1 | | (B12B 15W FP6) |
| | D2 | | (B10B 15W FP6) |
| | D3 | | (B8B 15W FP6) |
| | D4 | | (B6B 15W FP6) |
| | D5 | | (B12B 45W FP6) |
| | D6 | | (B10B 45W FP6) |
| | D7 | | (B8B 45W AM4) |
| | D8 | | (B10B 65W AM4) |
| | D9 | | (B8B 65W AM4) |
| | DA | | (B6B 65W AM4) |
| | DB | | (B10B 35W AM4) |
| | DC | | (B8B 35W AM4) |
| | DD | | (B6B 35W AM4) |
| | DE | | (B4B 35W AM4) |
| | E1 | | <!-- Acton --> |
| | E2 | | <!-- Acton --> |
| | E3 | | <!-- Acton --> |

<!--
	VANGOGH 163F:00
	gfx1032 / gfx1033 [^13]

[^13]: [P4 to Git Change 2008166 by vsytchen@vsytchen-remote-ocl-win10 on 201… · ROCm-Developer-Tools/ROCclr@df2e0b9](https://github.com/ROCm-Developer-Tools/ROCclr/commit/df2e0b9ae27f43ba8e23a3afa185c16dd3bb5ebc)
-->

[Page Top](#table-of-content)

## Discrete GPU {#dgpu}
<!-- FAMILY_AI -->
### Vega10 ( gfx900 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 6860 &darr; | 00 | Radeon Instinct MI25 &darr; | (Vega10 GLXT SERVER) &darr; |
| | 01 | | |
| | 02 | | |
| | 03 | Radeon Pro V340 | |
| | 04 | Radeon Instinct MI25x2 | |
| | 07 | Radeon Pro V320 (/V340?) [^6]| |
| | C0 | Radeon Pro Vega 48 /56 /64 /64X[^1] | ? |
| 6861 | 00 | Radeon Pro (TM) WX 9100 | (Vega10 GLXT) |
| 6862 | 00 | Radeon Pro SSG | (Vega10 SSG) |
| 6863 | 00 | Radeon Vega Frontier Edition | (Vega10 GLXTX) |
| 6864 &darr; | 00 | | |
| | 03 | Radeon Pro V340 | (Vega10 GLXT SERVER) &darr; |
| | 04 | Instinct MI25x2 | |
| | 05 | Radeon Pro V340 | |
| 6867 | 00 | Radeon Pro Vega 56 | (Vega10 XLA) |
| 6868 | 00 | Radeon (TM) Pro WX 8200 | (Vega10 GLXL) |
| 6869 | 00 | | (Vega10 XGA) |
| 686A | 00 | | (Vega10 LEA) |
| 686B | 00 | Radeon Pro Vega 64X | (Vega10 XTXA) |
| 686C &darr; | 00 | Radeon Instinct MI25 MxGPU &darr; | (Vega10 GLXT SERVER VF) &darr; |
| | 01 | | |
| | 02 | | |
| | 03 | Radeon Pro V340 MxGPU | |
| | 04 | Radeon Instinct MI25x2 MxGPU | |
| | 05 | Radeon Pro V340 MxGPU | |
| | C1 | | |
| 686D | 00 | | (Vega10 GLXTA) |
| 686E | 00 | | (Vega10 GLXLA) |
| 687F &darr; | 01 | Radeon RX Vega | (Vega10 XT - Blade) |
| | C0 | Radeon RX Vega | (Vega10 XTX) |
| | C1 | Radeon RX Vega (64) | (Vega10 DESKTOP XT) |
| | C3 | Radeon RX Vega (56) | (Vega10 XL) |
| | C4 | | (Vega10 XL) |
| | C7 | Radeon RX Vega | (Vega10 XL) |
[Page Top](#table-of-content)

[^1]: SubSystem ID? ( Vega 48:0x0196, Vega 56:0x017B, Vega 64:0x017C Vega 64X:0x0188 )
[^6]: [Compute Performance of AMD Radeon Pro V320 - CompuBench](https://compubench.com/device.jsp?benchmark=compu20d&os=Windows&api=cl&D=AMD+Radeon+Pro+V320&testgroup=info)

### Vega12 ( gfx904 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 69A0 | 00 | | (Vega12 GL MXT) |
| 69A1 | 00 | | (Vega12 GL MXL) |
| 69A2 | 00 | | (Vega12 GL XL) |
| 69A3 | 00 | | (Vega12 Reserved) |
| 69AF &darr; | C0 | Radeon Pro Vega 20 | (Vega12 XTA) |
| | C1 | | (Vega12 MXT) |
| | C3 | | (Vega12 MLP) |
| | C7 | Radeon Pro Vega 16 | (Vega12 XLA) |
| | CF | | (Vega12 XL) |
| | FF | | (Vega12 LE) |
[Page Top](#table-of-content)

### Vega20 ( gfx906 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 66A0 | 00 | Radeon Instinct | |
| 66A1 &darr; | 00 | | (Server XT 32GB) |
| | 02 | | |
| | 03 | | |
| | 06 | Radeon Pro VII | (Vega20 WKS GL-XE) |
| 66A2 &darr; | 00 | | |
| | 02 | | |
| 66A3 | 00 | Radeon Pro Vega II (Duo) | |
| 66A4 | 00 | | |
| 66A7 | 00 | | |
| 66AF &darr; | C0 | | |
| | C1 | Radeon VII | (Vega20 XT) |
| | C3 | | |
| | C7 | | |
| | CF | | |
[Page Top](#table-of-content)

### Arcturus ( gfx908 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 738C | | MI100[^5] | |
| 7388 | | | |
| 738E | | | |
| 7390 | | | (Arcturus VF) |
[Page Top](#table-of-content)

<!-- FAMILY_NV -->
<!--
Navi LITE 13E9:00
-->

### Navi10 ( gfx1010 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 66AF &darr; | 70 | | (Fake DID?) |
| | 71 | | (Fake DID?) |
| | 72 | | (Fake DID?) |
| | 73 | | (Fake DID?) |
| | F0 | | (Navi10 XTX Fake DID?[^2]) |
| | F1 | | (Navi10 XT Fake DID?[^2]) |
| | F2 | | (Navi10 XL Fake DID?[^2]) |
| | F3 | | (12Gbps, Fake DID?) |
| | F4 | | (12Gbps, Fake DID?) |
| | F5 | | (12Gbps, Fake DID?) |
| | F6 | | (Navi10 XME Fake DID?[^2]) |
| 7310 | 00 | Radeon Pro W5700X |  |
| 7312 | 00 | Radeon Pro W5700 | (Navi10 Pro XL?[^4]) |
| 7318 | 40 | | |
| 7319 | 40 | Radeon Pro 5700 XT | |
| 731A | 40 | | |
| 731B | 40 | Radeon Pro 5700 | |
| 731F &darr; | C0 | RX 5700 XT 50th | (Navi10 XTX) |
| | C1 | RX 5700 XT | (Navi10 XT) |
| | C2 | RX 5600M | (Navi10 XME) |
| | C3 | RX 5700M | |
| | C4 | RX 5700 | (Navi10 XL) |
| | C5 | RX 5700 XT | |
| | C7 | | |
| | CA | RX 5600 XT | (Navi10 XLE) |
| | CB | RX 5600 | (Navi10 XE) |
| | E1 | | |
| | E3 | | |
| | E7 | | |
[Page Top](#table-of-content)

[^2]: [navi10_ppt.c#n1593](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/navi10_ppt.c?h=amd-staging-drm-next#n1593)
[^4]:[AMD Radeon Pro W5700 8 GB BIOS - TechPowerUp](https://www.techpowerup.com/vgabios/216287/216287)

### Navi14 ( gfx1012 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 67DF &darr; | 3C | | (Navi14 Pro-? Fake DID?) |
|  | 3D | | (Navi14 Pro-? Fake DID?) |
|  | 3E | | (Navi14 Pro-? Fake DID?) |
|  | 3F | | (Navi14 Pro-XLM Fake DID?) |
|  | F2 | | (Navi14 XTM Fake DID?[^2]) |
|  | F3 | | (Navi14 XLM Fake DID?[^2]) |
|  | F4 | | (Navi14 XT Fake DID?[^2]) |
|  | F5 | | (Navi14 XL Fake DID??) |
|  | F6 | | (Navi14 XTX Fake DID?[^2]) |
| 7340 &darr; | 00 | | <!-- Falcon GL-XTA --> |
| | 40 | Radeon Pro 5500M | (Navi14 ULA[^3]) |
| | 41 | | <!-- Falcon XTA 8G --> |
| | 43 | Radeon Pro 5300M | |
| | 47 | | <!-- Falcon XLXA --> | 
| | 70 | | |
| | C1 | RX 5500M | (Navi14 XTM) |
| | C3 | RX 5300M | (Navi14 XLM) |
| | C5 | RX 5500 XT | (Navi14 XTX) |
| | C7 | RX 5500 | (Navi14 XT) |
| | CF | RX 5300? | (Navi14 XL?) |
| | F2 | | |
| | F3 | | |
| 7341 | 00 | Radeon Pro W5500 | (Navi14 Pro-XL) |
| 7343 | 00 | | |
| 7347 | 00 | Radeon Pro W5500M | (Navi14 Pro-XTM) |
| 734F | 00 | Radeon Pro W5300M | (Navi14 Pro-XLM) |
[Page Top](#table-of-content)

<!--
SubSystem ID? (Pro 5500M:0x020F, Pro 5300M:0x0210)
-->

[^3]:[Apple Pro 5500M 8 GB BIOS - TechPowerUp](https://www.techpowerup.com/vgabios/216534/apple-pro5500m-8192-191010)

### Navi12 ( gfx1011 )
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 69B0 &darr; | 71 | | |
| | F1 | | |
| | F3 | | |
| 7360 &darr; | 40 | | |
| | 41 | | |
| | C1 | | |
| | C3 | | |
| 7362 | | | (Navi12 VF) |
[Page Top](#table-of-content)

<!--
	Navi21: gfx1030 [^12]

    [^12]: [P4 to Git Change 1759681 by asalmanp@asalmanp-ocl-stg on 2019/03/21 1… · ROCm-Developer-Tools/ROCclr@42c005d](https://github.com/ROCm-Developer-Tools/ROCclr/commit/42c005d656a8bd99f87fa4fc86ad086701c59a86)

    Navi22: gfx1031?
    Navi23: gfx1032?

    MI200: gfx909??? maybe temp id

[^14]: [P4 to Git Change 2042212 by kjayapra@1_HIPWS_LNX1_PAL on 2019/12/06 1… · ROCm-Developer-Tools/ROCclr@a35c1d2](https://github.com/ROCm-Developer-Tools/ROCclr/commit/a35c1d2f2d954363c7d2121d5334f9e7766beeae)
-->


[^5]:<https://github.com/ROCm-Developer-Tools/aomp-extras/blob/f3d316fe64347e697a9789f0f2499fec50024db1/utils/bin/gputable.txt#L1867>

{{< ref >}}

 * [DeviceInfoUtils.cpp - GPUOpen-Tools/common-src-DeviceInfo](https://github.com/GPUOpen-Tools/common-src-DeviceInfo/blob/master/DeviceInfoUtils.cpp)

{{< /ref >}}
