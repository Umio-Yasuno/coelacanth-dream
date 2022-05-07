---
title: "さらなる RDNA 2 APU の IPブロック ―― GC 10.3.6, DCN 3.1.5, VCN 3.1.2"
date: 2022-02-17T08:25:01+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "GC_10_3_6", "RDNA_2", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日の新たな IPブロック、*GC 10.3.7, DCN 3.1.6, VCN 3.1.1* に続き、Linux Kernel における AMDGPUドライバーに向けて *GC 10.3.6, DCN 3.1.5, VCN 3.1.2* のサポートを追加するパッチが Alex Deucher氏より投稿されている。  
{{< link >}} [新たな RDNA 2 APU の IPブロック ―― GC 10.3.7, DCN 3.1.6, VCN 3.1.1 | Coelacanth's Dream](/posts/2022/02/17/amdgpu-new-rdna_2-ip/) {{< /link >}}

IPバージョンベースのドライバーサポートについては、前回か以下の記事を参照。  
{{< link >}} [IPバージョンベースのサポートに移行する AMDGPUドライバー | Coelacanth's Dream](/posts/2022/01/31/amdgpu-ip-version/) {{< /link >}}

* [[PATCH] drm/amdgpu: add nv common init for gc 10.3.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075519.html)
* [[PATCH] drm/amdgpu: add Clock and Power Gating support for gc 10.3.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075520.html)
* [[PATCH] drm/amdgpu: add support for gmc10 for gc 10.3.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075521.html)
* [[PATCH] drm/amdgpu: add gc 10.3.6 support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075522.html)
* [[PATCH] drm/amdgpu: add support for sdma 5.2.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075585.html)
* [[PATCH] drm/amdgpu/vcn: add vcn support for vcn 3.1.2](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075586.html)
* [[PATCH] drm/amdgpu: enable vcn pg and cg for vcn 3.1.2](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075587.html)
* [[PATCH 00/13] Update DCN 3.1 support for 3.1.5](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075706.html)

詳細は後述するが、*GC 10.3.6* は APU であり、*DCN 3.1.5, VCN 3.1.2* は *GC 10.3.6* に関連付けられている。  
ほぼ同じタイミングでその他 IPブロックサポートへのパッチも投稿されているが、*GC 10.3.6* に明確に関連付けられてはいない。  

## GC 10.3.6 {#gc}
*GC 10.3.7* と同様、コードネームの代わりとして *GC_10_3_6* が使われている。  
*GC 10.3.6* は IPバージョンこそ *Yellow Carp/Rembrandt APU* と *GC 10.3.7* に近いが、別の Family ID が割り当てられている。  
Family ID は AMD GPUドライバーでは主にディスプレイ部のコードで使われている。*RDNA 2* APU では *VanGogh, Yellow Carp/Rembrandt, GC 10.3.6, GC 10.3.7* でそれぞれ Family ID が異なることになるが、ディスプレイ部にそれだけ違いがあるということなのかもしれない。  

 > 		 #define AMDGPU_FAMILY_VGH			144 /* Van Gogh */
 > 		 #define AMDGPU_FAMILY_YC			146 /* Yellow Carp */
 > 		+#define AMDGPU_FAMILY_GC_10_3_6			149 /* GC 10.3.6 */
 > 		 #define AMDGPU_FAMILY_GC_10_3_7			151 /* GC 10.3.7 */
 > 		
 > {{< quote >}} [[PATCH] drm/amdgpu: add gc 10.3.6 support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075522.html) {{< /quote >}}

APU のビットフラグが立てられていることから、*GC 10.3.6* は APU であることがわかる。  
補足すると、以下引用部にある *GC 10.3.1* は *VanGogh/Aerith APU* 、*GC 10.3.3* は *Yellow Carp/Rembrandt APU* の IPバージョンとなる。  

 > 		 	case IP_VERSION(10, 1, 4):
 > 		 	case IP_VERSION(10, 3, 1):
 > 		 	case IP_VERSION(10, 3, 3):
 > 		+	case IP_VERSION(10, 3, 6):
 > 		 	case IP_VERSION(10, 3, 7):
 > 		 		adev->flags |= AMD_IS_APU;
 > 		 		break;
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: add gc 10.3.6 support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075522.html) {{< /quote >}}

GPUキャッシュ構成を示すコードが追加されていない点も *GC 10.3.7* と同様。  

*GC 10.3.6/10.3.7* では、ShaderArray が無効化されることを想定したコードがあるが、*Yellow Carp/Rembrandt APU* のように複数の ShaderArray を持ち、そして *Radeon 660M* モデルのように一部 ShaderArray を無効化した SKU を計画しているのかもしれない。  

 > 		 		for (j = 0; j < adev->gfx.config.max_sh_per_se; j++) {
 > 		 			bitmap = i * adev->gfx.config.max_sh_per_se + j;
 > 		 			if (((adev->ip_versions[GC_HWIP][0] == IP_VERSION(10, 3, 0)) ||
 > 		-				(adev->ip_versions[GC_HWIP][0] == IP_VERSION(10, 3, 3)) ||
 > 		-				(adev->ip_versions[GC_HWIP][0] == IP_VERSION(10, 3, 7))) &&
 > 		+			     (adev->ip_versions[GC_HWIP][0] == IP_VERSION(10, 3, 3)) ||
 > 		+			     (adev->ip_versions[GC_HWIP][0] == IP_VERSION(10, 3, 6)) ||
 > 		+			     (adev->ip_versions[GC_HWIP][0] == IP_VERSION(10, 3, 7))) &&
 > 		 			    ((gfx_v10_3_get_disabled_sa(adev) >> bitmap) & 1))
 > 		 				continue;
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: add gc 10.3.6 support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075522.html) {{< /quote >}}

