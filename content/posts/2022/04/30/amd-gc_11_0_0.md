---
title: "次世代 GPU IPブロックのサポートが進む AMDGPUドライバー ―― GFX11, GC 11.0, MES 11.0, IMU"
date: 2022-04-30T18:16:11+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "Linux_Kernel", "GFX11", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日の *SoC21* に続き、次世代 AMD GPU に採用されるであろう IPブロックのサポートに向けたパッチが、AMD の Alex Deucher 氏より amd-gfx メーリングリストに投稿されている。  
詳細は後述するが、AMDGPUドライバーでは IPブロックごとのサポートに移行したのに加え、GPU 情報がよりハードウェア側のバイナリ部分から読み取れるようになったことで、ソースコードから得られる情報は少なくなりつつある。  

 * [[PATCH 00/73] MES support](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078410.html)
 * [[PATCH 00/29] Add support for GC 11.0](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078485.html)
 * [[PATCH 01/11] drm/amdgpu: update latest IP discovery table structures](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078213.html)
 * [[PATCH 1/2] drm/amdgpu: add GC v11_0_0 family id](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078399.html)
    * [[PATCH 2/2] drm/amdgpu/discovery: Set GC family for GC 11.0 IP](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078398.html)

{{< pindex >}}
 * [GC 11.0](#gc_11)
    * [MES](#mes)
    * [IMU](#imu)
    * [Other](#other)
 * [IP discovery](#ip)
    * [Cache info](#cache_info)
{{< /pindex >}}

## GC 11.0 {#gc_11}

*GC 11.0/GFX11* の特徴として、GFX, SDMA, Compute の HWスケジューリングエンジンとして MES 11.0 (Micro Engine Scheduler) を採用すること、  
新たに電力管理ユニットとして、IMU を採用することが挙げられる。  

### MES {#mes}

 * [[PATCH 00/73] MES support](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078410.html)

MES は従来の HWS (HardWare Scheduler) に近い機能を持つ。  
用語の解説を挟むと、まず AMD GPU にはハードウェアブロックとして CP (Command Processor) があり、CP は多数のマイクロコントローラーで構成されている。その一部が MEC (MicroEngine Compute) であり、HWS は MEC内に位置する。  
MES は MEC を置き換えるものと思われるが、今の所パッチ内などで明言はされていない。  

 * [Core Driver Infrastructure — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/driver-core.html#gpu-hardware-structure)
 * [AMDGPU Glossary — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/amdgpu-glossary.html)

MES は *RDNA 1/GFX10* 世代から搭載されていたが、デフォルトでは無効化されていた。AMDGPUドライバー側のサポートが進んでいなかったことが理由の一つにあると思われる。  
今回のパッチでサポートが進んだものの、デフォルトでは無効化されている点は変更されていない。  

 > 		/**
 > 		 * DOC: mes (int)
 > 		 * Enable Micro Engine Scheduler. This is a new hw scheduling engine for gfx, sdma, and compute.
 > 		 * (0 = disabled (default), 1 = enabled)
 > 		 */
 > 		MODULE_PARM_DESC(mes,
 > 			"Enable Micro Engine Scheduler (0 = disabled (default), 1 = enabled)");
 > 		module_param_named(mes, amdgpu_mes, int, 0444);
 >
 > {{< quote >}} [linux/amdgpu_drv.c at f95af4a9236695caed24fe6401256bb974e8f2a7 · torvalds/linux](https://github.com/torvalds/linux/blob/f95af4a9236695caed24fe6401256bb974e8f2a7/drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c#L630-L637) {{< /quote >}}

### IMU {#imu}

IMU が何の略語なのかは触れられていないが、パッチのコメントから GFXコア部のみを対象に絞った電力管理ブロック/ユニットであることが考えられる。  
*RDNA 2/GFX10.3* 世代では、CU数が非対称な ShaderArray における電力効率を向上させる機能が追加されるなど、以前から GFXコア部を対象にした電力管理機能が強化されている節はあった。  
{{< link >}} [AMD RDNA 2 ノート　―― 非対称な ShaderArray、gpu_info、XSS 【2020/09/23】 | Coelacanth's Dream](/posts/2020/09/23/rdna_2-note-2020-09-23/#shader-array) {{< /link >}}

 > 		Add support to initialize imu for gfx v11.
 > 		IMU is a new power management block for
 > 		gfx which manages gfx power.
 > 
 > {{< quote >}} [[PATCH 19/29] drm/amdgpu: support imu for gfx11](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078500.html) {{< /quote >}}

### Other {#other}

*GC 11.0/GFX11* 世代では最大 ShaderEngine 8基 (ShaderArray 16基) の構成に対応し、一部無効化された CU を示すマスクも 16基分が想定されている。  
*RDNA 1/RDNA 2* 世代における同様の処理は `gfx_v10_0.c` に記述されているが、それらの世代では最大 ShaderEngine 4基 (ShaderArray 8基) までの対応となっている。[^gfx_v10_0]  

[^gfx_v10_0]: [linux/gfx_v10_0.c at 96f2b7a3571618a1c8aed694c9e668014c70898b · torvalds/linux](https://github.com/torvalds/linux/blob/96f2b7a3571618a1c8aed694c9e668014c70898b/drivers/gpu/drm/amd/amdgpu/gfx_v10_0.c#L9636-L9690)

 > 		+static int gfx_v11_0_get_cu_info(struct amdgpu_device *adev,
 > 		+				 struct amdgpu_cu_info *cu_info)
 > 		+{
 > 		+	int i, j, k, counter, active_cu_number = 0;
 > 		+	u32 mask, bitmap;
 > 		+	unsigned disable_masks[8 * 2];
 > 		+
 > 		+	if (!adev || !cu_info)
 > 		+		return -EINVAL;
 > 		+
 > 		+	amdgpu_gfx_parse_disable_cu(disable_masks, 8, 2);
 > 		+
 > 		+	mutex_lock(&adev->grbm_idx_mutex);
 > 		+	for (i = 0; i < adev->gfx.config.max_shader_engines; i++) {
 > 		+		for (j = 0; j < adev->gfx.config.max_sh_per_se; j++) {
 > 		+			mask = 1;
 > 		+			counter = 0;
 > 		+			gfx_v11_0_select_se_sh(adev, i, j, 0xffffffff);
 > 		+			if (i < 8 && j < 2)
 > 		+				gfx_v11_0_set_user_wgp_inactive_bitmap_per_sh(
 > 		+					adev, disable_masks[i * 2 + j]);
 > 		+			bitmap = gfx_v11_0_get_cu_active_bitmap_per_sh(adev);
 > 		+
 > 		+			/**
 > 		+			 * GFX11 could support more than 4 SEs, while the bitmap
 > 		+			 * in cu_info struct is 4x4 and ioctl interface struct
 > 		+			 * drm_amdgpu_info_device should keep stable.
 > 		+			 * So we use last two columns of bitmap to store cu mask for
 > 		+			 * SEs 4 to 7, the layout of the bitmap is as below:
 > 		+			 *    SE0: {SH0,SH1} --> {bitmap[0][0], bitmap[0][1]}
 > 		+			 *    SE1: {SH0,SH1} --> {bitmap[1][0], bitmap[1][1]}
 > 		+			 *    SE2: {SH0,SH1} --> {bitmap[2][0], bitmap[2][1]}
 > 		+			 *    SE3: {SH0,SH1} --> {bitmap[3][0], bitmap[3][1]}
 > 		+			 *    SE4: {SH0,SH1} --> {bitmap[0][2], bitmap[0][3]}
 > 		+			 *    SE5: {SH0,SH1} --> {bitmap[1][2], bitmap[1][3]}
 > 		+			 *    SE6: {SH0,SH1} --> {bitmap[2][2], bitmap[2][3]}
 > 		+			 *    SE7: {SH0,SH1} --> {bitmap[3][2], bitmap[3][3]}
 > 		+			 */
 > 		+			cu_info->bitmap[i % 4][j + (i / 4) * 2] = bitmap;
 > 		+
 > 		+			for (k = 0; k < adev->gfx.config.max_cu_per_sh; k++) {
 > 		+				if (bitmap & mask)
 > 		+					counter++;
 > 		+
 > 		+				mask <<= 1;
 > 		+			}
 > 		+			active_cu_number += counter;
 > 		+		}
 > 		+	}
 > 		+	gfx_v11_0_select_se_sh(adev, 0xffffffff, 0xffffffff, 0xffffffff);
 > 		+	mutex_unlock(&adev->grbm_idx_mutex);
 > 		+
 > 		+	cu_info->number = active_cu_number;
 > 		+	cu_info->simd_per_cu = NUM_SIMD_PER_CU;
 > 		+
 > 		+	return 0;
 > 		+}
 >
 > {{< quote >}} [[PATCH 21/29] drm/amdgpu: add init support for GFX11 (v2)](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078513.html) {{< /quote >}}

以下の引用部から、WGP 1基あたりの CU数は 2基という構成は *GC 11.0/GFX11* でも変わらないと考えられる。  

 > 		+static u32 gfx_v11_0_get_cu_active_bitmap_per_sh(struct amdgpu_device *adev)
 > 		+{
 > 		+	u32 wgp_idx, wgp_active_bitmap;
 > 		+	u32 cu_bitmap_per_wgp, cu_active_bitmap;
 > 		+
 > 		+	wgp_active_bitmap = gfx_v11_0_get_wgp_active_bitmap_per_sh(adev);
 > 		+	cu_active_bitmap = 0;
 > 		+
 > 		+	for (wgp_idx = 0; wgp_idx < 16; wgp_idx++) {
 > 		+		/* if there is one WGP enabled, it means 2 CUs will be enabled */
 > 		+		cu_bitmap_per_wgp = 3 << (2 * wgp_idx);
 > 		+		if (wgp_active_bitmap & (1 << wgp_idx))
 > 		+			cu_active_bitmap |= cu_bitmap_per_wgp;
 > 		+	}
 > 		+
 > 		+	return cu_active_bitmap;
 > 		+}
 > 		+
 >
 > {{< quote >}} [[PATCH 21/29] drm/amdgpu: add init support for GFX11 (v2)](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078513.html) {{< /quote >}}

CU内の SIMDユニットに関する記述もある。ここでは `navi10_enum.h` ファイルを include しているため、`NUM_SIMD_PER_CU` には *RDNA 1/RDNA 2* 世代と同じ `0x2` が入る。[^navi10_enum]  

[^navi10_enum]: [linux/navi10_enum.h at aa6158112645aae514982ad8d56df64428fcf203 · torvalds/linux](https://github.com/torvalds/linux/blob/aa6158112645aae514982ad8d56df64428fcf203/drivers/gpu/drm/amd/include/navi10_enum.h)

 > 		+	cu_info->number = active_cu_number;
 > 		+	cu_info->simd_per_cu = NUM_SIMD_PER_CU;
 > 		+
 > 		+	return 0;
 > 		+}
 >
 > {{< quote >}} [[PATCH 21/29] drm/amdgpu: add init support for GFX11 (v2)](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078513.html) {{< /quote >}}

## IP discovery {#ip}
AMD GPU では *Vega/GFX9* 世代から、電源投入時に PSP (Platform Security Processor) が内部的に GPU 情報を出力する IP discovery table に対応している。  
IP discovery table は世代ごとに拡張されており、AMDGPUドライバーでは最近になって、GPU あたりの GL2キャッシュ、ShaderArray あたりの GL1キャッシュ の数とサイズ等の情報が追加された `gc_info_v1_2` に対応するパッチが投稿されている。  

 * [[PATCH 01/11] drm/amdgpu: update latest IP discovery table structures](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078213.html)

 > 		+struct gc_info_v1_2 {
 > 		+	struct gpu_info_header header;
 > 		+	uint32_t gc_num_se;
 > 		+	uint32_t gc_num_wgp0_per_sa;
 > 		+	uint32_t gc_num_wgp1_per_sa;
 > 		+	uint32_t gc_num_rb_per_se;
 > 		+	uint32_t gc_num_gl2c;
 > 		+	uint32_t gc_num_gprs;
 > 		+	uint32_t gc_num_max_gs_thds;
 > 		+	uint32_t gc_gs_table_depth;
 > 		+	uint32_t gc_gsprim_buff_depth;
 > 		+	uint32_t gc_parameter_cache_depth;
 > 		+	uint32_t gc_double_offchip_lds_buffer;
 > 		+	uint32_t gc_wave_size;
 > 		+	uint32_t gc_max_waves_per_simd;
 > 		+	uint32_t gc_max_scratch_slots_per_cu;
 > 		+	uint32_t gc_lds_size;
 > 		+	uint32_t gc_num_sc_per_se;
 > 		+	uint32_t gc_num_sa_per_se;
 > 		+	uint32_t gc_num_packer_per_sc;
 > 		+	uint32_t gc_num_gl2a;
 > 		+	uint32_t gc_num_tcp_per_sa;
 > 		+	uint32_t gc_num_sdp_interface;
 > 		+	uint32_t gc_num_tcps;
 > 		+	uint32_t gc_num_tcp_per_wpg;
 > 		+	uint32_t gc_tcp_l1_size;
 > 		+	uint32_t gc_num_sqc_per_wgp;
 > 		+	uint32_t gc_l1_instruction_cache_size_per_sqc;
 > 		+	uint32_t gc_l1_data_cache_size_per_sqc;
 > 		+	uint32_t gc_gl1c_per_sa;
 > 		+	uint32_t gc_gl1c_size_per_instance;
 > 		+	uint32_t gc_gl2c_per_gpu;
 > 		+};
 >
 > {{< quote >}} [[PATCH 01/11] drm/amdgpu: update latest IP discovery table structures](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078213.html) {{< /quote >}}

*RDNA 2/GFX10.3* 世代ではキャッシュ情報がソースコード内にハードコードされていたが、`gc_info_v1_2` への対応によりその必要がなくなる。ソースコードから得られる AMD GPU の情報は少なくなるが、その方がドライバーとしては効率的である。  

その他にも、*MALL/Infinity Cache* と VCN の構成情報を取得することが可能になった。  
*MALL/Infinity Cache* では、メモリチャネルあたりのキャッシュサイズ (`mall_size_per_m`)、その倍のサイズ、もしくは半分のみを使用可能とするかのフラグ (`m_s_present, m_half_use`) といった情報が出力される。  
今後はメモリチャネル数は変えずに *MALL/Infinity Cache* のサイズを変更した SKU も可能になると思われるが、実際に計画されているかは当然不明。  

 > 		+struct mall_info_v1_0 {
 > 		+	struct mall_info_header header;
 > 		+	uint32_t mall_size_per_m;
 > 		+	uint32_t m_s_present;
 > 		+	uint32_t m_half_use;
 > 		+	uint32_t m_mall_config;
 > 		+	uint32_t reserved[5];
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH 01/11] drm/amdgpu: update latest IP discovery table structures](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078213.html) {{< /quote >}}
 >
 > 		+	case 1:
 > 		+		mall_size = 0;
 > 		+		mall_size_per_umc = le32_to_cpu(mall_info->v1.mall_size_per_m);
 > 		+		m_s_present = le32_to_cpu(mall_info->v1.m_s_present);
 > 		+		half_use = le32_to_cpu(mall_info->v1.m_half_use);
 > 		+		for (u = 0; u < adev->gmc.num_umc; u++) {
 > 		+			if (m_s_present & (1 << u))
 > 		+				mall_size += mall_size_per_umc * 2;
 > 		+			else if (half_use & (1 << u))
 > 		+				mall_size += mall_size_per_umc / 2;
 > 		+			else
 > 		+				mall_size += mall_size_per_umc;
 > 		+		}
 > 		+		adev->gmc.mall_size = mall_size;
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 07/11] drm/amdgpu/discovery: add a function to get the mall_size](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078220.html) {{< /quote >}}

VCN では、インスタンス数と、恐らくはどの対応コーデックが無効化されているか、といった情報が出力される。エンコーダーの有無等の情報は現時点では見当たらない。  

 > 		+#define VCN_INFO_TABLE_MAX_NUM_INSTANCES 4
 > 		+
 > 		+struct vcn_info_header {
 > 		+    uint32_t table_id; /* table ID */
 > 		+    uint16_t version_major; /* table version */
 > 		+    uint16_t version_minor; /* table version */
 > 		+    uint32_t size_bytes; /* size of the entire header+data in bytes */
 > 		+};
 > 		+
 > 		+struct vcn_instance_info_v1_0
 > 		+{
 > 		+	uint32_t instance_num; /* VCN IP instance number. 0 - VCN0; 1 - VCN1 etc*/
 > 		+	union _fuse_data {
 > 		+		struct {
 > 		+			uint32_t av1_disabled : 1;
 > 		+			uint32_t vp9_disabled : 1;
 > 		+			uint32_t hevc_disabled : 1;
 > 		+			uint32_t h264_disabled : 1;
 > 		+			uint32_t reserved : 28;
 > 		+		} bits;
 > 		+		uint32_t all_bits;
 > 		+	} fuse_data;
 > 		+	uint32_t reserved[2];
 > 		+};
 > 		+
 > 		+struct vcn_info_v1_0 {
 > 		+	struct vcn_info_header header;
 > 		+	uint32_t num_of_instances; /* number of entries used in instance_info below*/
 > 		+	struct vcn_instance_info_v1_0 instance_info[VCN_INFO_TABLE_MAX_NUM_INSTANCES];
 > 		+	uint32_t reserved[4];
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH 01/11] drm/amdgpu: update latest IP discovery table structures](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078213.html) {{< /quote >}}

### Cache info {#cache_info}

以下引用部は、IP discovery table からキャッシュ情報を読み取った場合の処理となる。  
まず、CU ごとに持つプライベートキャッシュ (TCP, Texture Cache per Pipe) だが、共有する CU数を示す `num_cu_shared` には WGP あたりの数を 2 で割っている。  
このことから WGP あたりの CU数は 2 で変わらないことが考えられる。  
ソースコード中では WGP ではなく、"WPG" になっているが、ここでは一旦誤字とする。  

 > 		+static int kfd_fill_gpu_cache_info_from_gfx_config(struct kfd_dev *kdev,
 > 		+						   struct kfd_gpu_cache_info *pcache_info)
 > 		+{
 > 		+	struct amdgpu_device *adev = kdev->adev;
 > 		+	int i = 0;
 > 		+
 > 		+	/* TCP L1 Cache per CU */
 > 		+	if (adev->gfx.config.gc_tcp_l1_size) {
 > 		+		pcache_info[i].cache_size = adev->gfx.config.gc_tcp_l1_size;
 > 		+		pcache_info[i].cache_level = 1;
 > 		+		pcache_info[i].flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+					CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+					CRAT_CACHE_FLAGS_SIMD_CACHE);
 > 		+		pcache_info[0].num_cu_shared = adev->gfx.config.gc_num_tcp_per_wpg / 2;
 > 		+		i++;
 > 		+	}
 > 		+	/* Scalar L1 Instruction Cache per SQC */
 > 		+	if (adev->gfx.config.gc_l1_instruction_cache_size_per_sqc) {
 > 		+		pcache_info[i].cache_size =
 > 		+			adev->gfx.config.gc_l1_instruction_cache_size_per_sqc;
 > 		+		pcache_info[i].cache_level = 1;
 > 		+		pcache_info[i].flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+					CRAT_CACHE_FLAGS_INST_CACHE |
 > 		+					CRAT_CACHE_FLAGS_SIMD_CACHE);
 > 		+		pcache_info[i].num_cu_shared = adev->gfx.config.gc_num_sqc_per_wgp * 2;
 > 		+		i++;
 > 		+	}
 > 		+	/* Scalar L1 Data Cache per SQC */
 > 		+	if (adev->gfx.config.gc_l1_data_cache_size_per_sqc) {
 > 		+		pcache_info[i].cache_size = adev->gfx.config.gc_l1_data_cache_size_per_sqc;
 > 		+		pcache_info[i].cache_level = 1;
 > 		+		pcache_info[i].flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+					CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+					CRAT_CACHE_FLAGS_SIMD_CACHE);
 > 		+		pcache_info[i].num_cu_shared = adev->gfx.config.gc_num_sqc_per_wgp * 2;
 > 		+		i++;
 > 		+	}
 >
 > {{< quote >}} [[PATCH 11/11] drm/amdkfd: add helper to generate cache info from gfx config](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078217.html) {{< /quote >}}

{{< ref >}}
 * [Core Driver Infrastructure — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/driver-core.html#gpu-hardware-structure)
 * [AMDGPU Glossary — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/amdgpu-glossary.html)
 * [AMD gem5 APU Simulator ISCA 2018 - AMD_gem5_APU_simulator_isca_2018_gem5_wiki.pdf](http://old.gem5.org/wiki/images/1/19/AMD_gem5_APU_simulator_isca_2018_gem5_wiki.pdf)
{{< /ref >}}
