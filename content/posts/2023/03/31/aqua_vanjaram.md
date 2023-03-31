---
title: "MI300/GFX940/GC 9.4.3 における NPS, Aqua Vanjaram, AID と XCC の関係性"
date: 2023-03-31T17:38:02+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "CDNA_3", "gfx940", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

引き続き *MI300/GFX940/GC 9.4.3* のサポートに向けたパッチが amd-gfx メーリングリストにて公開されている。そのメモ。  

## NPS (Nodes Per Socket)
*MI300/GFX940/GC 9.4.3* では NPS (Nodes Per Socket) をサポートする。  
NPS は *AMD EPYC CPU* でもサポートされている機能であり、ソケット内で NUMA (Non-Uniform Memory Access) ノードを単一もしくは複数構成する。ノードが複数の場合、それぞれに近いメモリコントローラー、CPU ダイ (L3 キャッシュ)、PCIe ルートコンプレックスが割り当てられる。  
パッチではどのように *MI300/GFX940/GC 9.4.3* をメモリ的に分割するかは書かれていないが、CPU と GPU で共有する巨大なキャッシュがあればそことメモリコントローラーで分割されるものと思われる。  
*MI300/GFX940/GC 9.4.3* は HBM3 8チップを搭載し、NPS のモードには `NPS1/2/4/8` をサポートしている。  

*MI300/GFX940/GC 9.4.3* は CPU と GPU で実際のメモリを共有した APU ではあるが、NPS をサポートしている点で従来の APU の定義と異なる。  
AMDGPU ドライバーでは、単一ノードの場合 (`NPS1`) を従来の APU に近い `native APU mode`、複数ノードの場合 (`NPS2,NPS4,NPS8`) を `carveout mode` としている。  
また、CPU と GPU でメモリ空間を別とした `dGPU mode` もサポートされているのではないかと思われる。  

 * [[PATCH 1/9] drm/amdgpu: detect current GPU memory partition mode](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091427.html)
 * [[PATCH 8/9] drm/amdgpu: Check APU supports true APP mode](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091434.html)
 * [[PATCH 08/23] drm/amdgpu: set MTYPE in PTE for GFXIP 9.4.3](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091466.html)

 >         +	case IP_VERSION(9, 4, 3):
 >         +		/* FIXME: Needs more work for handling multiple memory
 >         +		 * partitions (> NPS1 mode) e.g. NPS4 for both APU and dGPU
 >         +		 * modes.
 >         +		 */
 >         +		snoop = true;
 >         +		if (uncached) {
 >         +			mtype = MTYPE_UC;
 >         +		} else if (adev->gmc.is_app_apu) {
 >         +			/* FIXME: APU in native mode, NPS1 single socket only
 >         +			 *
 >         +			 * For suporting NUMA partitioned APU e.g. in NPS4 mode,
 >         +			 * this need to look at the NUMA node on which the
 >         +			 * system memory allocation was done.
 >         +			 *
 >         +			 * Memory access by a different partition within same
 >         +			 * socket should be treated as remote access so MTYPE_RW
 >         +			 * cannot be used always.
 >         +			 */
 >         +			mtype = MTYPE_RW;
 >         +		} else if (adev->flags & AMD_IS_APU) {
 >         +			/* APU on carve out mode */
 >         +			mtype = MTYPE_RW;
 >         +		} else {
 >         +			/* dGPU */
 >         +			/*
 >         +			if ((mem->alloc_flags & KFD_IOC_ALLOC_MEM_FLAGS_VRAM) &&
 >         +			    bo_adev == adev)
 >         +				mapping_flags |= AMDGPU_VM_MTYPE_RW;
 >         +			else
 >         +			*/
 >         +			/* Temporarily comment out above lines and use MTYPE_NC
 >         +			 * on both VRAM and system memory access until
 >         +			 * MTYPE_RW can properly work on VRAM access
 >         +			 */
 >         +			mtype = MTYPE_NC;
 >         +		}
 >
 > {{< quote >}} [[PATCH 08/23] drm/amdgpu: set MTYPE in PTE for GFXIP 9.4.3](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091466.html) {{< /quote >}}

