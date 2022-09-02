---
title: "L0ベクタキャッシュと GL1データキャッシュが増やされる RDNA 3/GFX11 APU"
date: 2022-09-02T14:42:02+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "GFX11", "Linux_Kernel", "Phoenix" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

*RDNA 3/GFX11* では CU ごとに持つ L0ベクタキャッシュと Shader Array (SA) ごとに持つ GL1データキャッシュ (RDNA L1キャッシュ) が、*RDNA 2/GFX10.3* から倍のサイズに増やされることが AMD の Liu, Aaron 氏によって明かされている。  

 * [[PATCH] drm/amdkfd: Match GC 11.0.1 cache info to yellow carp](https://lists.freedesktop.org/archives/amd-gfx/2022-September/083698.html)

*RDNA 3/GFX11* からは IP Discovery Table からキャッシュ情報を読み取るようになり、AMDGPUドライバー中にハードコードされることはなくなったとされている。  
しかし、*Phoenix APU (GC 11.0.1, GFX1103)* に限ってはキャッシュ情報が含まれておらず、他の *RDNA 3/GFX11* GPU のように読み取れないため、*Yellow Carp/Rembrandt APU* のキャッシュ情報を用いるようにするのが当該のパッチとなる。[^ip-discovery]  
パッチは AMD の Zhang, Yifan 氏によって投稿されている。  

[^ip-discovery]: [次世代 GPU IPブロックのサポートが進む AMDGPUドライバー ―― GFX11, GC 11.0, MES 11.0, IMU | Coelacanth's Dream](http://localhost:1313/posts/2022/04/30/amd-gc_11_0_0/#ip)

*Phoenix APU (GC 11.0.1, GFX1103)* と *Yellow Carp/Rembrandt APU* のキャッシュ情報は完全には一致せず、異なる点が L0ベクタキャッシュ (TCP L1 Cache per CU) と GL1データキャッシュとなる。  
それに触れたのが以下の Liu, Aaron 氏によるコメント。  

 > 		[Public]
 > 		
 > 		Hi Yifan,
 > 		
 > 		Yellow carp's cache info cannot be duplicated to GC_11_0_1.
 > 		
 > 		Different point to GC_11_0_1:
 > 		TCP L1  Cache size is 32
 > 		GL1 Data Cache size per SA is 256
 > 		
 > 		Others looks good to me
 > 		
 > 		--
 > 		Best Regards
 > 		Aaron Liu
 > 		
 >
 > {{< quote >}} [[PATCH] drm/amdkfd: Match GC 11.0.1 cache info to yellow carp](https://lists.freedesktop.org/archives/amd-gfx/2022-September/083698.html) {{< /quote >}}

*Phoenix APU (GC 11.0.1, GFX1103)* では、L0ベクタキャッシュは 16KiB から 32KiB に、GL1データキャッシュは 128KiB から 256KiB になるとされている。  
他キャッシュは同じとされており、WGP (CU 2基) ごとに持つスカラ命令キャッシュ (L1i) とスカラデータキャッシュ (L1k)、GPU全体で共有する L2データキャッシュのサイズについては *Yellow Carp/Rembdrandt APU* と変わらない。  
以下は *Yellow Carp/Rembrandt APU* のキャッシュ情報となり、スカラ命令キャッシュは 32KiB、スカラデータキャッシュは 16KiB、L2データキャッシュは 2048KiB (2MiB) となる。  
それと MALL (Infinity Cache, L3データキャッシュ) に触れられていないことから、やはり APU では MALL を搭載しない方針と思われる。  

 > 		static struct kfd_gpu_cache_info yellow_carp_cache_info[] = {
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
 > 				.num_cu_shared = 6,
 > 			},
 > 			{
 > 				/* L2 Data Cache per GPU (Total Tex Cache) */
 > 				.cache_size = 2048,
 > 				.cache_level = 2,
 > 				.flags = (CRAT_CACHE_FLAGS_ENABLED |
 > 						CRAT_CACHE_FLAGS_DATA_CACHE |
 > 						CRAT_CACHE_FLAGS_SIMD_CACHE),
 > 				.num_cu_shared = 6,
 > 			},
 > 		};
 > 		
 > {{< quote >}} [linux-kernel v5.19] drivers/gpu/drm/amd/amdkfd/kfd_crat.c {{< /quote >}}

L0ベクタキャッシュと GL1データキャッシュの変化はあくまでも *Phoenix APU (GC 11.0.1, GFX1103)* を指定したものではあるが、*RDNA 3/GFX11* dGPU でも同様の変更を採用していると思われる。  
*RDNA 3/GFX11* では他にもキャッシュ構成の変更が施されており、スカラ命令キャッシュのキャッシュラインサイズが *RDNA 2/GFX10.3* の 64Byte から 128Byte に拡大されている。  

| Cache Line Size               | L1i  | L1k   | L1d   | GL1   | GL2 |
| :--------------               | :-:  | :-:   | :-:   | :-:   | :-: |
| GFX9/Vega (without Aldebaran) | 32B  | 32B   | 64B   | -     | 64B |
| GFX10/RDNA 1, GFX10.3/RDNA 2  | 64B  | 64B   | 128B  | 128B  | 128B |
| GFX11                         | 128B | 128B? | 128B? | 128B? | 128B |

*RDNA 3/GFX11* では `VOPD (Dual issue wave32)` 命令[^vopd]や `WMMA (Wave Matrix Multiply-accumulate)` 命令[^wmma]に対応し、WGP (CU) あたりの性能が向上していると考えられ、それに伴って L0ベクタキャッシュのサイズが増やされたと考えられる。  
GL1データキャッシュは Shader Array 内の WGP 以外に RB (Render Backend) のキャッシュにも接続されており、GL1データキャッシュを増やすことでピクセル処理性能の向上を狙ったことも考えられる。  

[^l1i]: [RadeonSI ドライバーで GFX11 のサポートが進められる | Coelacanth's Dream](/posts/2022/05/05/radeonsi-gfx11/#inst-cache-line-size)
[^vopd]: [GFX11 でサポートされる VOPD (Dual issue wave32) 命令の範囲と特徴 | Coelacanth's Dream](/posts/2022/06/21/gfx11-vopd-instruction/)
[^wmma]: [RDNA 3/GFX11 では行列積和演算命令、WMMA (Wave Matrix Multiply-accumulate) をサポート | Coelacanth's Dream](/posts/2022/06/29/gfx11-wmma-inst/)

| Cache Info                | Yellow Carp (Rembrandt) | Phoenix (GC 11.0.1, GFX1103) |
| :--                       | :--:                    | :--:                         |
| L1 Vector Data (per CU)   | 16KiB                   | *32KiB* |
| L1 Scalar Inst. (per WGP) | 32KiB                   | 32KiB |
| L1 Scalar Data (per WGP)  | 16KiB                   | 16KiB |
| GL1 Data (per SA)         | 128KiB                  | *256KiB* |
| L2 Data                   | 2048KiB (2MiB)          | 2048KiB (2MiB) |
| L3 (MALL)                 | N/A                     | N/A |

| GC (Graphics Compute) IP ver | GFX ID | AMDGPU_FAMILY | Type |
| :-------- | :-----: | :--: | :--: |
| 11.0.0    | gfx1100 (Navi31)[^tensile-gfx11] | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100) | dGPU |
| 11.0.1    | gfx1103 (Phoenix) | AMDGPU_FAMILY_GC_11_0_1 (FAMILY_GFX1103) | APU  |
| 11.0.2    | gfx1102 (Navi33) | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100) | dGPU |
| 11.0.3?   | gfx1101 (Navi32)? | AMDGPU_FAMILY_GC_11_0_0 (FAMILY_GFX1100)? | dGPU |

[^tensile-gfx11]: [Support Tensile for gfx11 series platform by TonyYHsieh · Pull Request #1521 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1521/commits/3796d41aec358721fced1ed4337c27f69aeda3bb)


