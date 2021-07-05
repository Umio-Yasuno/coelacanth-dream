---
title: "ECC に対応する Radeon Pro W6800 と過去のパッチ"
date: 2021-07-03T15:26:42+09:00
draft: true
tags: [ "Sienna_Cichlid", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

こうした個人的なデータベースも兼ねたブログをやっていると、そう言えばあれはどうなった？という気掛かりを抱えることがままある。  

[ecc-explained-radeon-pro-W6000.pdf](https://www.amd.com/system/files/documents/ecc-explained-radeon-pro-W6000.pdf)

 > 		/* HBM  Memory Channel Width */
 > 		#define UMC_V6_1_HBM_MEMORY_CHANNEL_WIDTH	128
 > 		/* number of umc channel instance with memory map register access */
 > 		#define UMC_V6_1_CHANNEL_INSTANCE_NUM		4
 > 		/* number of umc instance with memory map register access */
 > 		#define UMC_V6_1_UMC_INSTANCE_NUM		8
 > 		/* total channel instances in one umc block */
 > 		#define UMC_V6_1_TOTAL_CHANNEL_NUM	(UMC_V6_1_CHANNEL_INSTANCE_NUM * UMC_V6_1_UMC_INSTANCE_NUM)
 > 		/* UMC regiser per channel offset */
 > 		#define UMC_V6_1_PER_CHANNEL_OFFSET_VG20	0x800
 > 		#define UMC_V6_1_PER_CHANNEL_OFFSET_ARCT	0x400
 >
 > {{< quote >}} [linux/umc_v6_1.h at 49070c4ea3d97b76c5666466efb35dcc42c6c8fd · torvalds/linux](https://github.com/torvalds/linux/blob/49070c4ea3d97b76c5666466efb35dcc42c6c8fd/drivers/gpu/drm/amd/amdgpu/umc_v6_1.h) {{< /quote >}}

[linux/umc_v6_1.h at 49070c4ea3d97b76c5666466efb35dcc42c6c8fd · torvalds/linux](https://github.com/torvalds/linux/blob/49070c4ea3d97b76c5666466efb35dcc42c6c8fd/drivers/gpu/drm/amd/amdgpu/umc_v6_1.h)  

 > 		/* HBM  Memory Channel Width */
 > 		#define UMC_V8_7_HBM_MEMORY_CHANNEL_WIDTH	128
 > 		/* number of umc channel instance with memory map register access */
 > 		#define UMC_V8_7_CHANNEL_INSTANCE_NUM		2
 > 		/* number of umc instance with memory map register access */
 > 		#define UMC_V8_7_UMC_INSTANCE_NUM		8
 > 		/* total channel instances in one umc block */
 > 		#define UMC_V8_7_TOTAL_CHANNEL_NUM	(UMC_V8_7_CHANNEL_INSTANCE_NUM * UMC_V8_7_UMC_INSTANCE_NUM)
 > 		/* UMC regiser per channel offset */
 > 		#define UMC_V8_7_PER_CHANNEL_OFFSET_SIENNA	0x400
 >
 > {{< quote >}} [linux/umc_v8_7.h at 49070c4ea3d97b76c5666466efb35dcc42c6c8fd · torvalds/linux](https://github.com/torvalds/linux/blob/49070c4ea3d97b76c5666466efb35dcc42c6c8fd/drivers/gpu/drm/amd/amdgpu/umc_v8_7.h) {{< /quote >}}

[linux/umc_v8_7.h at 49070c4ea3d97b76c5666466efb35dcc42c6c8fd · torvalds/linux](https://github.com/torvalds/linux/blob/49070c4ea3d97b76c5666466efb35dcc42c6c8fd/drivers/gpu/drm/amd/amdgpu/umc_v8_7.h)
