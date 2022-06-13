---
title: "VanGogh_LITE (Mariner, Gopher) 向け AMDGPU ドライバーのソースコードを読んでみる"
date: 2022-06-14T01:47:03+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "VanGogh", "RDNA_2", "VanGogh_LITE" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Samsung Exynos 2200 SoC は GPU に *AMD RDNA 2 アーキテクチャ* ベースの *Xclipse 920* を採用している。  
そして Exynos 2200 SoC を搭載する Samsung Galaxy S22 Ultra は Android OS を採用しており、その関係で Linux Kernel と *Xclipse 920* 向けの AMDGPU ドライバーのソースコードが公開されているため読んでみた。  
ソースコードは [Samsung Open Source](https://opensource.samsung.com/uploadList) からダウンロードでき、*SM-S908B* で検索すると出てくる。  
今回参照したソースコードのバージョンは `S908BXXU2AVEH` となる。  

通常の AMDGPU ドライバーのソースコードは `drivers/gpu/drm/amd/` 下に置いてあるが、*Xclipse 920* 向けのドライバーは AMDGPU ドライバーをベースにしながら `drivers/gpu/samsung/{include, sgpu}/` 下に置かれている。  

## VanGogh Lite {#vgh_lite}
*Xclipse 920* の `ASIC Type/Family` は *VanGogh_LITE* とされている。  
`_LITE` は恐らくカスタム GPU に付けられるもので、過去には *Cyan Skilfish/Skillfish* に関連する GPU に *Navi10_LITE, Navi12_LITE* があった。  
{{< link >}} [Cyan Skilfishシリーズ、Navi10_LITE、Navi12_LITE に Robin に Ariel | Coelacanth's Dream](/posts/2022/02/05/cyan_skilfish-robin-apu/) {{< /link >}}
GC (Graphics, Compute) の IPバージョンは `10.4.0` とされる。  

*Xclipse 920* のフルスペックは不明だが、ターゲットに合わせた `eval_mode` を指定し、最大 WGP (CU) 数を制限しダウングレードするカーネルパラメータが追加されている。  
ShaderEngine 1基、RenderBackend 3基という設定は共通するが、`eval_mode=1` の場合は WGP 3基、`eval_mode=2` の場合は WGP 4基に再設定される。  

 > 		/**
 > 		 * DOC: eval_mode (int)
 > 		 * Specify evaluation target.
 > 		 * 0: disable evalation mode by default.
 > 		 * 1: evaluation target is GC10.4(3RB, 3WGP, 1SE)
 > 		 * 2: evaluation target is GC40.1(3RB, 4WGP, 1SE)
 > 		 * it can be extended for other projects
 > 		 */
 > 		int amdgpu_eval_mode = 0;
 > 		MODULE_PARM_DESC(eval_mode, "Evaluate gpu performace. 0:disable 1:GC10.4 2:GC40.1");
 > 		module_param_named(eval_mode, amdgpu_eval_mode, int, 0444);
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/amdgpu_drv.c {{< /quote >}}

また、`eval_mode=1` には `M0`、`eval_mode=2` には `M1` という識別子も付けられている。  

 > 		 enum eval_mode_config {
 > 			eval_mode_config_disabled           = 0,
 > 			eval_mode_config_gc_10_4            = 1, /* M0 */
 > 			eval_mode_config_gc_40_1            = 2, /* M1 */
 > 			eval_mode_config_end
 > 		};
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/amdgpu.h {{< /quote >}}

再設定を行う部分のコードにおいて、ShaderEngine あたりの ShaderArray 数は 1基とされており、これは *VanGogh* と同様の構成となる。  
*RDNA 1/2 アーキテクチャ* では基本 ShaderEngine あたり ShaderArray 2基という構成を採っており、片方のみを無効化する設定も可能だが、*VanGogh* では最初から ShaderArray 1基のみを搭載する構成を採っている。  
ShaderArray ごとに GL1キャッシュを持つため、1基の場合 GL1キャッシュが減ることになるが、その分ダイ面積を減らすことができる。  

 > 		static struct gc_down_config gc_cfg_tb[] = {
 > 			{
 > 				eval_mode_config_disabled, // config_mode
 > 				false,                        // change_adapter_info
 > 				0,                            // wgps_per_sa
 > 				0,                            // rbs_per_sa
 > 				0,                            // se_count
 > 				0,                            // sas_per_se
 > 				0,                            // packers_per_sc
 > 				0,                            // num_scs
 > 				false,                        // write_shader_array_config
 > 				false,                        // use_reset_lowest_vgt_fw
 > 				true,                         // force_gl1_miss
 > 			},
 > 			{
 > 				eval_mode_config_gc_10_4, // config_mode M0
 > 				true,                     // change_adapter_info
 > 				3,                        // wgps_per_sa
 > 				3,                        // rbs_per_sa
 > 				1,                        // se_count
 > 				1,                        // sas_per_se
 > 				2,                        // packers_per_sc
 > 				1,                        // num_scs
 > 				true,                     // write_shader_array_config
 > 				true,                     // use_reset_lowest_vgt_fw
 > 				false,                    // force_gl1_miss
 > 			},
 > 			{
 > 				eval_mode_config_gc_40_1, // config_mode M1
 > 				true,                     // change_adapter_info
 > 				4,                        // wgps_per_sa
 > 				3,                        // rbs_per_sa
 > 				1,                        // se_count
 > 				1,                        // sas_per_se
 > 				2,                        // packers_per_sc
 > 				1,                        // num_scs
 > 				true,                     // write_shader_array_config
 > 				true,                     // use_reset_lowest_vgt_fw
 > 				false,                    // force_gl1_miss
 > 			},
 > 		
 > 		};
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/gfx_v10_0.c {{< /quote >}}

*VanGogh_LITE* の ME (GFX Ring) と MEC (MicroEngine Compute, Compute Ring) の設定は、ME/MEC 共に 1基、ME/MEC あたり 1パイプ、パイプあたり 4キューとなっている。  
これは他 *RDNA 1/2 アーキテクチャ* ベースの GPU の設定と異なる。*RDNA 1/2 アーキテクチャ* では ME 1基、ME あたり 1パイプ、パイプあたり 1キュー、MEC は 2基、MEC あたり 4パイプ、*RDNA 1 アーキテクチャ* はパイプあたり 8キュー、*RDNA 2 アーキテクチャ* は 4キューとなっている。  
しかし最新の AMDGPU ドライバー内の設定と異なるというだけで、その意図までは読めない。またこのバージョンでは *Navi14* だけが ME 4基という設定になっているが、これも最新のバージョンとは異なる。  

 > 			case CHIP_VANGOGH_LITE:
 > 				/* 4 queues on pipe0 of me */
 > 				adev->gfx.me.num_me = 1;
 > 				adev->gfx.me.num_pipe_per_me = 1;
 > 				adev->gfx.me.num_queue_per_pipe = 4;
 > 				/* 4 queues on pipe0 of mec0 */
 > 				adev->gfx.mec.num_mec = 1;
 > 				adev->gfx.mec.num_pipe_per_mec = 1;
 > 				adev->gfx.mec.num_queue_per_pipe = 4;
 > 				break;
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/gfx_v10_0.c {{< /quote >}}

### MGFX0, MGFX1 {#mgfx}

`GRBM_CHIP_REVISION` の値から `MGFX0, MGFX1` のどちらかを判定するコードがあり、それによって返される AMDGPU Family が変わる。`MGFX0` の場合は `AMDGPU_FAMILY_VGH` が、`MGFX1` の場合は `AMDGPU_FAMILY_MGFX` となる。  
しかし `AMDGPU_FAMILY_MGFX` でファイルを検索すると、一部関数で早期リターンが行われており、`MGFX1` はサポート途中もしくは中止されたモデルではないかと思われる。  

 > 		/* GRBM_CHIP_REVISION definition
 > 		 * [7:0], minor ID
 > 		 * [15:8], Major/EVT ID
 > 		 * [23:16], Model ID, 0x60 is 'Premium'
 > 		 * [31:24], Generation ID
 > 		 */
 > 		#define MGFX_GEN(gcr) (((gcr) & 0xFF000000) >> 24)
 > 		#define MGFX_MOD(gcr) (((gcr) & 0xFF0000) >> 16)
 > 		#define MGFX_EVT(gcr) (((gcr) & 0xFF00) >> 8)
 > 		#define MGFX_MINOR(gcr) (((gcr) & 0xFF) >> 0)
 > 		#define AMDGPU_IS_MGFX0(gcr) ((MGFX_MOD(gcr) == 0x60) && (MGFX_GEN(gcr) == 0))
 > 		#define AMDGPU_IS_MGFX1(gcr) ((MGFX_MOD(gcr) == 0x60) && (MGFX_GEN(gcr) == 1))
 > 		#define AMDGPU_IS_EVT0(gcr) ((MGFX_MOD(gcr) == 0x60) && (MGFX_EVT(gcr) == 1))
 > 		#define AMDGPU_IS_EVT1(gcr) ((MGFX_MOD(gcr) == 0x60) && (MGFX_EVT(gcr) == 2) && (MGFX_MINOR(gcr) == 0))
 > 		#define AMDGPU_IS_EVT1_1(gcr) ((MGFX_MOD(gcr) == 0x60) && (MGFX_EVT(gcr) == 2) && (MGFX_MINOR(gcr) == 1))
 > 		#define AMDGPU_IS_MGFX0_EVT0(gcr) (AMDGPU_IS_MGFX0(gcr) && AMDGPU_IS_EVT0(gcr))
 > 		#define AMDGPU_IS_MGFX0_EVT1(gcr) (AMDGPU_IS_MGFX0(gcr) && AMDGPU_IS_EVT1(gcr))
 > 		#define AMDGPU_IS_MGFX0_EVT1_1(gcr) (AMDGPU_IS_MGFX0(gcr) && AMDGPU_IS_EVT1_1(gcr))
 > 		#define AMDGPU_IS_MGFX1_EVT0(gcr) (AMDGPU_IS_MGFX1(gcr) && AMDGPU_IS_EVT0(gcr))
 > 		#define AMDGPU_IS_MGFX1_EVT1(gcr) (AMDGPU_IS_MGFX1(gcr) && AMDGPU_IS_EVT1(gcr))
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/amdgpu.h {{< /quote >}}
 >
 > 		static uint32_t amdgpu_get_family_mgfx(struct amdgpu_device *adev)
 > 		{
 > 			if (AMDGPU_IS_MGFX1(adev->grbm_chip_rev))
 > 				return AMDGPU_FAMILY_MGFX;
 > 			else if (AMDGPU_IS_MGFX0(adev->grbm_chip_rev))
 > 				return AMDGPU_FAMILY_VGH;
 > 			else
 > 				return AMDGPU_FAMILY_UNKNOWN;
 > 		}
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/nv.c {{< /quote >}}

モデル名が `MGFX0` は *Xclipse 920*、`MGFX1` は *Xclipse 930* とされているが、Exynos 2200 の GPU は *Xclipse 920* であるため、`MGFX0` が主なサポート対象と考えられる。  
また Khronos Group の Conformant Products に登録されているのも *Xclipse 920* のみであり、*Xclipse 930* は登録されていない。[^conformant-products]  

[^conformant-products]: <https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_649>

 > 		static ssize_t gpu_model_show(struct kobject *kobj,
 > 					      struct kobj_attribute *attr, char *buf)
 > 		{
 > 			const char *product = NULL;
 > 			ssize_t count = 0;
 > 		
 > 			if (AMDGPU_IS_MGFX0(p_adev->grbm_chip_rev))
 > 				product = "920";
 > 			else if (AMDGPU_IS_MGFX1(p_adev->grbm_chip_rev))
 > 				product = "930";
 > 		
 > 			count = scnprintf(buf, PAGE_SIZE, "Samsung Xclipse %s\n", product);
 > 		
 > 			return count;
 > 		}
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/exynos_gpu_interface.c {{< /quote >}}


### Mariner, Gopher {#mariner_gopher}
`VANGOGH_LITE` とは別に *Mariner* と *Gopher* というコードネームが付けられている。  

 > 		enum amd_reset_method {
 > 			AMD_RESET_METHOD_LEGACY = 0,
 > 			AMD_RESET_METHOD_MODE0,
 > 			AMD_RESET_METHOD_MODE1,
 > 			AMD_RESET_METHOD_MODE2,
 > 			AMD_RESET_METHOD_BACO,
 > 			AMD_RESET_METHOD_MARINER_GOPHER
 > 		};
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/amdgpu.h {{< /quote >}}

以下引用部では *Mariner* の DeviceID (PCI ID) は `0x73A0` と *Navi21/Sienna Cichlid* と同じ ID が割り当てられているが、別の部分で `CHIP_VANGOGH_LITE` の場合は DeviceID を `0x73A0` に上書きするコードがあるため、実際の DeviceID ではないと思われる。  

 > 			/* Mariner */
 > 			{0x1002, 0x73A0, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_VANGOGH_LITE},
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/amdgpu_drv.c {{< /quote >}}
 >
 > 			/* TODO: assign PCI device ID */
 > 			if ((flags & AMD_ASIC_MASK) == CHIP_VANGOGH_LITE)
 > 				adev->device_id = 0x73A0;
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/amdgpu_drv.c {{< /quote >}}

*Mariner* と *Gopher* の違いだが、*Mariner* は `eval_mode=1, M0, mobile0, MGFX0`、*Gopher* は `eval_mode=2, M1, mobile1, MGFX1` に対応している記述が見られることから、有効 WGP 数が主な違いと思われる。  
*Mariner* は *Xclipse 920*、*Gopher* は *Xclipse 930* だとも言える。  

先に書いたように、*Gopher/Xclipse 930* はコードの内容や現時点で未発表なことを合わせるに、開発されていたが中止されたモデルである可能性が高い。  
仮の DeviceID ではあるが *Mariner/Xclipse 920* の ID は記述されているのに対し、*Gopher/Xclipse 930* のはない。  

 > 		int emu_soc_asic_init(struct amdgpu_device *adev)
 > 		{
 > 			if (sgpu_no_hw_access != 0)
 > 				return 0;
 > 			else if (AMDGPU_IS_MGFX0_EVT1(adev->grbm_chip_rev))
 > 				emu_mobile0_asic_init(adev);
 > 			else if (AMDGPU_IS_MGFX1(adev->grbm_chip_rev))
 > 				emu_mobile1_asic_init(adev);
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/emu_soc.c {{< /quote >}}

 > 		/* From the mobile1 Gopher build, cl72071 */
 > 		struct __reg_setting_m1 emu_mobile1_setting_cl72071[] = {
 > 			{0x000098F8, 0x00000142},
 > 			{0x00033440, 0x0000008D},
 > 			{0x00033640, 0x0000008D},
 > 			{0x00009508, 0x010B0000},
 > 			{0x00009510, 0x00000000},
 > 			{0x0003380C, 0xFFFFFFF3},
 > 			{0x00033884, 0xFFFFFFF3},
 > 			{0x00033894, 0x32103210},
 > 			{0x00033898, 0x32103210},
 > 			{0x00009F80, 0x00000500},
 > 		};
 >
 > {{< quote >}} drivers/gpu/drm/samsung/sgpu/emu_mobile1_asic_init.h {{< /quote >}}

{{< ref >}}
 * [Samsung Open Source](https://opensource.samsung.com/uploadList)
    * <https://opensource.samsung.com/uploadList?menuItem=mobile&classification1=mobile_phone>
 * [Exynos 2200 Mobile Processor | Specs | Samsung Semiconductor Global](https://semiconductor.samsung.com/processor/mobile-processor/exynos-2200/)
 * [Exynos 2200: Playtime is over | Samsung Semiconductor Global](https://semiconductor.samsung.com/newsroom/tech-blog/exynos-2200-playtime-is-over/)
{{< /ref >}}
