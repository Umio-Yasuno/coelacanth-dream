---
title: "【シーラカンスノート】 RadeonSI が VRS に対応、他 RDNA 2 dGPU の DeviceID 【2020/11/18】"
date: 2020-11-18T15:47:28+09:00
draft: false
tags: [ "RadeonSI", "RDNA_2", "Navy_Flounder", "Dimgrey_Cavefish" ]
# keywords: [ "", ]
categories: [ "Note", "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

## ソフトウェア側も AV1 HWデコードに対応 {#av1}

*RDNA 2 / GFX10.3* 世代の GPU/APU、厳密に言えばそれらがマルチメディアエンジンとして搭載している VCN 3.0 が対応する AV1 HWデコードをサポートするパッチが Mesa3D に投稿、メインラインに取り込まれている。  
{{< link >}} [Radeon: AV1 HW video acceleration with OMX (!7596) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7596) {{< /link >}}

これで *RDNA 2 / GFX10.3* 世代の GPU/APU を持つユーザーが AV1 HWデコードを活用できるようになる。  
AV1コーディックは主に Youtube で使われている。HWデコードにはブラウザ側も対応する必要があるが、Firefox、Chromium系は (確か) VA-API を使っているはずなので、そこで引っ掛かることはないと思われる (たぶん)。  

## RadeonSI が VRS に対応 {#vrs}

*RDNA 2 / GFX10.3* 世代の GPU から対応している VRS (Variable Rate Shader) 機能を有効にするパッチが Mesa3D に投稿され、こちらも既にメインラインに組み込まれている。  
{{< link >}} [radeonsi: add an option to enable 2x2 coarse shading for non-GUI elements (c3432ad8) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/c3432ad852449ec31580a0b77af785e37eaa48f9) {{< /link >}}

現時点ではまだ実験的なサポートであり、driconf かアプリケーション起動時に設定できるオプションで有効する形となっている。  
driconf はユーザー側から選択できる最適化オプションの 1つと言えるのだが、気が向いたら記事の形にまとめるかもしれない。  

**RadeonSI (OpenGL)** ドライバー側が対応したため、既存の OpenGL を用いるアプリケーション、ベンチマークでも VRS の有効無効で性能に影響が出そうだが、その場合はどういった風に GPU の性能を評価することになるのか。  
VRS には Intel *Gen11アーキテクチャ* からハードウェア側は対応しているはずなのだが、Mesa3D にそれを有効にするコード、オプションは追加されていない。  


## Navy Flounder、Dimgrey Cavefish の DeviceID 追加 {#did}

*Sienna Cichlid* 以外の *RDNA 2 / GFX10.3* dGPU、*Navy Flounder* 、*Dimgrey Cavefish* の DeviceID を追加するパッチが Linux Kernel (amd-gfx) に投稿されている。  

 >       +	/* Navy_Flounder */
 >       +	{0x1002, 0x73C0, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_NAVY_FLOUNDER},
 >       +	{0x1002, 0x73C1, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_NAVY_FLOUNDER},
 >       +	{0x1002, 0x73C3, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_NAVY_FLOUNDER},
 >       +	{0x1002, 0x73DF, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_NAVY_FLOUNDER},
 >       +
 >
 > {{< quote >}} [[PATCH 1/2] drm/amdgpu: add device ID for navy_flounder (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-November/056293.html) {{< /quote >}}
 >
 >       
 >       +	/* DIMGREY_CAVEFISH */
 >       +	{0x1002, 0x73E0, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_DIMGREY_CAVEFISH},
 >       +	{0x1002, 0x73E1, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_DIMGREY_CAVEFISH},
 >       +	{0x1002, 0x73E2, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_DIMGREY_CAVEFISH},
 >       +	{0x1002, 0x73FF, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_DIMGREY_CAVEFISH},
 >       +
 >
 > {{< quote >}} [[PATCH 2/2] drm/amdgpu: add DID for dimgrey_cavefish](https://lists.freedesktop.org/archives/amd-gfx/2020-November/056294.html) {{< /quote >}}

とは言っても DeviceID だけで、RevisionID とか SKU名は全くの不明。  
AMD が以下のコード等に追加してくれる日を待つしかない。  

 * [common_src_device_info/DeviceInfoUtils.cpp at master · GPUOpen-Tools/common_src_device_info](https://github.com/GPUOpen-Tools/common_src_device_info/blob/master/DeviceInfoUtils.cpp)
 * [data/amdgpu.ids · master · Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm/blob/master/data/amdgpu.ids)
