---
title: "RDNA 3/GFX11 APU は AMDGPU_FAMILY_GC_11_0_1 に"
date: 2022-08-05T10:59:41+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU", "APU" ]
tags: [ "GFX11", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

以前に *RDNA 3/GFX11 APU* で採用されるディスプレイエンジン *DCN (Display Core Next) 3.1.4* のサポートを AMDGPUドライバーに追加するパッチを取り上げた。  
{{< link >}} [RDNA 3/GFX11 APU に採用される DCN 3.1.4 | Coelacanth's Dream](/posts/2022/07/09/dcn3_1_4-apu/) {{< /link >}}
その際、それらに関連付けられている `AMDGPU_FAMILY` が `AMDGPU_FAMILY_GC_11_0_2` となっており、*RDNA 3/GFX11 APU* は *GC (Graphics Compute) IP* のバージョンが *GC 11.0.1* であるため、少しややこしいということを書いた。  
それがやはりミスだったようで、AMD の Yifan Zhang 氏により、`AMDGPU_FAMILY_GC_11_0_2` を `AMDGPU_FAMILY_GC_11_0_1` に置き換えるパッチが投稿され、修正された。  

 * [[PATCH] drm/amd/display: change family id name for DCN314](https://lists.freedesktop.org/archives/amd-gfx/2022-August/082370.html)

AMDGPUドライバーは DeviceID とそれに紐付けられた GPU ASIC コードネームのサポートから、IPバージョンベースのサポートに移行したが、ユニークな名前を使わなくなった分、タイポや混同が起きやすい状況になったのかもしれない。  
自分も以前の記事で *GC IP* と *GFXID*、*AMDGPU_FAMILY* の対応表を掲載したが、最初のリビジョンで *Navi32 (gfx1101)* と *Navi33 (gfx1102)* を取り違えていた。  
以下は訂正版。  

| GC (Graphics Compute) IP ver | GFX ID | AMDGPU_FAMILY | Type |
| :-------- | :-----: | :--: | :--: |
| 11.0.0    | gfx1100 (Navi31)[^tensile-gfx11] | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100) | dGPU |
| 11.0.1    | gfx1103 | AMDGPU_FAMILY_GC_11_0_1 (FAMILY_GFX1103) | APU  |
| 11.0.2    | gfx1102 (Navi33) | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100) | dGPU |
| 11.0.3?   | gfx1101 (Navi32)? | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100)? | dGPU? |

[^tensile-gfx11]: [Support Tensile for gfx11 series platform by TonyYHsieh · Pull Request #1521 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1521/commits/3796d41aec358721fced1ed4337c27f69aeda3bb)

 > 		-  'gfx1010':'navi10', 'gfx1011':'navi12', 'gfx1012':'navi14', 'gfx1030':'navi21'
 > 		+  'gfx1010':'navi10', 'gfx1011':'navi12', 'gfx1012':'navi14', 'gfx1030':'navi21',
 > 		+  'gfx1100':'navi31', 'gfx1101':'navi32', 'gfx1102':'navi33'
 > 		 }
 >
 > {{< quote >}} [Support Tensile for gfx11 series platform by TonyYHsieh · Pull Request #1521 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1521/commits/3796d41aec358721fced1ed4337c27f69aeda3bb) {{< /quote >}}

最近の AMDGPUドライバーや ROCmソフトウェアの *RDNA 3/GFX11* 対応状況を追うと、*GFXID* から *Navi31/GC 11.0.0/gfx1100*、*Navi33/GC 11.0.2/gfx1102*、*GC 11.0.1/gfx1103* の対応が主に進められていることがわかる。  
ただ AMDGPUドライバーではまだ *Navi32/gfx1101* がまだ追加されておらず、[ROCmSoftwarePlatform/rocBLAS](https://github.com/ROCmSoftwarePlatform/rocBLAS) ライブラリでは *Navi31, Navi33* のサポートが追加されたが、こちらも *Navi32/gfx1101* は追加されていない。  

 > 				case IP_VERSION(11, 0, 0):
 > 					gfx_target_version = 110000;
 > 					f2g = &gfx_v11_kfd2kgd;
 > 					break;
 > 				case IP_VERSION(11, 0, 1):
 > 					gfx_target_version = 110003;
 > 					f2g = &gfx_v11_kfd2kgd;
 > 					break;
 > 				case IP_VERSION(11, 0, 2):
 > 					gfx_target_version = 110002;
 > 					f2g = &gfx_v11_kfd2kgd;
 > 					break;
 > 		
 >
 > {{< quote >}} [linux-5.19] drivers/gpu/drm/amd/amdkfd/kfd_device.c {{< /quote >}}

 > 		 # gpu arch configuration
 > 		 set( AMDGPU_TARGETS "all" CACHE STRING "Compile for which gpu architectures?")
 > 		-set_property( CACHE AMDGPU_TARGETS PROPERTY STRINGS all gfx803 gfx900 gfx906:xnack- gfx908:xnack- gfx90a:xnack+ gfx90a:xnack- gfx1010 gfx1012 gfx1030 )
 > 		+set_property( CACHE AMDGPU_TARGETS PROPERTY STRINGS all gfx803 gfx900 gfx906:xnack- gfx908:xnack- gfx90a:xnack+ gfx90a:xnack- gfx1010 gfx1012 gfx1030 gfx1100 gfx1102 )
 > 		
 >
 > {{< quote >}} [support Tensile GEMM functionality on gfx11 platforms · ROCmSoftwarePlatform/rocBLAS@df665de](https://github.com/ROCmSoftwarePlatform/rocBLAS/commit/df665dec2f05011531def741eca4eb0f07a6aa22) {{< /quote >}}

これに関しては LLVM にヒントがあり、一部 *RDNA 3/GFX11* には SGPR (スカラレジスタ) の初期化処理にバグ `FeatureUserSGPRInit16Bug` があるらしいのだが、影響を受けるのが現状 *Navi31/gfx1100* と *Navi33/gfx1102* だけとされ、*Navi32/gfx1101* と *gfx1103 (APU)* は影響を受けないとされている。  
*GFXID* では連番となっているが、恐らく開発時期に違いがあるのだろう。このことから *Navi32* のサポートの追加が *Navi31, Navi33* と一緒にされていないのだと思われる。  
他の理由としては、単に発表、発売時期から逆算してまだパッチを公開する必要がないとかが考えられるが。  

 > 		def FeatureISAVersion11_0_0 : FeatureSet<
 > 		  !listconcat(FeatureISAVersion11_Common.Features,
 > 		    [FeatureUserSGPRInit16Bug])>;
 > 		
 > 		def FeatureISAVersion11_0_1 : FeatureSet<
 > 		  !listconcat(FeatureISAVersion11_Common.Features,
 > 		    [])>;
 > 		
 > 		def FeatureISAVersion11_0_2 : FeatureSet<
 > 		  !listconcat(FeatureISAVersion11_Common.Features,
 > 		    [FeatureUserSGPRInit16Bug])>;
 > 		
 > 		def FeatureISAVersion11_0_3 : FeatureSet<
 > 		  !listconcat(FeatureISAVersion11_Common.Features,
 > 		    [])>;
 >
 > {{< quote >}} <https://github.com/llvm/llvm-project/blob/3cfa9b14312bd0f5eab84965a85d366ea59914ac/llvm/lib/Target/AMDGPU/AMDGPU.td#L1275-L1289> {{< /quote >}}
