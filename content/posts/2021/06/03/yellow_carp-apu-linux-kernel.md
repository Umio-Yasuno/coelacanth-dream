---
title: "RDNA 2 APU 「Yellow Carp」 をサポートするパッチが Linux Kernel に投稿される　―― DCN3.1 / VanGogh より大きいキャッシュ"
date: 2021-06-03T03:59:37+09:00
draft: false
tags: [ "Yellow_Carp", "RDNA_2", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

先日に [Marek Olšák](https://github.com/marekolsak) 氏よりオープンソース OpenGL ドライバー **RadeonSI** に向けて投稿されたパッチから、[VanGogh](/tags/vangogh) に続く RDNA 2 APU であることが判明した *Yellow Carp* だが、  
{{< link >}} [Yellow Carp は VanGogh に続く RDNA 2 APU に | Coelacanth's Dream](/posts/2021/05/19/radeonsi-beige_goby-yellow_carp/) {{< /link >}}
Linux Kernel (amd-gfx) メーリングリスト、KMD (Kernel Mode Driver) 側にも *Yellow Carp* をサポートするための新たなパッチ 89個が投稿され、そのさらなる詳細が明らかにされた。  

 * [[PATCH 00/89] Add initial support for Yellow Carp](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064672.html)

*Yellow Carp* というコードネームは 2020/12 に投稿された SMUv13 (System Management Unit) に関するパッチが初出であり、そのパッチ内容から APU とされている。  
{{< link >}} [新たな AMD APU、"Yellow Carp"　―― SMU は v13 に | Coelacanth's Dream](/posts/2020/12/08/amd-yellow-carp-apu/) {{< /link >}}

 >        +void yellow_carp_set_ppt_funcs(struct smu_context *smu)
 >        +{
 >        +	smu->ppt_funcs = &yellow_carp_ppt_funcs;
 >        +	smu->message_map = yellow_carp_message_map;
 >        +	smu->table_map = yellow_carp_table_map;
 >        +	smu->is_apu = true;
 >        +}
 >
 > {{< quote >}} [[PATCH 6/8] drm/amd/pm: add yellow_carp_ppt implementation(V3)](https://lists.freedesktop.org/archives/amd-gfx/2020-December/057163.html) {{< /quote >}}

Linux Kernel (amd-gfx) にサポートを追加する作業自体は以前より行われていたが、最初のパッチから半年して外部にオープンソースとして公開できる段階となったのだろう。  

{{< pindex >}}
 * [Yellow Carp](#yc)
    * [DCN 3.1](#dcn3_1)
    * [Yellow Carp APU の GPUキャッシュ構成](#yc-cache)
{{< /pindex >}}

## Yellow Carp {#yc}
*Yellow Carp* は *RDNA 2* GPU として見ればそう大きく異なる部分は見受けられず、*RDNA 2* APU として見ても *Van Gogh* と共通している点が多く、コード部も同様のものを利用している。  
マルチメディアエンジンには VCN 3.0、JPEG 3.0 を搭載しているが、これは *RDNA 2 dGPU* {{< comple >}} Sienna Cichlid, Navy Flounder, Dimgrey Cavefish, Beige Goby {{< /comple >}}、*VanGogh APU* と同じ IPバージョンだ。  

 > 		+
 > 		+		amdgpu_device_ip_block_add(adev, &vcn_v3_0_ip_block);
 > 		+		amdgpu_device_ip_block_add(adev, &jpeg_v3_0_ip_block);
 >
 > {{< quote >}} [[PATCH 43/89] drm/amdgpu: enable vcn/jpeg on yellow carp](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064702.html) {{< /quote >}}

サポートするメモリについても、ディスプレイエンジン部のコードに DDR4メモリと LPDDR5メモリのテーブルが用意されており、少なくともそれらをサポートしていると考えられるが、ここも *VanGogh APU* と同じ点となる。  

 > 		+		if (ctx->dc_bios->integrated_info->memory_type == LpDdr5MemType) {
 > 		+			dcn31_bw_params.wm_table = lpddr5_wm_table;
 > 		+		} else {
 > 		+			dcn31_bw_params.wm_table = ddr4_wm_table;
 > 		+		}
 >
 > {{< quote >}} [[PATCH 71/89] drm/amd/display: Add DCN3.1 clock manager support](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064731.html) {{< /quote >}}

だが *Yellow Carp APU* に真新しい部分、新しい機能が無い訳ではなく、 *Yellow Carp* に採用されるディスプレイエンジンIP、DCN 3.1 と *VanGogh APU* とは異なる GPUキャッシュ構成が最大の特徴だと個人的に考えている。  

### DCN 3.1 {#dcn3_1}
*RDNA 2 dGPU* と *VanGogh* はディスプレイエンジンに DCN 3.0 系列を採用し、それらの違いは最大画面出力数数といくつかの機能のみだった。  
*Yellow Carp APU* ではそれよりも進んだ DCN 3.1 を採用する。  

DCN 3.1 では新たに ZState、あるいは z9/10 と呼ぶクロックステートが追加されている。パッチのコメントによれば他にもクロックに関する機能が DCN 3.0 から追加されており、クロックを状況に合わせて柔軟に切り替えることでモバイル向けをメインとする APU では重要となる、さらなる省電力化を狙っているものと思われる。  

 > 		
 > 		Adds support for clock requests for the various parts of the DCN3.1 IP
 > 		and the interfaces and definitions for sending messages to SMU/PMFW.
 > 		
 > 		Includes new support for z9/10, detecting SMU timeout and p-state
 > 		support enablement.
 >
 > {{< quote >}} [[PATCH 71/89] drm/amd/display: Add DCN3.1 clock manager support](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064731.html) {{< /quote >}}

また、最大画面出力数は 4画面とされ、この数は *Raven/Picasso/Renoir APU* 、*VanGogh APU* と同じ。  

 > 		+#if defined(CONFIG_DRM_AMD_DC_DCN3_1)
 > 		+	case CHIP_YELLOW_CARP:
 > 		+		adev->mode_info.num_crtc = 4;
 > 		+		adev->mode_info.num_hpd = 4;
 > 		+		adev->mode_info.num_dig = 4;
 > 		+		break;
 > 		+#endif
 >
 > {{< quote >}} [[PATCH 87/89] drm/amd/display: Add DCN3.1 Yellow Carp support to DM](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064743.html) {{< /quote >}}

### Yellow Carp APU の GPUキャッシュ構成 {#yc-cache}

ROCmソフトウェアを動作させるためのドライバー、KFD (Kernel Fusion Driver) 部には他 dGPU/APU 同様に *Yellow Carp APU* のキャッシュ構成を追加するパッチが投稿されている。  
{{< link >}} [AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/) {{< /link >}}

 > 		+static struct kfd_gpu_cache_info yellow_carp_cache_info[] = {
 > 		+	{
 > 		+		/* TCP L1 Cache per CU */
 > 		+		.cache_size = 16,
 > 		+		.cache_level = 1,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 1,
 > 		+	},
 > 		+	{
 > 		+		/* Scalar L1 Instruction Cache per SQC */
 > 		+		.cache_size = 32,
 > 		+		.cache_level = 1,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_INST_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 2,
 > 		+	},
 > 		+	{
 > 		+		/* Scalar L1 Data Cache per SQC */
 > 		+		.cache_size = 16,
 > 		+		.cache_level = 1,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 2,
 > 		+	},
 > 		+	{
 >
 > {{< quote >}} [[PATCH 13/89] drm/amdkfd: add yellow carp KFD support](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064671.html) {{< /quote >}}

L1キャッシュに関しては他と同じ構成であり、CU ごとに 16KiB の L1ベクタキャッシュを持ち、WGP 1基 (CU 2基) で L1スカラキャッシュ 16 KiB、L1命令キャッシュ 32 KiB を共有する。  
そして *RDNA 2 アーキテクチャ* でありながら L3/Infiniy Cache を搭載しないが、これは *VanGogh* と同様だ。*RDNA 2 APU* では L3/Infinity Cache を搭載しないのが通常の構成となるのかもしれない。  

だが、ShaderArray ごとに存在する 128 KiB の GL1データキャッシュ (*RDNA/2 アーキテクチャ* における L1キャッシュ) と L2キャッシュを共有する CU数、それと L2キャッシュサイズが *VanGogh* とは異なっている。  

 > ##### Yellow Carp cache config
 > 		+		/* GL1 Data Cache per SA */
 > 		+		.cache_size = 128,
 > 		+		.cache_level = 1,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 6,
 > 		+	},
 > 		+	{
 > 		+		/* L2 Data Cache per GPU (Total Tex Cache) */
 > 		+		.cache_size = 2048,
 > 		+		.cache_level = 2,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 6,
 > 		+	},
 > 		+};
 >
 > {{< quote >}} [[PATCH 13/89] drm/amdkfd: add yellow carp KFD support](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064671.html) {{< /quote >}}

 > ##### VanGogh cache config
 >        +		/* GL1 Data Cache per SA */
 >        +		.cache_size = 128,
 >        +		.cache_level = 1,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 8,
 >        +	},
 >        +	{
 >        +		/* L2 Data Cache per GPU (Total Tex Cache) */
 >        +		.cache_size = 1024,
 >        +		.cache_level = 2,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 8,
 >        +	},
 >        +};
 >
 > {{< quote >}} [[PATCH] drm/amdkfd: Update L1 and add L2/3 cache information](https://lists.freedesktop.org/archives/amd-gfx/2021-March/061392.html) {{< /quote >}}

*VanGogh* では ShaderArray あたり CU 8基を持ち、L2キャッシュは 1 MiB (1024 KiB) の構成を取っていたが、*Yellow Carp* では ShaderArray あたり CU 6基、L2キャッシュは 2 MiB (2048 KiB) の構成を取っている。  
ShaderArray の規模は *VanGogh* よりは小さくなっているが、あくまでもGFXコア部のクラスタであり単位となる ShaderArray の規模であり、全体としてはどちらが大きいかはまだ不明だ。  
GPU全体で使用する L2キャッシュが *VanGogh* より大きいサイズであることから、*Yellow Carp* の GPU部が *VanGogh* よりも全体では大きいか、広いメモリバス幅である可能性は考えられる。  

また APU が持つ GPU L2キャッシュとしては *GFX9/Vega* 世代の GPU を搭載する *Raven/Picasso/Renoir/Lucienne/Cezanne APU* が 1 MiB だったことを思えば 2 MiB 搭載することは大きな変化であり、*Yellow Carp* では GL1データキャッシュと合わせ、大幅にキャッシュ部分が強化されたことによる性能と電力効率の向上に期待ができる。  
*RDNA 2 アーキテクチャ* を採用していることもそれらの向上に大きく貢献してくれることだろう。  

| AMD GPU<br>cache info | RDNA L1 (GL1) | L2 (TCC) | L3/Infinity Cache (MALL) |
| :-- | :--: | :--: | :--: |
| Raven/Picasso<br>Renoir/Lucienne/Cezanne | - | 1M | - |
| Raven2 | - | 128K[^rv2-l2c] | - |
| Vega20 | - | 4M | - |
| Arcturus | - | 8M? | - |
| Aldebaran | - | 8M | - |
| Navi10/Navi12 | 128K | 4M | - |
| Navi14 | 128K | 2M | - |
| VanGogh | 128K | 1M | - |
| *Yellow Carp* | 128K | 2M | - |
| Sienna Cichlid | 128K | 4M | 128M |
| Navy Flounder | 128K | 3M | 96M |
| Dimgrey Cavefish | 128K | 2M | 32M |
| Beige Goby | 128K | 1M | 16M |

[^rv2-l2c]: [Raven2 の GPU L2キャッシュは 128KB | Coelacanth's Dream](/posts/2021/02/11/raven2-gpu-l2c-size/)
