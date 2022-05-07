---
title: "Intel GPU ドライバーに Ponte Vecchio をサポートするパッチが投稿される"
date: 2022-05-05T12:49:26+09:00
draft: false
categories: [ "Intel", "GPU", "Hardware" ]
tags: [ "Linux_Kernel", "Xe-HPC" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Matt Roper 氏より、Linux Kernel における Intel GPU/GFX ドライバー、i915 に *Ponte Vecchio GPU* を初期サポートを追加するパッチが投稿されている。  

 * [[Intel-gfx] [PATCH 00/11] i915: Introduce Ponte Vecchio](https://lists.freedesktop.org/archives/intel-gfx/2022-May/296862.html)
 * [[Intel-gfx] [PATCH v2 00/12] i915: Introduce Ponte Vecchio](https://lists.freedesktop.org/archives/intel-gfx/2022-May/297296.html)

パッチ内のコメントで Matt Roper 氏は、*Ponte Vecchio* は *{{< xe class="hpc" >}} アーキテクチャ* に基づいたコンピューティング向け GPU であり、コンピューティングエンジンと拡張されたコピーエンジン (BCS, blitting {copy} engine) を備えるが、ジオメトリパイプラインを含むレンダーエンジンは持たず、またディスプレイ出力機能も持たない、コンピューティング向けプラットフォームだと説明している。  

ディスプレイ出力を持たない Intel GPU には他に、サーバー向けの *XeHP_SDV* と *{{< xe class="hpg" >}}* をベースにした *ArcticSound-M* がいる。[^ats_m] [^xehp_sdv]  

[^ats_m]: [[Intel-gfx] [PATCH 1/2] drm/i915/ats-m: add ATS-M platform info](https://lists.freedesktop.org/archives/intel-gfx/2022-March/294043.html)
[^xehp_sdv]: [[Intel-gfx] [PATCH v2 15/50] drm/i915/xehpsdv: add initial XeHP SDV definitions](https://lists.freedesktop.org/archives/intel-gfx/2021-July/271840.html)

今回のパッチは初期的な、エンジンの有効化を目的としたパッチであり、対象のエンジンも拡張されたコピーエンジンが主となる。  

## Copy Engine {#copy_engine}
BCS はデータの移動やメモリの指定された部分を固定データで埋める機能を持つ。  
Intel GPU では BCS を通常 1基備えているが、*Ponte Vecchio* では 8基備えていることを示す記述がある。  
*Ponte Vecchio* はコンピューティング向けとしてマルチ GPU を想定しており、BCS の拡張と複数搭載はマルチ GPU 性能を高めるためのものと思われる。  

 > 		+#define   XEHPC_BCS1				(14)
 > 		+#define   XEHPC_BCS2				(13)
 > 		+#define   XEHPC_BCS3				(12)
 > 		+#define   XEHPC_BCS4				(11)
 > 		+#define   XEHPC_BCS5				(10)
 > 		+#define   XEHPC_BCS6				(9)
 > 		+#define   XEHPC_BCS7				(8)
 > 		+#define   XEHPC_BCS8				(23)
 >
 > {{< quote >}} [[Intel-gfx] [PATCH v2 08/12] drm/i915/pvc: Engine definitions for new copy engines](https://lists.freedesktop.org/archives/intel-gfx/2022-May/297308.html) {{< /quote >}}

## Platform info {#info}

*{{< xe class="hpc" >}} アーキテクチャ* で DMA に使用可能なアドレスサイズは 52-bits であり、*{{< xe class="hp/hpg" >}}* の 46-bits より広いアドレスサイズとなっている。  
Graphics IP と Media IP のバージョンについては、世代を表す基本的なバージョンは 12、プラットフォームの違いを表す rel (release) 部分は 60 となる。  

 > 		+#define XE_HPC_FEATURES \
 > 		+	XE_HP_FEATURES, \
 > 		+	.dma_mask_size = 52
 > 		+
 > 		+__maybe_unused
 > 		+static const struct intel_device_info pvc_info = {
 > 		+	XE_HPC_FEATURES,
 > 		+	XE_HPM_FEATURES,
 > 		+	DGFX_FEATURES,
 > 		+	.graphics.rel = 60,
 > 		+	.media.rel = 60,
 > 		+	PLATFORM(INTEL_PONTEVECCHIO),
 > 		+	.display = { 0 },
 > 		+	.has_flat_ccs = 0,
 > 		+	.platform_engine_mask =
 > 		+		BIT(BCS0) |
 > 		+		BIT(VCS0) |
 > 		+		BIT(CCS0) | BIT(CCS1) | BIT(CCS2) | BIT(CCS3),
 > 		+	.require_force_probe = 1,
 > 		+};
 > 		+
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 01/11] drm/i915/pvc: add initial Ponte Vecchio definitions](https://lists.freedesktop.org/archives/intel-gfx/2022-May/296869.html) {{< /quote >}}

| Intel GPU / (ver.release_ver) | Graphics IP ver | Display IP ver | Media IP ver |
| :-- | :--: | :--: | :--: |
| Tiger Lake,<br>Rocket Lake? | Gen12 | Gen12 | Gen12 |
| DG1 | Gen12.1 | Gen12 | Gen12 |
| Alder Lake-S | Gen12 | Gen12 | Gen12.2[^gen12_2] |
| Alder Lake-P | Gen12 | Gen13<br>([XE_LPD](/tags/xe_lpd)) | Gen12.2[^gen12_2] |
| {{< xe class="hp" >}} | Gen12.5 | - | Gen12.5 |
| {{< xe class="hpg" >}} | Gen12.55 | Gen13<br>([XE_LPD](/tags/xe_lpd)) | Gen12.55 |
| {{< xe class="hpc" >}} | Gen12.6 | - | Gen12.6 |


{{< ref >}}
 * [drm/i915 Intel GFX Driver — The Linux Kernel documentation](https://01.org/linuxgraphics/gfx-docs/drm/gpu/i915.html)
 * [[Intel-gfx] [PATCH 01/53] drm/i915: Add "release id" version](https://lists.freedesktop.org/archives/intel-gfx/2021-July/270870.html)
 * [Intel® Iris® Xe MAX Graphics Open Source Programmer's Reference Manual For the 2020 Discrete GPU formerly named "DG1" Volume 10: Copy Engine - intel-gfx-prm-osrc-dg1-vol10-copyengine.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-dg1-vol10-copyengine.pdf)
{{< /ref  >}}
