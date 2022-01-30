---
title: "既に 2つの DeviceID が用意されている Yellow Carp APU ファミリー"
date: 2021-07-26T11:39:41+09:00
draft: false
tags: [ "Yellow_Carp", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

*Yellow Carp APU* の Device (PCI) ID は、 AMD GPUドライバーへの最初のパッチでは追加されなかったが、先日 amd-gfxメーリングリストにて DeviceID を追加するパッチが公開された。  
追加された *Yellow Carp APU* の DeviceID は `0x164D` と `0x1681` の 2つとなる。  

 * [[PATCH 1/2] drm/amdgpu: update yellow carp external rev_id handling](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066870.html)
 * [[PATCH 2/2] drm/amdgpu: add yellow carp pci id (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066869.html)

 > 		+	/* Yellow Carp */
 > 		+	{0x1002, 0x164D, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_YELLOW_CARP|AMD_IS_APU},
 > 		+	{0x1002, 0x1681, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_YELLOW_CARP|AMD_IS_APU},
 > 		+
 >
 > {{< quote >}} [[PATCH 2/2] drm/amdgpu: add yellow carp pci id (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066869.html) {{< /quote >}}

パッチでは、2つでそれぞれ異なる `external_rev_id` を持つことが示されている。  
`external_rev_id` は APU/GPU のシリコンに割り当てられる ID であり、 8-bit の範囲 (0x00-0xFF) から一部を取る。また、`external_rev_id` は APU/GPU のファミリー内で識別するための ID となる。  

 > 		-		adev->external_rev_id = adev->rev_id + 0x01;
 > 		+		if (adev->pdev->device == 0x1681)
 > 		+			adev->external_rev_id = adev->rev_id + 0x19;
 > 		+		else
 > 		+			adev->external_rev_id = adev->rev_id + 0x01;
 >
 > {{< quote >}} [[PATCH 1/2] drm/amdgpu: update yellow carp external rev_id handling](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066870.html) {{< /quote >}}

異なる `external_rev_id` から、今回のパッチは *Yellow Carp* と同様の GPUコア部、各種 IP (マルチメディアエンジン、ディスプレイエンジン/コントローラー) を搭載した APU が存在する可能性を示している。  
*Yellow Carp* の DeviceID として追加されているが、これは APU/GPU の DeviceID が記述されている `amdgpu_drv.c` では GPU部、各種 IP が基本同じとなる APU をまとめて管理しており、その関係と思われる。  
*Raven* には *Raven, Raven2, Picasso* 、*Renoir* には *Renoir, Lucienne, Cezanne (Green Sardine), Barcelo* がそれぞれ属する。  

 > 			/* Renoir */
 > 			{0x1002, 0x15E7, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 > 			{0x1002, 0x1636, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 > 			{0x1002, 0x1638, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 > 			{0x1002, 0x164C, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 >
 > {{< quote >}} [linux/amdgpu_drv.c at 27f5355f5d9706dfc1c2542253689f421008c969 · torvalds/linux](https://github.com/torvalds/linux/blob/27f5355f5d9706dfc1c2542253689f421008c969/drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c#L1170-L1174) {{< /quote >}}

ディスプレイエンジン/コントローラー部のバージョンを識別するのに使われる部分を記述した `dal_asci_id.h` は、*Yellow Carp* に A0ステッピングと B0ステッピングが存在し、それぞれ異なる ID を持つとしているが、  
今回のパッチで追加された `DeviceId: 0x1681` の `external_rev_id` 範囲から外れるため、やはり *Yellow Carp* と同様の GPUコア部を搭載した異なる APU が存在し、既にサポートを追加する用意が進められていると考えられる。  
コードネームも *Yellow Carp* とは異なるものになるだろう。  

 > 		#define FAMILY_YELLOW_CARP                     146
 > 		
 > 		#define YELLOW_CARP_A0 0x01
 > 		#define YELLOW_CARP_B0 0x02		// TODO: DCN31 - update with correct B0 ID
 > 		#define YELLOW_CARP_UNKNOWN 0xFF
 > 		
 > 		#ifndef ASICREV_IS_YELLOW_CARP
 > 		#define ASICREV_IS_YELLOW_CARP(eChipRev) ((eChipRev >= YELLOW_CARP_A0) && (eChipRev < YELLOW_CARP_UNKNOWN))
 > 		#endif
 >
 > {{< quote >}} [linux/dal_asic_id.h at master · torvalds/linux](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/display/include/dal_asic_id.h#L226-L235) {{< /quote >}}

最近の AMD APU では、*Raven* からプロセスを改良した *Picasso* 、*Raven* に存在したバグを修正し、規模を小さくして同じプロセスで製造される *Raven2* 、  
*Renoir* と同じ構成、同じ製造プロセスだが電源管理機能を改良した *Lucienne* 、*Renoir* から CPUアーキテクチャを一世代進んだものを採用した *Cezanne (Green Sardine)* など、ある APU からの派生パターンが増えていることもあり、`DeviceID: 0x1681` が *Yellow Carp* とどこが異なるのかは現時点では全く予想できないが、だからこそパッチ、公式発表を待つことは楽しいのだと思う。  

{{< ref >}}
 * [src/amd/addrlib/src/amdgpu_asic_addr.h · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/amd/addrlib/src/amdgpu_asic_addr.h)
{{< /ref >}}
