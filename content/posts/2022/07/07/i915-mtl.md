---
title: "Intel GFX ドライバーに Meteor Lake をサポートする最初のパッチが投稿される ―― Gen12.7, Xe_LPD+, Xe_LPM+"
date: 2022-07-07T11:57:16+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Linux_Kernel", "Meteor_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の [Radhakrishna Sripada](https://www.linkedin.com/in/rkinvictus) 氏より、Linux Kernel における Intel GPU/GFX ドライバー `i915` に *Meteor Lake-M/P* のサポートを追加する最初のパッチが投稿されている。  

 * [[Intel-gfx] [PATCH 0/2] i915: Introduce Meteorlake](https://lists.freedesktop.org/archives/intel-gfx/2022-July/301011.html)
 * [[Intel-gfx] [PATCH 1/2] drm/i915/mtl: Add MeteorLake platform info](https://lists.freedesktop.org/archives/intel-gfx/2022-July/301009.html)
 * [[Intel-gfx] [PATCH 2/2] drm/i915/mtl: Add MeteorLake PCI IDs](https://lists.freedesktop.org/archives/intel-gfx/2022-July/301010.html)

*Meteor Lake-M/P* の各 IP バージョンは、GraphicsIP は `ver12.70`、DisplayIP は `ver14 (Xe_LPD+)`、MediaIP は `ver13 (Xe_LPM+)` とされている。  
ただ GraphicsIP については、あくまでもドライバー内で区別するためのバージョンであり、実際のバージョンはハードウェアの `GMD_ID` レジスタから読み取れる。  
また、[intel/gmmlib](https://github.com/intel/gmmlib) などでは、`PRODUCT_FAMILY` や `GFXCORE_FAMILY` としては `IGFX_DG2 = 1270` (ver12.7) を割り当てているが、Intel GFX ドライバーでは *DG2/Alchemist* の GraphicsIP は ver12.55 とされており、それぞれバージョン番号は近いが意味する所は異なっている。[^gmmlib]  

[^gmmlib]: [gmmlib/igfxfmid.h at c70167bee652c7f4cc656d651a1705ffb6bcc0c9 · intel/gmmlib](https://github.com/intel/gmmlib/blob/c70167bee652c7f4cc656d651a1705ffb6bcc0c9/Source/inc/common/igfxfmid.h#L69-L82)

*Xe_LPD+* の新機能に関するパッチはまだ投稿されていない。*Xe_LPD+* では *Xe_LPD (ver13)* でサポートする機能に加えて `has_cdclk_crawl` のフラグが有効化されているが、これは *Alder Lake-P* でもサポートされている。  
同様に *Xe_LPD (ver13)* IP を採用する *DG2/Alchemist* では `has_cdclk_crawl` が有効化されておらず、*Xe_LPD (ver13)* においては分離していた部分だった。[^cdclk_crawl]  

[^cdclk_crawl]: <https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/i915/i915_pci.c?h=next-20220706#n983>

`platform_engine_mask` に `CCS0` のビットがセットされていることから、*Meteor Lake-M/P* では *Compute Engine (Compute Command Streamer)* を 1基搭載、有効化される。  
*Xe_HP SDV, DG2/Alchemist, Ponte Vecchio* では 4基搭載されているため、それよりも小さい規模となる。  
公開されているドキュメントでは *Tiger Lake* でも CCS 1基を搭載しているとされているが、i915 ドライバーでは何らかの理由で有効化されていないものと思われる。  
*Rocket Lake, Alder Lake-N/M/P/S* でも同様のため、iGPU では *Meteor Lake-M/P* で CCS がようやく有効化されることとなる。  
{{< link >}} [Intel OSS OpenGL/Vulkan ドライバーが "Compute Engine" に対応 | Coelacanth's Dream](/posts/2022/06/16/intel-ccs-mesa3d/) {{< /link >}}

 > 		+#define XE_LPDP_FEATURES	\
 > 		+	XE_LPD_FEATURES,	\
 > 		+	.display.ver = 14,	\
 > 		+	.display.has_cdclk_crawl = 1
 > 		+
 > 		+__maybe_unused
 > 		+static const struct intel_device_info mtl_info = {
 > 		+	XE_HP_FEATURES,
 > 		+	XE_LPDP_FEATURES,
 > 		+	/*
 > 		+	 * Real graphics IP version will be obtained from hardware GMD_ID
 > 		+	 * register.  Value provided here is just for sanity checking.
 > 		+	 */
 > 		+	.graphics.ver = 12,
 > 		+	.graphics.rel = 70,
 > 		+	.media.ver = 13,
 > 		+	.memory_regions = REGION_SMEM | REGION_STOLEN_LMEM,
 > 		+	PLATFORM(INTEL_METEORLAKE),
 > 		+	.platform_engine_mask = BIT(RCS0) | BIT(BCS0) | BIT(CCS0),
 > 		+	.require_force_probe = 1,
 > 		+	.has_flat_ccs = 0,
 > 		+	.has_snoop = 1,
 > 		+	.display.has_modular_fia = 1,
 > 		+};
 > 		+
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 1/2] drm/i915/mtl: Add MeteorLake platform info](https://lists.freedesktop.org/archives/intel-gfx/2022-July/301009.html) {{< /quote >}}

*Alder Lake-M/P* では GPU部は IPバージョン、規模ともに同じであったため、ドライバーではまとめて管理していたが、*Meteor Lake-M/P* では DeviceID、`SUBPLATFORM` で分けて管理されるようになる。  
`SUBPLATFORM` はプラットフォームの判定やハードウェア側のバグに対する回避策 (ワークアラウンド) の設定の際にも使われている。  
*Meteor Lake* ではタイルアーキテクチャを採用すると発表されているが、*-M* と *-P* でアーキテクチャは同じだが、パッケージの違いから規模が異なる GPUタイルを採用することが考えられる。  

 > 		+/* MTL */
 > 		+#define INTEL_MTL_M_IDS(info) \
 > 		+	INTEL_VGA_DEVICE(0x7D40, info), \
 > 		+	INTEL_VGA_DEVICE(0x7D43, info), \
 > 		+	INTEL_VGA_DEVICE(0x7DC0, info)
 > 		+
 > 		+#define INTEL_MTL_P_IDS(info) \
 > 		+	INTEL_VGA_DEVICE(0x7D45, info), \
 > 		+	INTEL_VGA_DEVICE(0x7D47, info), \
 > 		+	INTEL_VGA_DEVICE(0x7D55, info), \
 > 		+	INTEL_VGA_DEVICE(0x7D60, info), \
 > 		+	INTEL_VGA_DEVICE(0x7DC5, info), \
 > 		+	INTEL_VGA_DEVICE(0x7DD5, info), \
 > 		+	INTEL_VGA_DEVICE(0x7DE0, info)
 > 		+
 > 		+#define INTEL_MTL_IDS(info) \
 > 		+	INTEL_MTL_M_IDS(info), \
 > 		+	INTEL_MTL_P_IDS(info)
 > 		+
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 2/2] drm/i915/mtl: Add MeteorLake PCI IDs](https://lists.freedesktop.org/archives/intel-gfx/2022-July/301010.html) {{< /quote >}}

| Intel GPU ({ver}.{release_ver}) | Graphics IP ver | Display IP ver | Media IP ver |
| :-- | :--: | :--: | :--: |
| Tiger Lake, Rocket Lake? | 12 | 12 | 12 |
| DG1 | 12.1 | 12 | 12 |
| Alder Lake-S | 12 | 12 | 12.2 |
| Alder Lake-P | 12 | 13 (XE_LPD) | 12.2 |
| {{< xe class="hp" >}} | 12.5 | - | 12.5 |
| {{< xe class="hpg" >}} | 12.55 | 13 (XE_LPD) | 12.55 |
| {{< xe class="hpc" >}} | 12.6 | - | 12.6 |
| Meteor Lake | 12.7 | 14 (XE_LPD+) | 13 (XE_LPM) |

