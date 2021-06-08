---
title: "新たな RDNA 2 GPU、「Beige Goby」 をサポートするパッチが投稿される"
date: 2021-05-13T04:02:05+09:00
draft: false
tags: [ "Beige_Goby", "RDNA_2", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

Linux Kernel (amd-gfx) に、新たな RDNA 2 GPU、*Beige Goby* をサポートするための 49個のパッチが投稿された。  
それらパッチから垣間見える *Beige Goby* は小規模な GPU であり、だが *Beige Goby* を APU とするパッチは無く、他 RDNA 2 dGPU {{< comple >}} Sienna Cichlid/Navi21, Navy Flounder/Navi22, Dimgrey Cavefish/Navi23 {{< /comple >}} と共通するコード部が多いことから、*Beige Goby* は新たな RDNA 2 *dGPU* のコードネームだと思われる。  

`chipRevision` は `0x46` から始まり、*Navy Flounder* と *Dimgrey Cavefish* では過去 [GPUOpen-Drivers/pal: Platform Abstraction Library](https://github.com/GPUOpen-Drivers/pal) に投稿された内容から、前者は *Navi22* 、後者は *Navi23* という推測を立てられたが、*Beige Goby* のそれは含まれておらず、現時点では内部的な別のコードネーム (Navi2x) はまだ不明。  
{{< link >}} [新たな AMD RDNA 2 GPU、"Dimgrey Cavefish" & "VanGogh" | Coelacanth's Dream](/posts/2020/09/23/amd-vangogh-dimgrey_cavefish/) {{< /link >}}

{{< pindex >}}
 * [Beige Goby のキャッシュ構成](#cache)
 * [DCN 3.03](#dcn303)
 * [エンコード機能を持たない VCN3](#vcn3)
 * [AMD GPU cache info](#cache-info)
 * [コードネーム: Beige Goby](#codename)
{{< /pindex >}}

## Beige Goby のキャッシュ構成 {#cache}

以前、追加された *Vega10* 以降の AMD GPU のキャッシュ構成情報も一連のパッチに含まれている。  
{{< link >}} [AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/#dimgrey_cavefish) {{< /link >}}
以下引用部によれば、*Beige Goby* は L2キャッシュ 1024KiB (1MiB)、L3キャッシュ/Infinity Cache 16MiB を持ち、このキャッシュ容量は *Dimgrey Cavefish/Navi23* の半分のサイズとなる。  


 > 		+static struct kfd_gpu_cache_info beige_goby_cache_info[] = {
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
 > 		+		/* GL1 Data Cache per SA */
 > 		+		.cache_size = 128,
 > 		+		.cache_level = 1,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 8,
 > 		+	},
 > 		+	{
 > 		+		/* L2 Data Cache per GPU (Total Tex Cache) */
 > 		+		.cache_size = 1024,
 > 		+		.cache_level = 2,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 8,
 > 		+	},
 > 		+	{
 > 		+		/* L3 Data Cache per GPU */
 > 		+		.cache_size = 16*1024,
 > 		+		.cache_level = 3,
 > 		+		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 		+				CRAT_CACHE_FLAGS_DATA_CACHE |
 > 		+				CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 		+		.num_cu_shared = 8,
 > 		+	},
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH 18/49] drm/amdkfd: support beige_goby KFD](https://lists.freedesktop.org/archives/amd-gfx/2021-May/063493.html) {{< /quote >}}

また、*Beige Goby* は *Dimgrey Cavefish* 同様、メモリチャネルあたりに 4MiB の L3キャッシュ/Infinity Cache を持つことがディスプレイエンジン関連のコード部で示唆されており、だとすると *Beige Goby* のメモリバス幅は 64-bit と考えられ、dGPU としては非常に小規模なメモリ構成を採っている。  
それでいながら ShaderArray ごとに持つ GL1データキャッシュ (RDNA L1データキャッシュ) を共用する CU 数は 8基と *Dimgrey Cavefish* と同数であり、GFX部は *Dimgrey Cavefish* から ShaderEngine 数を減らしたような構成になっているのではないかと思われる。  

 > 		+	dc->caps.mall_size_per_mem_channel = 4;
 > 		+	/* total size = mall per channel * num channels * 1024 * 1024 */
 > 		+	dc->caps.mall_size_total = dc->caps.mall_size_per_mem_channel *
 > 		+				   dc->ctx->dc_bios->vram_info.num_chans *
 > 		+				   1024 * 1024;
 > 		+	dc->caps.cursor_cache_size =
 > 		+		dc->caps.max_cursor_size * dc->caps.max_cursor_size * 8;
 >
 > {{< quote >}} [[PATCH 42/49] drm/amd/display: Initial DC support for Beige Goby](https://lists.freedesktop.org/archives/amd-gfx/2021-May/063522.html) {{< /quote >}}

## DCN 3.03 {#dcn303}

ディスプレイエンジン部、DCN (Display Core Next) の IPバージョンは DCN3.03 となり、*Dimgrey Cavefish* の DCN3.02 からわずかにバージョンが進んだものとなる。  
最大画面出力数は意外、あるいは興味深いことに 2画面となっており、*Dimgrey Cavefish* の 5画面の半分以下、さらには APU である *VanGogh* は最大 4画面出力の対応であるから、APU よりもずっと少ない。  
dGPU としてはかなり割り切ったものになっているという印象を受ける。  

 > 		+static const struct resource_caps res_cap_dcn303 = {
 > 		+		.num_timing_generator = 2,
 > 		+		.num_opp = 2,
 > 		+		.num_video_plane = 2,
 > 		+		.num_audio = 2,
 > 		+		.num_stream_encoder = 2,
 > 		+		.num_dwb = 1,
 > 		+		.num_ddc = 2,
 > 		+		.num_vmid = 16,
 > 		+		.num_mpc_3dlut = 1,
 > 		+		.num_dsc = 2,
 > 		+};
 > {{< quote >}} [[PATCH 42/49] drm/amd/display: Initial DC support for Beige Goby](https://lists.freedesktop.org/archives/amd-gfx/2021-May/063522.html) {{< /quote >}}

## エンコード機能を持たない VCN3 {#vcn3}
割り切っているのはメモリ周りやディスプレイエンジン/コントローラー数だけでなく、他にも及ぶ。  

*Beige Goby* は他 RDNA 2 GPU 同様、マルチメディアエンジンとして AV1デコードのサポートといった機能を持つ VCN3 を搭載するが、パッチによれば *Beige Goby* は VCN3 1基を搭載し (*Sienna Cichlid* は 2基持ち、他は 1基)、そして *Beige Goby* の VCN3 にはエンコード機能を持たされていない、あるいは無効化されている。  
エンコード処理に関する部分のコードに `if (adev->asic_type != CHIP_BEIGE_GOBY)` という判定が追加されており、これにより *Beige Goby* ではその部分の処理がスキップされることになるため、やはりエンコード機能は持たないと考えられる。  

 > 		+	if (adev->asic_type == CHIP_BEIGE_GOBY) {
 > 		+		adev->vcn.num_vcn_inst = 1;
 > 		+		adev->vcn.num_enc_rings = 0;
 > 		+	}
 >
 > {{< quote >}} [[PATCH 24/49] drm/amdgpu: Enable VCN for Beige Goby](https://lists.freedesktop.org/archives/amd-gfx/2021-May/063498.html) {{< /quote >}}

また、PCIe や *XGMI/Infinity Fabric* などのバスを経由してデータを転送する際に用いられる SDMA (System DMA) エンジン/コントローラー数は 1基となり、これも CPU用に 2基持つ *Navy Flounder* や *Dimgrey Cavefish* よりも少なく、*VanGogh APU* と同じ 1基とされている。  

 > 		 	case CHIP_VANGOGH:
 > 		+	case CHIP_BEIGE_GOBY:
 > 		 		adev->sdma.num_instances = 1;
 > 		 		break;
 >
 > {{< quote >}} [[PATCH 13/49] drm/amd/amdgpu: add sdma ip block for beige_goby](https://lists.freedesktop.org/archives/amd-gfx/2021-May/063486.html) {{< /quote >}}

*Beige Goby* は RDNA 2 dGPU の中では最も規模の小さい GPU となり、メモリバス幅は 64-bit、マルチメディア機能も一部カットされている。また、最大画面出力数は 2画面と dGPU としては異例の少なさと言える。  
そうした *Beige Goby* のターゲット帯に考えられるのはモバイル向けだろう。モバイル向けであれば画面出力数の少なさはほとんど問題にならない。64-bit というメモリバス幅も GPU部のフットプリントを減らすのに貢献してくれる。  
また SMU (System Management Unit) に関連したコードに、**Smart Shift** のフラグが追加されており、タイミングとしては *Beige Goby* で活用することが期待される。  
**Smart Shift** はノートPC向けの機能であり、APU や dGPU を合わせた全体での消費電力を設定し、ワークロードに合わせて電力の割り振りを変えることで、限られた電力内で性能を最大限にするというもの。  

 > 		-#define FEATURE_SPARE_50_BIT            50
 > 		-#define FEATURE_SPARE_51_BIT            51
 > 		+#define FEATURE_GFX_PER_PART_VMIN_BIT   50
 > 		+#define FEATURE_SMART_SHIFT_BIT         51
 >
 > {{< quote >}} [[PATCH 27/49] drm/amd/pm: update smu11 driver interface header for beige_goby](https://lists.freedesktop.org/archives/amd-gfx/2021-May/063501.html) {{< /quote >}}

モバイル向け Radeon Pro には、*Polaris12* をベースにした CU 8基、メモリバス幅 64-bit、TBP 35W という仕様の **Radeon Pro WX2100 (Mobile)** が存在するが[^wx2100]、こうした低消費電力の GPU製品を *Beige Goby* ベースに置き換えていくのかもしれない。  

[^wx2100]: [Radeon™ Pro WX 2100 Graphics | Mobile Graphics | AMD](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-2100-mobile)

## コードネーム: Beige Goby {#codename}

*Beige Goby* というコードネームは他 RDNA 2 GPU 同様、色 + 魚 となっており、ベージュ (Beige) は明るい薄茶色、Goby はハゼの英名。  

ちなみに *Beige* は <span style="color: #f5f5dc;">こんな感じの色</span>。  
他 RDNA 2 GPU に使われている色はこんな感じ。  <span style="color: dimgrey;">Dimgrey</span>、<span style="color: #882D17">Sienna</span>、<span style="color: #000080">Navy</span>。  

## AMD GPU cache info {#cache-info}

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
| Sienna Cichlid | 128K | 4M | 128M |
| Navy Flounder | 128K | 3M | 96M |
| Dimgrey Cavefish | 128K | 2M | 32M |
| Beige Goby | 128K | 1M | 16M |

[^rv2-l2c]: [Raven2 の GPU L2キャッシュは 128KB | Coelacanth's Dream](/posts/2021/02/11/raven2-gpu-l2c-size/)