## Aqua Vanjaram
*MI300/GFX940/GC 9.4.3* のコードネームとして新たに *Aqua Vanjaram* がパッチから確認できる。  
AMDGPU ドライバーでは各 Hardware IP をサポートする形式に移行してから特定の GPU を指すコードネームは必要なくなったが、レジスタの初期化処理等、GPU ASIC 全体を指す場合にはコードネームが有用と判断されたのだろう。  
*RDNA 3/GFX11* 世代では初期化処理等は複数の dGPU, APU で共通しているため、コードネームではなく `SOC21` といった名前が使われている。  

 >          	case IP_VERSION(9, 4, 3):
 >         -		adev->asic_funcs = &vega20_asic_funcs;
 >         +		adev->asic_funcs = &aqua_vanjaram_asic_funcs;
 >          		adev->cg_flags =
 >
 > {{< quote >}} [[PATCH 5/5] drm/amdgpu: switch to aqua_vanjaram_doorbell_index_init](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091275.html) {{< /quote >}}

## AID と XCC の関係性
`AID` の略称は不明なままだが、*MI300/GFX940/GC 9.4.3* でサポートする最大 `AID` 数や `AID` が内蔵するユニットはパッチから読み取れる  

`AID` は通常 SDMA (System DMA, XGMI) エンジンを 4基、マルチメディアエンジン、VCN (Video Core Next) と JPEG 用を 1基ずつ持つ。  

 >         +	/* generally 1 AID supports 4 instances */
 >         +	adev->sdma.num_inst_per_aid = 4;
 >         +	adev->sdma.num_instances = NUM_SDMA(adev->sdma.sdma_mask);
 >         +
 >         +	adev->vcn.num_inst_per_aid = 1;
 >         +	adev->vcn.num_vcn_inst = adev->vcn.num_inst_per_aid * adev->num_aid;
 >         +	adev->jpeg.num_inst_per_aid = 1;
 >         +	adev->jpeg.num_jpeg_inst = adev->jpeg.num_inst_per_aid * adev->num_aid;
 >         +
 >
 > {{< quote >}} [[PATCH 04/23] drm/amdgpu: Move generic logic to soc config](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091450.html) {{< /quote >}}

そして `XCC (XCD)` 2基で `AID` 1基を共有し、`AID` 自体は最大 4基とされている。  
恐らく `AID` はマルチメディアやチップレット間接続、I/O 周りの機能と `XCC (XCD)` 2基のベースダイとしての役割を持っているのではないかと思われる。  

 >         +	unsigned int reg;
 >         +	/* Every two XCCs share one AID */
 >         +	unsigned int aid = xcc_inst / 2;
 >
 > {{< quote >}} [[PATCH 09/14] drm/amdgpu: Fix SWS on multi-XCD GPU](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091265.html) {{< /quote >}}

 * [[PATCH 24/32] drm/amdgpu: assign the doorbell index for sdma on non-AID0](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091126.html)
 * [[PATCH 27/32] drm/amdgpu: complement the IH node_id table for multiple AIDs](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091128.html)
 * [[PATCH 09/14] drm/amdgpu: Fix SWS on multi-XCD GPU](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091265.html)
 * [[PATCH 04/23] drm/amdgpu: Move generic logic to soc config](https://lists.freedesktop.org/archives/amd-gfx/2023-March/091450.html)

*MI300/GFX940/GC 9.4.3* に搭載される *VCN 4.0.3* の対応コーデックについては RadeonSI へのパッチで触れられているが、*RDNA 3/GFX11* 世代に搭載されている *VCN 4.0.x* と基本同じとされている。  

{{< ref >}}
 * [株式会社HPCテック | 開発コード名：Genoa](https://www.hpctech.co.jp/hardware/amd-genoa.html)
 * [最大25%性能が向上するAMD EPYC(Milan)を搭載したPowerEdgeサーバーが登場 | HPCwire Japan](https://www.hpcwire.jp/archives/42846)
{{< /ref >}}
