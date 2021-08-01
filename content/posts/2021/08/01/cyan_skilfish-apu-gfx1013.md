---
title: "HWレイトレーシングをサポートする RDNA APU 「Cyan Skilfish (gfx1013)」"
date: 2021-08-01T01:36:48+09:00
draft: false
tags: [ "gfx1013", "RDNA", "Cyan_Skilfish", "AMD_4700S" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

2021/06 に *GPUID: gfx1013* のサポート追加する LLVM へのパッチが公開されてから、関連する情報を追い続けてきたが、ある程度集まってくると同時に断片化してきたと感じてきたため、集約し整理した記事を作ることにした。  

ここでも最初に一応書くが、AMD GPU ドライバーのコード中では *SKILLFISH* で統一されているが、パッチタイトルの一部では *SKILFISH* となっていること、アブラボウズの英名として *SKILFISH*  があることを踏まえて、ここでは *SKILFISH* で統一している。  
コード内では統一されているため問題は起きないし、amd-gfx メーリングリスト内で指摘するメッセージも無いため、どちらが正しいとも言い切れず、またそこまでこだわるような事柄でもない。  

## gfx1013 {#gfx1013}

まず出てきたのは *GPUID: gfx1013* であり、GPUID はその割り当てられた GPU の IPバージョンを表現しており、`gfx{major}_{minor}_{stepping}` のフォーマットとなっている。  
*RDNA アーキテクチャ* は GFX10.1 (gfx101x) [Major: 10, Minor: 1] を、*RDNA 2 アーキテクチャ* は GFX10.3 (gfx103x) [Major: 10, Minor: 3] を基本に Stepping で同アーキテクチャを採用する AMD GPU を区別している。  
*gfx1013* の場合は [Major: 10, Minor: 1, Stepping: 3] となり、GPUID (GFX IP) 的には *RDNA アーキテクチャ* だと言える。  
{{< link >}} [RDNA/GFX10.1世代に新たな GPUID、gfx1013? | Coelacanth's Dream](/posts/2021/06/04/llvm-gpuid-gfx1013/) {{< /link >}}

LLVM へのパッチから分かる *gfx1013* の特徴としては、*RDNA APU* であること、対応命令的には *Navi10 (gfx1010)* をベースにしていること、*RDNA 2 アーキテクチャ* がサポートしているレイトレーシング命令をサポートしていることが挙げられる。  
{{< link >}} [GPUID: gfx1013 は RDNA/Navi10ベースの APU で、レイトレーシング命令をサポートしている? | Coelacanth's Dream](/posts/2021/06/06/gfx1013-apu-rt/) {{< /link >}}

*Navi10 (gfx1010)* は *Navi12 (gfx1011)/Navi14 (gfx1012)* がサポートしている一部ドット積命令 (`dot[1,2,5-7]-insts`)をサポートせず、*gfx1013* も同様にそれら命令をサポートしていないことになっている。  

 > 		    case GK_GFX1012:
 > 		    case GK_GFX1011:
 > 		      Features["dot1-insts"] = true;
 > 		      Features["dot2-insts"] = true;
 > 		      Features["dot5-insts"] = true;
 > 		      Features["dot6-insts"] = true;
 > 		      Features["dot7-insts"] = true;
 > 		      LLVM_FALLTHROUGH;
 > 		    case GK_GFX1013:
 > 		    case GK_GFX1010:
 > 		      Features["dl-insts"] = true;
 > 		      Features["ci-insts"] = true;
 > 		      Features["flat-address-space"] = true;
 > 		      Features["16-bit-insts"] = true;
 > 		      Features["dpp"] = true;
 > 		      Features["gfx8-insts"] = true;
 > 		      Features["gfx9-insts"] = true;
 > 		      Features["gfx10-insts"] = true;
 > 		      Features["s-memrealtime"] = true;
 > 		      Features["s-memtime-inst"] = true;
 > 		      break;
 >
 > {{< quote >}} [llvm-project/AMDGPU.cpp at aaba37187fda7f5a7fdc4c1e6129cbaaa1bbf709 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/aaba37187fda7f5a7fdc4c1e6129cbaaa1bbf709/clang/lib/Basic/Targets/AMDGPU.cpp#L210-L230) {{< /quote >}}

パッチでは、レイトレーシング命令 (`image_bvh_intersect_ray/image_bvh64_intersect_ray`) と、サンプラーを用いずにコンポーネントから最大 4個のサンプルをロードする `image_msaa_load` サポートの有無を判定する `FeatureGFX10_AEncoding` が追加されており、それを *gfx1013* と *RDNA 2 アーキテクチャ/gfx103x* が持つとしている。  

 > 		def FeatureISAVersion10_1_3 : FeatureSet<
 > 		  !listconcat(FeatureGroup.GFX10_1_Bugs,
 > 		    [FeatureGFX10,
 > 		     FeatureGFX10_AEncoding,
 > 		     FeatureLDSBankCount32,
 > 		     FeatureDLInsts,
 > 		     FeatureNSAEncoding,
 > 		     FeatureWavefrontSize32,
 > 		     FeatureScalarStores,
 > 		     FeatureScalarAtomics,
 > 		     FeatureScalarFlatScratchInsts,
 > 		     FeatureGetWaveIdInst,
 > 		     FeatureMadMacF32Insts,
 > 		     FeatureDsSrc2Insts,
 > 		     FeatureLdsMisalignedBug,
 > 		     FeatureSupportsXNACK])>;
 >
 > {{< quote >}} [llvm-project/AMDGPU.td at aaba37187fda7f5a7fdc4c1e6129cbaaa1bbf709 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/aaba37187fda7f5a7fdc4c1e6129cbaaa1bbf709/llvm/lib/Target/AMDGPU/AMDGPU.td#L1086-L1101) {{< /quote >}}
 
 > 		def FeatureGFX10_AEncoding : SubtargetFeature<"gfx10_a-encoding",
 > 		  "GFX10_AEncoding",
 > 		  "true",
 > 		  "Has BVH ray tracing instructions"
 > 		>;
 >
 > {{< quote >}} [llvm-project/AMDGPU.td at aaba37187fda7f5a7fdc4c1e6129cbaaa1bbf709 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/aaba37187fda7f5a7fdc4c1e6129cbaaa1bbf709/llvm/lib/Target/AMDGPU/AMDGPU.td#L468-L472) {{< /quote >}}

*RDNA アーキテクチャ* を採用する APU であり、*Navi10 (gfx1010)* をベースに *RDNA 2/GFX10.3* 世代同様のレイトレーシング命令周りを追加した奇妙な AMD GPU が *gfx1013* となる。  

## Cyan Skilfish {#cyan_skilfish}

*Cyan Skilfish* は 2021/07/20、AMD GPU ドライバーにサポートを追加するパッチが公開された *RDNA APU* 。  
{{< link >}} [Linux Kernel に RDNA APU 「Cyan Skilfish」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/07/21/amd-cyan_skilfish-rdna-apu/) {{< /link >}}

LLVM においては *gfx1013* が、AMD GPU ドライバーでは *Cyan Skilfish* が唯一の *RDNA APU* であることから関連が考えられていたが、先日投稿された AMDGPU KFD (Kernel Fusion Driver) に、GPUID 情報を追加するパッチにより、2つが確実に結び付けられた。  
*Cyan Skilfish* の GPUID は *gfx1013* であり、上で述べた特徴は *Cyan Skilfish* に適用されることとなる。  

 > 		 static const struct kfd_device_info cyan_skillfish_device_info = {
 > 		 	.asic_family = CHIP_CYAN_SKILLFISH,
 > 		 	.asic_name = "cyan_skillfish",
 > 		+	.gfx_version = 100103,
 > 		 	.max_pasid_bits = 16,
 > 		 	.max_no_of_hqd  = 24,
 > 		 	.doorbell_size  = 8,
 >
 > {{< quote >}} [[PATCH] drm/amdkfd: Expose GFXIP engine version to sysfs](https://lists.freedesktop.org/archives/amd-gfx/2021-July/067107.html) {{< /quote >}}

### ディスプレイ出力を持たない 8-Core APU {#8-core}

AMD GPU ドライバーへのパッチから分かる *Cyan Skilfish* の特徴 {{< comple >}} というより奇妙な点 {{< /comple >}} を言えば、当パッチでは *Cyan Skilfish* のディスプレイエンジン/コントローラーとマルチメディアエンジンの IPブロックのサポートが追加されていない。  

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

また、GPU ドライバーであるから APU の CPU側の情報は少ないが、SMU (System Management Unit) に関する部分に *Cyan Skilfish* が 8-Core APU とする記述が存在する。  

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

ディスプレイ出力機能を持たない、あるいは想定されておらず、CPU 8-Core を持つ AMD APU。  
これに近い存在として [AMD 4700S Desktop Kit](/tags/amd_4700s) がいる。  

*AMD 4700S* は GDDR6メモリを採用することからゲーム機 APU をベースにしている可能性が考えられるCPU で、*Zen 2 アーキテクチャ* 8-Core を持つ。  
{{< link >}} [AMD 4700S の正体は GPU部を無効化したゲーム機向け APU だったか | Coelacanth's Dream](/posts/2021/04/26/amd-4700s-identity/) {{< /link >}}
しかし、APU だとしても **AMD 4700S Desktop Kit** ではディスプレイ出力がボードに実装されておらず、PCIeスロットに搭載した dGPU に頼る必要があり、APU としての GPU部は実質無効化されている。  
ディスプレイ出力がボードに実装されておらず、APU の出力機能を活用することができず、CPU 8-Core を搭載する ―― こうした点が *Cyan Skilfish* と **AMD 4700S Desktop Kit** で共通する。  

あくまでもこれは推測であり、*AMD 4700S* の `lspci` 実行結果でも得られればもう少し近付けるように思うが、信頼できる実行結果が必要となる。自分で購入して確かめるのが手っ取り早いが、入手性とか、未だに構成を更新していないメインマシンを後回しにしてまで買うべきなのかとか、まあ色々問題と葛藤が起こる。  
