---
title: "Linux Kernel に RDNA APU 「Cyan Skilfish」 をサポートするパッチが投稿される"
date: 2021-07-21T01:39:15+09:00
draft: false
tags: [ "gfx1013", "Cyan_Skilfish", "RDNA", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

amd-gfxメーリングリストにて、Linux Kernel における AMD GPUドライバーに新たな RDNA APU *Cyan Skilfish* をサポートするパッチが公開された。一連のパッチの数は 29個で構成されている。  

 * [[PATCH 00/29] Cyan Skillfish support](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066803.html)

最初に書いておくと、一連のパッチの中でも *SKILFISH* と *SKILLFISH* で表記が揺れており、とりあえず辞書を引いたところ、海水魚 アブラボウズの英名として *SKILFISH* が出てきたため、前者が正しいと考えここでは *SKILFISH* で統一する。  

{{< pindex >}}
 * ["RDNA" APU](#rdna-apu)
    * [ディスプレイ出力はサポートせず?](#display-output)
 * [Cyan Skilfish と Cyan Skilfish2](#v2)
 * [8-Core APU](#8-core)
{{< /pindex >}}

## "RDNA" APU {#rdna-apu}

*Cyan Skilfish* は APU であり、パッチでは PCI (Device) ID に `0x13FE` 1つが追加されている。  
一応 [The PCI ID Repository](https://pci-ids.ucw.cz/read/PC/1002) を確認したが、`0x13FE` はまだ登録されていなかった。  

 > 		+	/* CYAN_SKILLFISH */
 > 		+	{0x1002, 0x13FE, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_CYAN_SKILLFISH|AMD_IS_APU},
 > 		+
 >
 > {{< quote >}} [[PATCH 29/29] drm/amdgpu: add pci device id for cyan_skillfish](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066832.html) {{< /quote >}}

*Cyan Skilfish* は Navi10 (Navi12) と同じキャッシュ構成を採っているとしている。
*Navi10/Navi12/Cyan Skilfish* のキャッシュ構成は、CU ごとの L1ベクタキャッシュは 16KiB、WGP 1基 (CU 2基) で共有する L1命令キャッシュは 32KiB、同様の L1データキャッシュは 16KiB、  
ShaderArray ごとに持つ RDNA L1キャッシュ、オープンソースドライバー上では GL1データキャッシュとも呼ばれるものは 128KiB、そしてメモリサイドキャッシュである L2データキャッシュは 4096KiB (4MiB)、というものになる。  
{{< link >}}[AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/#vgh) {{< /link >}}

 > 		 	case CHIP_NAVI10:
 > 		 	case CHIP_NAVI12:
 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		 		pcache_info = navi10_cache_info;
 > 		 		num_of_cache_types = ARRAY_SIZE(navi10_cache_info);
 > 		 		break;
 >
 > {{< quote >}} [[PATCH 14/29] drm/amdkfd: enable cyan_skillfish KFD](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066817.html) {{< /quote >}}

 > 		 	static struct kfd_gpu_cache_info navi10_cache_info[] = {
 > 			{
 > 				/* TCP L1 Cache per CU */
 > 				.cache_size = 16,
 > 				.cache_level = 1,
 > 				.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 						CRAT_CACHE_FLAGS_DATA_CACHE |
 > 						CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 				.num_cu_shared = 1,
 > 			},
 > 			{
 > 				/* Scalar L1 Instruction Cache per SQC */
 > 				.cache_size = 32,
 > 				.cache_level = 1,
 > 				.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 						CRAT_CACHE_FLAGS_INST_CACHE |
 > 						CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 				.num_cu_shared = 2,
 > 			},
 > 			{
 > 				/* Scalar L1 Data Cache per SQC */
 > 				.cache_size = 16,
 > 				.cache_level = 1,
 > 				.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 						CRAT_CACHE_FLAGS_DATA_CACHE |
 > 						CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 				.num_cu_shared = 2,
 > 			},
 > 			{
 > 				/* GL1 Data Cache per SA */
 > 				.cache_size = 128,
 > 				.cache_level = 1,
 > 				.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 						CRAT_CACHE_FLAGS_DATA_CACHE |
 > 						CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 				.num_cu_shared = 10,
 > 			},
 > 			{
 > 				/* L2 Data Cache per GPU (Total Tex Cache) */
 > 				.cache_size = 4096,
 > 				.cache_level = 2,
 > 				.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 						CRAT_CACHE_FLAGS_DATA_CACHE |
 > 						CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 				.num_cu_shared = 10,
 > 			},
 > 		};
 >
 > {{< quote >}} [linux/kfd_crat.c at bf9d4e88c28b397ec6ec289c592ed41b552b8929 · torvalds/linux](https://github.com/torvalds/linux/blob/bf9d4e88c28b397ec6ec289c592ed41b552b8929/drivers/gpu/drm/amd/amdkfd/kfd_crat.c#L377) {{< /quote >}}

ソースコード中では、*RDNA 2/GFX10.3 GPU* も一部は *RDNA/GFX10.1 GPU* と同じレジスタ設定等を用いていることもあり、*RDNA* か *RDNA 2* かという判断が曖昧になる部分が出てくるが、  
GFXコア部に関する部分では *Navi10/Navi12/Navi14* 、*RDNA/GFX10.1 GPU* と同じものを使用しているため、*Cyan Skilfish* は *RDNA/GFX10.1* を GPU部に採用した APU だと考えられる。  


 > 		 	[CHIP_DIMGREY_CAVEFISH] = &gfx_v10_3_kfd2kgd,
 > 		 	[CHIP_BEIGE_GOBY] = &gfx_v10_3_kfd2kgd,
 > 		 	[CHIP_YELLOW_CARP] = &gfx_v10_3_kfd2kgd,
 > 		+	[CHIP_CYAN_SKILLFISH] = &gfx_v10_kfd2kgd,
 > 		 };
 >
 > {{< quote >}} [[PATCH 14/29] drm/amdkfd: enable cyan_skillfish KFD](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066817.html) {{< /quote >}}

 > 		 @@ -4748,6 +4759,7 @@ static int gfx_v10_0_sw_init(void *handle)
 > 		 	case CHIP_NAVI10:
 > 		 	case CHIP_NAVI14:
 > 		 	case CHIP_NAVI12:
 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		 		adev->gfx.me.num_me = 1;
 > 		 		adev->gfx.me.num_pipe_per_me = 1;
 > 		 		adev->gfx.me.num_queue_per_pipe = 1;
 > 		@@ -7712,6 +7724,7 @@ static int gfx_v10_0_early_init(void *handle)
 > 		 	case CHIP_NAVI10:
 > 		 	case CHIP_NAVI14:
 > 		 	case CHIP_NAVI12:
 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		 		adev->gfx.num_gfx_rings = GFX10_NUM_GFX_RINGS_NV1X;
 > 		 		break;
 >
 > {{< quote >}} [[PATCH 11/29] drm/amdgpu: add cyan_skillfish support in gfx v10](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066814.html) {{< /quote >}}

そして興味深いことに、 `amd_asci_type` 列挙体に *Cyan Skilfish* を追加する際、最近になって追加された [Yellow Carp APU](/tags/yellow_carp) の後ではなく、*Navi10* と *Navi14* の間に入るように追加されている。  

若干直感的ではあるが、以前 LLVM に追加された [GPUID: gfx1013](/tags/gfx1013) が示す APU が *Cyan Skilfish* だと自分は考えている。  
*Navi10* と *Navi12/Navi14* は同じ *RDNA/GFX10.1* 世代であっても対応する機能や命令に違いが存在し、基本的に *Navi12/Navi14* が多くの機能、命令をサポートしている。  
*Navi12/Navi14* が対応する、RB (Render Backend) 部における DCC (Delta Color Compression) 機能やドット積命令の一部は *Navi10* ではサポートされていない。  
{{< link >}} [RadeonSI、RADV が AMD Navy Flounder をサポート | Coelacanth's Dream](/posts/2020/07/29/amd-navy_flounder-umd-patch/#navi10-rb) {{< /link >}}
*gfx1013* は命令範囲について *Navi10/gfx1010* をベースにしており、ドット積命令の一部に対応しない点も引き継いでいる。  
{{< link >}} [RDNA/GFX10.1世代に新たな GPUID、gfx1013? | Coelacanth's Dream](/posts/2021/06/04/llvm-gpuid-gfx1013/) {{< /link >}}
{{< link >}} [GPUID: gfx1013 は RDNA/Navi10ベースの APU で、レイトレーシング命令をサポートしている? | Coelacanth's Dream](/posts/2021/06/06/gfx1013-apu-rt/) {{< /link >}}

GPU ASIC の開発時期を反映したか、関連して *Navi10* をベースにしたとするならば、わざわざ *Navi10* と *Navi14* の間に *Cyan Skilfish* を置いたことがしっくりいく。  

 > 		 	CHIP_NAVI10,	/* 26 */
 > 		-	CHIP_NAVI14,	/* 27 */
 > 		-	CHIP_NAVI12,	/* 28 */
 > 		-	CHIP_SIENNA_CICHLID,	/* 29 */
 > 		-	CHIP_NAVY_FLOUNDER,	/* 30 */
 > 		-	CHIP_VANGOGH,	/* 31 */
 > 		-	CHIP_DIMGREY_CAVEFISH,	/* 32 */
 > 		-	CHIP_BEIGE_GOBY,	/* 33 */
 > 		-	CHIP_YELLOW_CARP,	/* 34 */
 > 		+	CHIP_CYAN_SKILLFISH,	/* 27 */
 > 		+	CHIP_NAVI14,	/* 28 */
 > 		+	CHIP_NAVI12,	/* 29 */
 > 		+	CHIP_SIENNA_CICHLID,	/* 30 */
 > 		+	CHIP_NAVY_FLOUNDER,	/* 31 */
 > 		+	CHIP_VANGOGH,	/* 32 */
 > 		+	CHIP_DIMGREY_CAVEFISH,	/* 33 */
 > 		+	CHIP_BEIGE_GOBY,	/* 34 */
 > 		+	CHIP_YELLOW_CARP,	/* 35 */
 > 		 	CHIP_LAST,
 > 		 };
 >
 > {{< quote >}} [[PATCH 03/29] drm/amdgpu: add cyan_skillfish asic type](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066805.html) {{< /quote >}}

### ディスプレイ出力はサポートせず? {#display-output}

それと *Cyan Skilfish* は APU だが、今回のパッチではディスプレイコントローラー/エンジン部のサポートが追加されていない。  
それだけでなくディスプレイコントローラー/エンジン部の IP に関しても仮想的なものを用いるように記述されており、ディスプレイ出力をサポートした *Cyan Skilfish* を想定していないか、あるいは本当に元から *Cyan Skilfish* にディスプレイ出力とコントローラー/エンジンが実装されていないか、だと思われる。  

 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		+		amdgpu_device_ip_block_add(adev, &nv_common_ip_block);
 > 		+                amdgpu_device_ip_block_add(adev, &gmc_v10_0_ip_block);
 > 		+                amdgpu_device_ip_block_add(adev, &navi10_ih_ip_block);
 > 		+                if (adev->enable_virtual_display || amdgpu_sriov_vf(adev))
 > 		+                        amdgpu_device_ip_block_add(adev, &dce_virtual_ip_block);
 > 		+                amdgpu_device_ip_block_add(adev, &gfx_v10_0_ip_block);
 > 		+                amdgpu_device_ip_block_add(adev, &sdma_v5_0_ip_block);
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 06/29] drm/amdgpu: set ip blocks for cyan_skillfish](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066808.html) {{< /quote >}}

## Cyan Skilfish と Cyan Skilfish2 {#v2}
*Cyan Skilfish* にはもう 1つ、*Cyan Skilfish2* が存在することが示されており、読み込むファームウェアはその 2つで別に用意されることになっている。  

 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		+		if (adev->apu_flags & AMD_APU_IS_CYAN_SKILLFISH2)
 > 		+			chip_name = "cyan_skillfish2";
 > 		+		else
 > 		+			chip_name = "cyan_skillfish";
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 07/29] drm/amdgpu: add cp/rlc fw loading support for cyan_skillfish](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066809.html) {{< /quote >}}

ただ、ここで `chip_name` を `"cyan_skillfish2"` と判定する部分に使われているフラグは PCI ID が `0x13FE` の時にセットされるようになっている。上で触れたように *Cyan Skilfish* の ID として追加されたのはその `0x13FE` のみである。  
今回のパッチで想定しているのは *Cyan Skilfish2* のみで、恐らくそうでない *Cyan Skilfish* は開発中に何かしらのバグがあった GPU ASIC ではないかと思われる。  

 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		+		if (adev->pdev->device == 0x13FE)
 > 		+			adev->apu_flags |= AMD_APU_IS_CYAN_SKILLFISH2;
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 03/29] drm/amdgpu: add cyan_skillfish asic type](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066805.html) {{< /quote >}}

また、*Cyan Skilfish/2* で処理が分けられているのは主に PSP (Platform Security Processor) に関する部分であり、開発中に問題が発生してしまったのも PSP部ではないかと考えられる。  
*Cyan Skilfish* は PSP の IPバージョンが異なり、PSP v11.0.8 を採用している。  
他の *RDNA / RDNA 2 APU/GPU* は PSP v11.0 を採用しているため、新規に開発したとすれば問題が起きやすいことが想像できる。  

 > 		 	case CHIP_CYAN_SKILLFISH:
 > 		+		if (adev->apu_flags & AMD_APU_IS_CYAN_SKILLFISH2 &&
 > 		+		    load_type > 1)
 > 		+			return AMDGPU_FW_LOAD_PSP;
 > 		 		return AMDGPU_FW_LOAD_DIRECT;
 >
 > {{< quote >}} [[PATCH 21/29] drm/amdgpu: use direct loading by default for cyan_skillfish2](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066822.html) {{< /quote >}}

## 8-Core APU {#8-core}

*Cyan Skilfish* は SMU (System Management Unit) に関する記述から、CPU部 8-Core を持つ APU だと考えられる。  
8-Core APU には *Xbox Series S|X SoC* や *PS5 SoC* 、後はまあ *AMD 4700S* とか色々存在するが、`0x13FE` は [The PCI ID Repository](https://pci-ids.ucw.cz/read/PC/1002) にもまだ追加されていなかった ID であり、そして AMD が公式的に提供している情報では *Cyan Skilfish* についてそれ以上の判断はできないため、とりあえず自分が触れるのもここまでにしておきたい。  

 > 		+#define NUMBER_OF_PSTATES		8
 > 		+#define NUMBER_OF_CORES			8
 >
 > {{< quote >}} [[PATCH 24/29] drm/amdgpu: add smu interface header for cyan_skilfish](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066825.html) {{< /quote >}}
 >
 > 		+typedef enum {
 > 		+	CPU_CORE0 = 0,
 > 		+	CPU_CORE1,
 > 		+	CPU_CORE2,
 > 		+	CPU_CORE3,
 > 		+	CPU_CORE4,
 > 		+	CPU_CORE5,
 > 		+	CPU_CORE6,
 > 		+	CPU_CORE7
 > 		+} CORE_ID_e;
 >
 > {{< quote >}} [[PATCH 24/29] drm/amdgpu: add smu interface header for cyan_skilfish](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066825.html) {{< /quote >}}
