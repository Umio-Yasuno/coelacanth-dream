---
title: "新たな RDNA 2 APU の IPブロック ―― GC 10.3.7, DCN 3.1.6, VCN 3.1.1"
date: 2022-02-17T03:57:56+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Linux_Kernel", "GC_10_3_7", "RDNA_2" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Linux Kernel における AMDGPUドライバーに向けて、新たな *RDNA 2* APU の各種 IPブロックのサポートを追加するパッチが投稿されている。  
各種 IPブロックはほとんどが、直接 IPバージョンを指定する形でサポートが追加されており、従来のように DeviceID から GPU ASIC を検出する形は採っていない。  
こうしたサポート形式は最近になって採られるようになり、VBIOS、ファームウェアから各種 IPバージョンを取得できる *GFX9 (Vega)* とそれ以降の世代では IPバージョンベースのコードに置き換えられている。  
{{< link >}} [IPバージョンベースのサポートに移行する AMDGPUドライバー | Coelacanth's Dream](/posts/2022/01/31/amdgpu-ip-version/) {{< /link >}}

GC: 13.0.7, DCN: 3.1.6, VCN: 3.1.1, NBIO: 7.5.1, SMU: 13.0.9, PSP: 13.0.8, SDMA 5.2.7、これら IPブロックをサポートするパッチが近いタイミングで投稿されている。  
ただ、一部 IPブロックは関連付けられているが、また一部はそうでない。そのため、厳密に言えば今回取り上げるパッチは 1つの AMD APU ではなく、また別の APU か複数の APU を想定している可能性がある。  

パッチでは、従来の DeviceIDベースのサポートと異なり、コードネームにまったく触れていない。  
APU/GPU、SoC として何らかのコードネームはあるかもしれないが、AMDGPUドライバーでそれを基に開発する必要性が薄くなった。  
DeviceID を追加することなく、 IPブロックを持つ AMD APU/GPU ASIC をサポートできることが IPバージョンベースの利点であり、今後はこの形式がデフォルトになると考えられる。  

* [[PATCH] drm/amdgpu: update vcn/jpeg PG flags for VCN 3.1.1](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075407.html)
* [[PATCH] drm/amdgpu/discovery: add nbio sw func for 7.5.1 nbio](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075329.html)
* [[PATCH] drm/amdgpu/discovery: Add 13.0.9 SMUIO block](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075330.html)
* [[PATCH] drm/amd/pm: Add support for MP1 13.0.8](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075431.html)
* [[PATCH] drm/amdgpu: set new revision id for 10.3.7 GC](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075332.html)
* [[PATCH] drm/amdgpu/sdma5.2: add support for SDMA 5.2.7](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075432.html)
* [[PATCH] drm/amdgpu/gfx10: Add GC 10.3.7 Support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075433.html)
* [[PATCH 0/6] Update DCN 3.1 support for 3.1.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075449.html)

補足として、IPバージョンは `{major}.{minor}.{revision}` のフォーマットで記述される。GC (Graphics and Compute) IP は GPUID と近いバージョン、フォーマットとなるが (`{major}.{minor}.{stepping}`) 、完全に一致はしない。  
また、IPブロックは略称が主に使われ、正式名称はドキュメントに記載されている。  

* [Core Driver Infrastructure — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/driver-core.html)

{{< pindex >}}
 * [GC (Graphics and Compute)](#gc)
 * [DCN 3.1.6](#dcn)
{{< /pindex >}}

## GC (Graphics and Compute) {#gc}
コンピュートユニットやグラフィクス専用の固定機能ブロックを含む GFXコア部、GC (Graphics and Compute) のバージョンは 10.3.7。  
ドキュメントでは、GC は GPU内で最大の IPブロックとしており、その関係かコードネームの代わりとして *GC_10_3_7* が使われている。  
IPバージョンベースのサポートでは今後コードネームらしいものは使わない方針となるのだろうか？  

*GC_10_3_7* は、IPバージョンが Major: 10, Minor: 3 となっていることから、*RDNA 2 アーキテクチャ* を採用していると思われ、コードも既存の *RDNA 2* APU/GPU とほとんど共通している。  

GPU部のキャッシュ構成を示すコード部も IPバージョンベースに移行しているが[^cache-ip-base]、*GC_10_3_7* の情報を追加するパッチはまだ投稿、公開されていない。  

[^cache-ip-base]: [kfd_crat.c « amdkfd « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdkfd/kfd_crat.c?h=next-20220216#n1383)

 > 		+#define AMDGPU_FAMILY_GC_10_3_7			151 /* GC 10.3.7 */
 >
 > {{< quote >}} [[PATCH] drm/amdgpu/gfx10: Add GC 10.3.7 Support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075433.html) {{< /quote >}}

 > 		+#define AMDGPU_FAMILY_GC_10_3_7                151
 > 		+#define GC_10_3_7_A0 0x01
 > 		+#define GC_10_3_7_UNKNOWN 0xFF
 > 		+
 > 		+#define ASICREV_IS_GC_10_3_7(eChipRev) ((eChipRev >= GC_10_3_7_A0) && (eChipRev < GC_10_3_7_UNKNOWN))
 >
 > {{< quote >}} [[PATCH 3/6] drm/amd/display: configure dc hw resource for DCN 3.1.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075446.html) {{< /quote >}}

*GC_10_3_7* は VCN 3.1.1 と DCN 3.1.6 に関連付けられている。  
VCN 3.1.1 は [Yellow Carp/Rembrandt APU](/tags/yellow_carp) にも採用されている。  
DCN 3.1.6 については、*Yellow Carp/Rembrandt* の DCN IPバージョンは DCN 3.1.{1, 2} であるため、そこからわずかにアップデートされている。  

* [[PATCH] drm/amdgpu: update vcn/jpeg PG flags for VCN 3.1.1](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075407.html)

 > 		+	case AMDGPU_FAMILY_GC_10_3_7:
 > 		+		if (ASICREV_IS_GC_10_3_7(asic_id.hw_internal_rev))
 > 		+			dc_version = DCN_VERSION_3_16;
 > 		+		break;
 > {{< quote >}} [[PATCH 3/6] drm/amd/display: configure dc hw resource for DCN 3.1.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075446.html) {{< /quote >}}

## DCN 3.1.6 {#dcn}
DCN 3.1.6 をサポートするパッチは 6個あるとしているが、内 2つはレジスタヘッダであり、パッチサイズが巨大なためメーリングリストには投稿されていない。  

* [[PATCH 0/6] Update DCN 3.1 support for 3.1.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075449.html)
* [[PATCH 3/6] drm/amd/display: configure dc hw resource for DCN 3.1.6](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075446.html)
* [[PATCH 4/6] drm/amd/display: Add DMUB support for DCN316](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075447.html)
* [[PATCH 5/6] drm/amd/display: Add DCN316 resource and SMU clock manager](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075448.html)
* [[PATCH 6/6] drm/amdgpu/discovery: Add sw DM function for 3.1.6 DCE](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075450.html)

コードには APU であることを示すフラグがあり、ここから DCN 3.1.6 を採用する *GC_10_3_7* は APU だと考えられる。  

 > 		+	dc->caps.edp_dsc_support = true;
 > 		+	dc->caps.extended_aux_timeout_support = true;
 > 		+	dc->caps.dmcub_support = true;
 > 		+	dc->caps.is_apu = true;
 >
 > {{< quote >}} [[PATCH 5/6] drm/amd/display: Add DCN316 resource and SMU clock manager](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075448.html) {{< /quote >}}

Watermark (WM) table は DDR4、LPDDR5 メモリに向けたものが用意されている。この点は [VanGogh/Aerith APU](/tags/vangogh) と一致する。[^vgh-dcn301]  
*Yellow Carp/Rembrandt* は DDR5、LPDDR5 分の WM table のみが用意されているため、DCN 3.1.x ながら *GC_10_3_7* はそれと異なるメモリに対応すると読める。だが、メモリタイプが LPDDR5 でない場合に DDR4 の WM table が使われるようになっており、LPDDR5メモリはまず対応していると思われるが、DDR4 については仮のデータである可能性がある。  

最大画面出力数といった規模は基本 *Yellow Carp/Rembrandt APU* と同じであり、最大 4画面出力に対応しているものと思われる。  

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

LPDDR5メモリに対応する AMD APU には、最近になって [AMD Sabrina APU/SoC](/tags/sabrina) が Coreboot へのパッチで出てきている。  
しかし、今の所 *GC_10_3_7* と *AMD Sabrina* で結び付けられるのは LPDDR5メモリに対応する点だけであるため、断定はできない。  

 > 		+		if (ctx->dc_bios->integrated_info->memory_type == LpDdr5MemType) {
 > 		+			dcn316_bw_params.wm_table = lpddr5_wm_table;
 > 		+		} else {
 > 		+			dcn316_bw_params.wm_table = ddr4_wm_table;
 > 		+		}
 >
 > {{< quote >}} [[PATCH 5/6] drm/amd/display: Add DCN316 resource and SMU clock manager](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075448.html) {{< /quote >}}

[^vgh-dcn301]: [vg_clk_mgr.c « dcn301 « clk_mgr « dc « display « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/display/dc/clk_mgr/dcn301/vg_clk_mgr.c?h=next-20220216#n781)
[^yc-dcn31]: [dcn31_clk_mgr.c « dcn31 « clk_mgr « dc « display « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/display/dc/clk_mgr/dcn31/dcn31_clk_mgr.c?h=next-20220216#n687)

