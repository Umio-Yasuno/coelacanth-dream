---
title: "RDNA 3/GFX11 APU に採用される DCN 3.1.4"
date: 2022-07-09T21:01:46+09:00
draft: false
categories: [ "Hardware", "APU", "AMD" ]
tags: [ "GFX11", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Alex Deucher 氏より、AMDGPU ドライバーに *RDNA 3/GFX11 APU* で採用されるディスプレイエンジン *DCN 3.1.4 (Display Core Next)* のサポートを追加するパッチが amd-gfx メーリングリストに投稿されている。  
例によってレジスタヘッダはパッチサイズが巨大となるため、メーリングリストには投稿されていない。補足すると、最近追加された `dcn_3_2_0_sh_mask.h (drivers/gpu/drm/amd/include/asic_reg/dcn/)` はファイル単体で 23.9MiB もある。  
以前は `nbio_7_2_0_sh_mask.h (drivers/gpu/drm/amd/include/asic_reg/nbio/)` が 15.9MiB で最大だったと思うのだが、一気に最大ファイルサイズが更新されたようだ。  
もはや余談だが、こうした自動生成された巨大なレジスタヘッダについて、Linus Torvalds 氏は以下のようにコメントしている。  

 > 		>
 > 		> AMD has started some new GPU support [...]
 > 		
 > 		Oh Christ. Which means another set of auto-generated monster headers. Lovely.
 > 		
 > 		                  Linus
 >
 > {{< quote >}} [LKML: Linus Torvalds: Re: [git pull] drm for 5.19-rc1](https://lkml.org/lkml/2022/5/25/1144) {{< /quote >}}

 * [[PATCH 0/9] Add DCN 3.1.4 Support](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081237.html)

## AMDGPU_FAMILY_GC_11_0_2 {#family_gc_11_0_2}
パッチでは `AMDGPU_FAMILY_GC_11_0_2` に関するコードも追加されており、そして `AMDGPU_FAMILY_GC_11_0_2` には *DCN 3.1.4* が結び付けられている。  
少しややこしいのが、`AMDGPU_FAMILY_GC_11_0_2` はあくまでも `AMDGPU_FAMILY` 内での ID を示し、*GC 11.0.2* とは別と思われることだ。  

 * [[PATCH 6/9] drm/amd/display: Add DCN314 version identifiers](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081240.html)
 * [[PATCH 7/9] drm/amd/display: Enable DCN314 in DC](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081242.html)

`AMDGPU_FAMILY_GC_11_0_2` の ID は `148 (0x94)` とされ、これは以前 AMD の Marek Olšák 氏によって RadeonSI ドライバーに投稿されたパッチと比較すると、`FAMILY_GFX1103` の ID と一致する。  
{{< link >}} [RadeonSI ドライバーで GFX11 のサポートが進められる | Coelacanth's Dream](/posts/2022/05/05/radeonsi-gfx11/#chip_family) {{< /link >}}
*gfx1103* は LLVM のドキュメントでは APU とされている。*GC 11.0.1* も AMDGPU ドライバー内で APU とされており、そして *GC 11.0.1* は *gfx1103* とされている。  
{{< link >}} [LLVM で GFX11 のサポートが進み始める | Coelacanth's Dream](/posts/2022/04/28/llvm-gfx11/) {{< /link >}}

 > 		 #define AMDGPU_FAMILY_GC_11_0_0 145
 > 		+#define AMDGPU_FAMILY_GC_11_0_2 148
 >
 > {{< quote >}} [[PATCH 6/9] drm/amd/display: Add DCN314 version identifiers](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081240.html) {{< /quote >}}
 >
 > 		+#define FAMILY_GFX1100 0x91
 > 		+#define FAMILY_GFX1103 0x94
 >
 > {{< quote >}} [amd: add chip identification for gfx1100-1103 (8e6d5992) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/8e6d599238881bc63590457813dc8532cb5462ad) {{< /quote >}}

 > 				case IP_VERSION(11, 0, 1):
 > 					gfx_target_version = 110003;
 > 					f2g = &gfx_v11_kfd2kgd;
 > 					break;
 >
 > {{< quote >}} drivers/gpu/drm/amd/amdkfd/kfd_device.c {{< /quote >}}
 >
 > 			case IP_VERSION(10, 1, 3):
 > 			case IP_VERSION(10, 1, 4):
 > 			case IP_VERSION(10, 3, 1):
 > 			case IP_VERSION(10, 3, 3):
 > 			case IP_VERSION(10, 3, 6):
 > 			case IP_VERSION(10, 3, 7):
 > 			case IP_VERSION(11, 0, 1):
 > 				adev->flags |= AMD_IS_APU;
 > 				break;
 >
 > {{< quote >}} drivers/gpu/drm/amd/amdgpu/amdgpu_discovery.c {{< /quote >}}

現状、*GC IP* と GFX ID と `AMDGPU_FAMILY` の関係性は以下のようになっている。  

| GC (Graphics Compute) IP ver | GFX ID | AMDGPU_FAMILY | Type |
| :-------- | :-----: | :--: | :--: |
| 11.0.0    | gfx1100 (Navi31)[^tensile-gfx11] | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100) | dGPU |
| 11.0.1    | gfx1103 | AMDGPU_FAMILY_GC_11_0_2 (FAMILY_GFX1103) | APU  |
| 11.0.2    | gfx1102 (Navi32) | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100) | dGPU |
| 11.0.3?   | gfx1101 (Navi33)? | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100)? | dGPU? |

[^tensile-gfx11]: [Support Tensile for gfx11 series platform by TonyYHsieh · Pull Request #1521 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1521/commits/3796d41aec358721fced1ed4337c27f69aeda3bb#diff-95d409aa7c33d03c94333a9a95ce6076cabf7428d1613137ccc7944151cd0972)

以上のことから、*RDNA 3/GFX11 APU* である *GC 11.0.1 (gfx1103)* ではディスプレイエンジンに *DCN 3.1.4* を採用することがわかる。  
*DCN 3.1.4* に関するコード部にも `dc->caps.is_apu = true;` の記述がある。  

 * [[PATCH 4/9] drm/amd/display: Add DCN314 DC resources](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081245.html)

## DCN 3.1.4 {#dcn3_1_4}
*DCN 3.1.4* 自体を見ると、まずバージョンは *RDNA 3/GFX11 dGPU* で採用されている *DCN 3.2.x* ではなく、*RDNA 2/GFX10.3 APU* で採用されている *DCN 3.1.x* 系列となる。  
*GC 11.0.0/Navi31* は *DCN 3.2.0* 、*GC 11.0.2/Navi32* は *DCN 3.2.1* を、  
*Yellow Carp/Rembrandt APU* が *DCN 3.1.0*、*GC 10.3.6 APU* が *DCN 3.1.5*、*GC 10.3.7/Sabrina APU* が *DCN 3.1.6* を採用している。  

また、*MALL (Memory Access at Last Level) /Infinity Cache* に関する記述がないことから、*RDNA 3/GFX11 APU* でも引き続き *MALL/Infinity Cache* は搭載しない方針と思われる。  

*DCN 3.1.4* の最大画面出力数は 4基と見られ、APU としては通常の規模となる。  
ただ *RDNA 3/GFX11 dGPU* に採用される *DCN 3.2.{0,1}* も最大 4基とする記述が見られる。[^dcn_3_2] 今後は dGPU でも最大 4基が通常となるのだろうか。  

[^dcn_3_2]: [[PATCH 12/43] drm/amd/display: Add DM support for DCN32/DCN321](https://lists.freedesktop.org/archives/amd-gfx/2022-May/079583.html)

 > 		+static const struct resource_caps res_cap_dcn314 = {
 > 		+	.num_timing_generator = 4,
 > 		+	.num_opp = 4,
 > 		+	.num_video_plane = 4,
 > 		+	.num_audio = 5,
 > 		+	.num_stream_encoder = 5,
 > 		+	.num_dig_link_enc = 5,
 > 		+	.num_hpo_dp_stream_encoder = 4,
 > 		+	.num_hpo_dp_link_encoder = 2,
 > 		+	.num_pll = 5,
 > 		+	.num_dwb = 1,
 > 		+	.num_ddc = 5,
 > 		+	.num_vmid = 16,
 > 		+	.num_mpc_3dlut = 2,
 > 		+	.num_dsc = 4,
 > 		+};
 >
 > {{< quote >}} [[PATCH 4/9] drm/amd/display: Add DCN314 DC resources](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081245.html) {{< /quote >}}

WaterMark Table には LPDDR5用と DDR5用があり、*GC 11.0.1 APU* ではメモリにそれらをサポートしていると考えられる。  

 > 		+		if (ctx->dc_bios->integrated_info->memory_type == LpDdr5MemType)
 > 		+			dcn314_bw_params.wm_table = lpddr5_wm_table;
 > 		+		else
 > 		+			dcn314_bw_params.wm_table = ddr5_wm_table;
 >
 > {{< quote >}} [[PATCH 3/9] drm/amd/display: Add DCN314 clock manager](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081238.html) {{< /quote >}}

{{< ref >}}
 * [Support Tensile for gfx11 series platform by TonyYHsieh · Pull Request #1521 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1521/commits/3796d41aec358721fced1ed4337c27f69aeda3bb#diff-95d409aa7c33d03c94333a9a95ce6076cabf7428d1613137ccc7944151cd0972)
{{< /ref >}}
