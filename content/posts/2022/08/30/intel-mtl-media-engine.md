---
title: "GPU と Media Engine が別々のタイルに搭載される Meteor Lake"
date: 2022-08-30T23:02:25+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Meteor_Lake", "Linux_Kernel", "Xe-HP", "Xe-HPC" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel のソフトウェアエンジニア Matt Roper 氏により、Linux Kernel における Intel GPU/GFX ドライバー `i915` に、*Meteor Lake* 向けた *Standalone Media* のサポートを追加するパッチが投稿されている。  

 * [[Intel-gfx] [PATCH 0/8] i915: Add "standalone media" support for MTL](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304507.html)
 * [[Intel-gfx] [PATCH v2 4/8] drm/i915: Prepare more multi-GT initialization](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304531.html)
 * [[Intel-gfx] [PATCH 5/8] drm/i915: Rename and expose common GT early init routine](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304512.html)
 * [[Intel-gfx] [PATCH 6/8] drm/i915/xelpmp: Expose media as another GT](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304513.html)

## Standalone Media Engine {#standalone}
*Meteor Lake* からメディア機能 (Media Engine, {{< xe >}}-LPM+) はハードウェアレベルにおいて 2番目の GT に移行し、*Standalone Media* と呼ばれる。  
*GT* は *Graphics Technology* の略であり、Intel GPU/GFX ドライバーでは GPU 全体、あるいは GPUダイ/タイル単位で指す用語として使われている。  

*Standalone Media* では独自の *GuC (Graphics micro (µ) Controller)*、電源管理、forcewake (強制起動?) を持ち、メインの GPUコア部とは分離されている。  
従来の Intel GPU において Media Engine は、他の GPU ユニットと同じ *GT* 1基、GPUダイ/タイルに属していた。  
こうした Media Engine とドライバー変更は *Meteor Lake* では SoC Tile に Media Engine が搭載されることが影響していると思われる。  
Intel は 2022/08/21 - 2022/08/23 に開催された Hot Chips 34 にて、*Meteor Lake* と *Arrow Lake* に採用される Foveros パッケージング技術に関する発表を行っており、その中で Media Engine が GPU Tile ではなく SoC Tile に統合されることを示していた。  

パッチ内で Matt Roper 氏は、*Standalone Media* は *{{< xe class="hp" >}} SDV* や *Ponte Vecchio*における remote tile(s) に多くの点で類似しており、コードのインフラも多くで共有できるとしている。  
しかし重要な違いがいくつか存在し、パッチはそれに対応するためのものとなる。  

 > 		Starting with MTL, media functionality has moved into a new, second GT
 > 		at the hardware level.  This new GT, referred to as "standalone media"
 > 		in the spec, has its own GuC, power management/forcewake, etc.  The
 > 		general non-engine GT registers for standalone media start at 0x380000,
 > 		but otherwise use the same MMIO offsets as the primary GT.
 > 		
 > 		Standalone media has a lot of similarity to the remote tiles
 > 		present on platforms like xehpsdv and pvc, and our i915 implementation
 > 		can share much of the general "multi GT" infrastructure between the two
 > 		types of platforms.  However there are a few notable differences
 > 		we must deal with:
 > 		
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 0/8] i915: Add "standalone media" support for MTL](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304507.html) {{< /quote >}}

複数の GPU/Tile で構成される *{{< xe >}}-HP SDV* や *Ponte Vecchio* といったプラットフォームをサポートするパッチは以前に投稿されており、CPU と接続されている GPU/Tile を primary/root とし、それを経由して他の GPU/Tile の初期化を実行する方法を取っていた。[^intel-multi-tile]  

[^intel-multi-tile]: [Intel のマルチタイル GPU をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/10/10/intel-xe_hp-multi-tile/)

*Standalone Media* では、複数の GPU/Tile で構成されるプラットフォームとは異なり、*primary GT* を経由して割り込み処理が行われるとされている。  
この点から、*Meteor Lake* では GPU と Media Engine が別々のタイルに搭載されるが、GPU が無効化された SKU では Media Engine も無効化されるものと思われる。  
また、GPU が無効化されている場合は DeviceID からプラットフォームの検出ができないようにも思う。  

 > 		 - Unlike platforms with remote tiles, all interrupt handling for
 > 		   standalone media still happens via the primary GT.
 > 		
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 0/8] i915: Add "standalone media" support for MTL](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304507.html) {{< /quote >}}

*Standalone Media ({{< xe >}}-LPM+)* の `engine_mask` を見ると `VECS0, VCS0, VCS2` のビットフラグが有効化されている。  
`VCS` はデコードエンジン (VDBox, BSD, Bit Stream Decode)、`VCES` はエンコードエンジン (VEBox, Video Enhancement Engine) を指し、エンジン数だけで言えば *Tiger Lake, Alder Lake, DG1* と同規模となる。  
*DG2/Alchemist* に採用されている *{{< xe >}}-HPM (ver12.50)* は AV1エンコードをサポートしているが、*Meteor Lake* に採用される *{{< xe >}}-LPM+ (ver13.00)* はまだ [intel/media-driver](https://github.com/intel/media-driver) 等へのパッチが公開されておらず、AV1エンコードのサポート状況については不明となっている。  

 > 		+static const struct intel_gt_definition xelpmp_extra_gt[] = {
 > 		+	{
 > 		+		.type = GT_MEDIA,
 > 		+		.name = "Standalone Media GT",
 > 		+		.setup = intel_sa_mediagt_setup,
 > 		+		.gsi_offset = MTL_MEDIA_GSI_BASE,
 > 		+		.engine_mask = BIT(VECS0) | BIT(VCS0) | BIT(VCS2),
 > 		+	},
 > 		+	{}
 > 		+};
 > 		+
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 6/8] drm/i915/xelpmp: Expose media as another GT](https://lists.freedesktop.org/archives/intel-gfx/2022-August/304513.html) {{< /quote >}}

| Intel Media Engine | VEBox | VDBox |
| :--                | :--:  | :--:  |
| TGL/DG1/ADL        | 1     | 2     |
| {{< xe >}}-HP SDV  | 4?    | 4?    |
| DG2/Alchemist      | 2?    | 2?    |
| Meteor Lake        | 1?    | 2?    |

{{< ref >}}
 * [Intel Meteor Lake and Arrow Lake - HotChips 34レポート | マイナビニュース](https://news.mynavi.jp/article/20220824-2433247/)
 * [Intel Disaggregates Client Chips with Meteor Lake in 2023](https://www.servethehome.com/intel-disaggregates-client-chips-with-meteor-lake-hc34/)
 * [Intel GFX ドライバーに Meteor Lake をサポートする最初のパッチが投稿される ―― Gen12.7, Xe_LPD+, Xe_LPM+ | Coelacanth's Dream](/posts/2022/07/07/i915-mtl/)
 * Linux Kernel
    * `drivers/gpu/drm/i915/i915_pci.c`
{{< /ref >}}
