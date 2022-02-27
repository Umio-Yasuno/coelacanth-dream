---
title: "Intel DG2-G10 (512EU) Bootlog"
date: 2022-02-27T12:09:56+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "DG2", "Xe-HPG", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel GFX CI に、*i5-9600K (Coffee Lake)* と *DG2-G10 (512EU)* を組み合わせたプラットフォームのブートログがアップロードされていた。  

* [bat-dg2-9 CI History](https://intel-gfx-ci.01.org/tree/drm-tip/bat-dg2-9.html)

## DG2-G10 {#g10}
DeviceID (PCI ID) は `0x56A0` であり、これは複数のバリアントがある DG2 の中で、*DG2-G10* に割り当てられた ID となっている。  
Graphics ver: 12.55, Media ver: 12.55, Display ver: 13 というのは、ドライバーのソースコードに記述されている情報をそのまま使っていると思われる。  
{{< link >}} [Linux Kernel に Intel Xe-HP SDV, DG2 をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/07/02/intel-xe_hp-dg2-linux-kernel-patch/) {{< /link >}}

 > 		<7>[    5.768646] i915 device info: pciid=0x56a0 rev=0x08 platform=DG2 (subplatform=0x1) gen=12
 > 		<7>[    5.768648] i915 device info: graphics version: 12.55
 > 		<7>[    5.768650] i915 device info: media version: 12.55
 > 		<7>[    5.768651] i915 device info: display version: 13
 >
 > {{< quote >}} [bat-dg2-9 CI History](https://intel-gfx-ci.01.org/tree/drm-tip/bat-dg2-9.html) {{< /quote >}}
 >
 > 		+/* DG2 */
 > 		+#define INTEL_DG2_G10_IDS(info) \
 > 		+	INTEL_VGA_DEVICE(0x5690, info), \
 > 		+	INTEL_VGA_DEVICE(0x5691, info), \
 > 		+	INTEL_VGA_DEVICE(0x5692, info), \
 > 		+	INTEL_VGA_DEVICE(0x56A0, info), \
 > 		+	INTEL_VGA_DEVICE(0x56A1, info), \
 > 		+	INTEL_VGA_DEVICE(0x56A2, info)
 >
 > {{< quote >}} [[Intel-gfx] [PATCH topic/for-CI] drm/i915: Add DG2 PCI IDs](https://lists.freedesktop.org/archives/intel-gfx/2022-February/289813.html) {{< /quote >}}

EU数には 512基が認識されており、内訳は Slice 1基、Sub-Slice 32基、Sub-Slice あたりの EU 16基となる。  
*DG2/Achemist* には *G10/G11/G12* という 3種のバリアントが存在することが、オープンソースドライバーへのパッチにて明かされており、それぞれの規模については不明だったが、今回で少なくとも *DG2-G10* が 512EU 構成であることがわかった。  
{{< xe >}}-Core に相当する Sub-Slice が 32基という構成は、Intel Architecture Day 2021 でも公開されていたものであり、*DG2/Alchemist* では *DG2-G10* が最大規模となるのではないかと思われる。  
*DG2-G11/G12* の規模は依然として不明。  

動作クロックについては不明。  
もしかしたらブートログ中にあるのかもしれないが、自分にはわからなかった。  
ログ中の CDCLK (Core Display Clock) はディスプレイエンジン部のクロック、RAWCLK はバックライトPWM等のブロックで使用される固定クロックであり、どちらも GPUコア部の動作クロックではない。  

 > 		<7>[    5.768720] i915 device info: slice total: 1, mask=0001
 > 		<7>[    5.768722] i915 device info: subslice total: 32
 > 		<7>[    5.768723] i915 device info: slice0: 32 subslices, mask=ffffffff
 > 		<7>[    5.768725] i915 device info: EU total: 512
 > 		<7>[    5.768726] i915 device info: EU per subslice: 16
 > 		<7>[    5.768727] i915 device info: has slice power gating: yes
 > 		<7>[    5.768728] i915 device info: has subslice power gating: no
 > 		<7>[    5.768729] i915 device info: has EU power gating: no
 >
 > {{< quote >}} [bat-dg2-9 CI History](https://intel-gfx-ci.01.org/tree/drm-tip/bat-dg2-9.html) {{< /quote >}}

*DG2-G10* のメモリサイズは `Local memory available` の値から 16 GiB ではないかと思われる。(0x00000003fa000000 -\> 17079205888 [byte] -\> 15.9 [GiB] )  
*DG2/Alchemist* では GDDR6メモリを採用することが発表されているため、*DG2-G10* のメモリバス幅はメモリサイズから 256-bit だろうか。  

 > 		<7>[    5.475163] i915 0000:03:00.0: [drm:i915_gem_stolen_lmem_setup [i915]] Stolen Local memory IO start: 0x00000043fe800000
 > 		<7>[    5.476003] i915 0000:03:00.0: [drm:intel_gt_setup_lmem [i915]] Local memory: [mem 0x00000000-0x3f9ffffff]
 > 		<7>[    5.476139] i915 0000:03:00.0: [drm:intel_gt_setup_lmem [i915]] Local memory IO start: 0x0000004000000000
 > 		<6>[    5.476252] i915 0000:03:00.0: [drm] Local memory available: 0x00000003fa000000
 >
 > {{< quote >}} [bat-dg2-9 CI History](https://intel-gfx-ci.01.org/tree/drm-tip/bat-dg2-9.html) {{< /quote >}}

余談となるが、*DG2* の D は Dash の意らしい。  

 > 		<7>[    5.526277] i915 0000:03:00.0: [drm:intel_bios_init [i915]] Found valid VBT in SPI flash
 > 		<7>[    5.526427] i915 0000:03:00.0: [drm:intel_bios_init [i915]] VBT signature "$VBT DASH G2        ", BDB version 242
 > 		<7>[    5.526549] i915 0000:03:00.0: [drm:intel_bios_init [i915]] BDB_GENERAL_FEATURES int_tv_support 0 int_crt_support 0 lvds_use_ssc 0 lvds_ssc_freq 120000 display_clock_mode 1 fdi_rx_polarity_inverted 0
 > 		<7>[    5.526669] i915 0000:03:00.0: [drm:intel_bios_init [i915]] crt_ddc_bus_pin: 2
 >
 > {{< quote >}} [bat-dg2-9 CI History](https://intel-gfx-ci.01.org/tree/drm-tip/bat-dg2-9.html) {{< /quote >}}

Intel は Investor Meeting 2022 にて、*DG2/Alchemist* の出荷時期について、搭載ノートPCは 2022Q1、デスクトップ向けは 2022Q2、ワークステーション向けは 2022Q3 になると発表している。  
UMD (User Mode Driver) である Mesa3D では現在 *DG2/Alchemist* のサポートが無効化されているが、これは KMD (Kernel Mode Driver) である i915 に合わせるため。[^dg2-pciid] そして i915 に *DG2-G10/G11/G12* の DeviceID (PCI ID) が CI用に追加され、ブートログも確認できる状態にある。  
Linux環境の *DG2/Alchemist* サポート作業は確実に進んでおり、安定版リリースに取り込まれる日は近いのではないかと思われる。  

[^dg2-pciid]: [intel/dev: Add more DG2 pci-ids (still disabled, waiting on i915) (!14884) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14884)

{{< ref >}}
* [Intel Architecture Day 2021](https://www.intel.com/content/www/us/en/newsroom/resources/press-kit-architecture-day-2021.html)
* [Xe-HPG Microarchitecture](https://www.intel.com/content/www/us/en/architecture-and-technology/visual-technology/arc-discrete-graphics/xe-hpg-microarchitecture.html)
* [drm/i915 Intel GFX Driver — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/i915.html)
{{< /ref >}}