## VCN 3.1.2 {#vcn}
*GC 10.3.6* には *VCN 3.1.2* が関連付けられており、*Yellow Carp/Rembrandt* と *GC 10.3.7* に採用されている *VCN 3.1.1* とは異なる。  
だが IPバージョン以外の違いは不明で、対応するコーデックとサイズは *VCN 3.1.1* とされている。  

 > 		 	case IP_VERSION(3, 1, 1):
 > 		+	case IP_VERSION(3, 1, 2):
 > 		 		if (encode)
 > 		 			*codecs = &nv_video_codecs_encode;
 > 		 		else
 >
 > {{< quote >}} [[PATCH] drm/amdgpu/vcn: add vcn support for vcn 3.1.2](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075586.html) {{< /quote >}}

## DCN 3.1.5 {#dcn}
* [[PATCH 00/13] Update DCN 3.1 support for 3.1.5](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075706.html)
    * [[PATCH 03/13] drm/amd/display: Add DCN315 family information](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075707.html)
    * [[PATCH 04/13] drm/amd/display: Add DCN315 CLK_MGR](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075708.html)
    * [[PATCH 05/13] drm/amd/display: Add DCN315 GPIO](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075710.html)
    * [[PATCH 06/13] drm/amd/display: Add DCN315 IRQ](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075711.html)
    * [[PATCH 07/13] drm/amd/display: Add DCN315 DMUB](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075712.html)
    * [[PATCH 08/13] drm/amd/display: Add DCN315 Resource](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075713.html)
    * [[PATCH 09/13] drm/amd/display: Add DCN315 Command Table Helper](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075709.html)
    * [[PATCH 10/13] drm/amd/display: Add DCN315 blocks to Makefile](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075715.html)
    * [[PATCH 11/13] drm/amd/display: Add DCN315 CORE](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075717.html)
    * [[PATCH 12/13] drm/amd/display: Add DCN315 DM Support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075716.html)
    * [[PATCH 13/13] drm/amdgpu: add dm ip block for dcn 3.1.5](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075714.html)

*GC 10.3.6* には *DCN 3.1.5* が関連付けられており、*Yellow Carp/Rembrandt* とも *GC 10.3.7* とも異なる。  

 > 		+	case AMDGPU_FAMILY_GC_10_3_6:
 > 		+		if (ASICREV_IS_GC_10_3_6(asic_id.hw_internal_rev))
 > 		+			dc_version = DCN_VERSION_3_15;
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 11/13] drm/amd/display: Add DCN315 CORE](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075717.html) {{< /quote >}}

最大 4画面出力に対応する点は *GC 10.3.7* 等と同じで、APU ではよくある規模となっている。  

 > 		+static const struct resource_caps res_cap_dcn31 = {
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
 > 		+	.num_dsc = 3,
 > 		+};
 >
 > {{< quote >}} [[PATCH 5/6] drm/amd/display: Add DCN316 resource and SMU clock manager](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075448.html) {{< /quote >}}

Watermark (WM) Table に DDR5用と LPDDR5用が用意されている。  
この部分では DDR4用と LPDDR5用の WM Table がある *GC 10.3.7, DCN 3.1.6* と異なり、*Yellow Carp/Rembrandt* と一致する。  

 > 		+		if (ctx->dc_bios->integrated_info->memory_type == LpDdr5MemType) {
 > 		+			dcn315_bw_params.wm_table = lpddr5_wm_table;
 > 		+		} else {
 > 		+			dcn315_bw_params.wm_table = ddr5_wm_table;
 > 		+		}
 >
 > {{< quote >}} [[PATCH 04/13] drm/amd/display: Add DCN315 CLK_MGR](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075708.html) {{< /quote >}}

今回のパッチを含め、短期間で新たな *RDNA 2* APU 2つに向けたパッチが投稿された。どちらも既存の *RDNA 2* APU、*VanGogh/Aerith* と *Yellow Carp/Rembrandt* に近い部分を持つが、完全に同じではない。  
また、IPバージョンベースのサポートに移行してから、初めて新たな APU がサポートされた。  
実質的に AMDGPUドライバーにおけるコードネームは廃止され、より効率的なサポート方式となった。  
他の OSS、User Moder Driver である Mesa3D (RadeonSI, RADV) では GC (Graphics and Compute) の IPバージョンを、コンパイラバックエンドに採用されている LLVM、ROCmソフトウェアではこれまで通り GPUID (gfx{major.minor.stepping}) を使うものと思われる。  

サポートという一点では DeviceID ベースよりも効率的となったのだから喜ばしいことだと言えるが、コードネームが出てこなくなるということは少し寂しく感じる。  

