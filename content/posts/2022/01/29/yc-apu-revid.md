---
title: "Yellow Carp/Rembrandt APU の DeviceID と Revision、Stepping"
date: 2022-01-29T03:01:05+09:00
draft: false
tags: [ "Yellow_Carp", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

以前、*Yellow Carp/Rembrandt APU* にはすでに 2つの PCI ID/DeviceID が用意されており、DeviceID と externel revision id (external_rev_id) の関係から、*Yellow Carp* と同じ GPUアーキテクチャ、各種 IP を採用する別の APU が存在するのではないか、ということを書いたが、これは間違っていた。  
{{< link >}} [既に 2つの DeviceID が用意されている Yellow Carp APU ファミリー | Coelacanth's Dream](/posts/2021/07/26/yc-apu-two-did/) {{< /link >}}

## Yellow Carp A0/B0/B1 {#step}
まず 2021/07 に投稿されたパッチから、*Yellow Carp* には 2つの DeviceID、`0x164D`, `0x1681` が割り当てられていることが明らかにされた。  
そして AMD APU/GPU には、シリコン自体に割り当てられる `internal/external_rev_id` があるが、上記 2つの DeviceID はそれぞれ異なる `external_rev_id` が割り当てられていた。  
以上のことから、基本的な構成は同じだが *Yellow Carp* とは別の APU が存在すると自分は推測した。  
しかしその後、2021/10 に `external_rev_id` を修正するパッチが投稿された。  

 * [[PATCH] drm/amdgpu: support B0&B1 external revision id for yellow carp](https://lists.freedesktop.org/archives/amd-gfx/2021-October/070389.html)
 * [[PATCH] drm/amdgpu/display: add yellow carp B0 with rest of driver](https://lists.freedesktop.org/archives/amd-gfx/2021-October/070502.html)

上 2つのパッチでは、*Yellow Carp B0/B1ステッピング* の正しい `external_rev_id` は `0x20` だとしている。  
*Yellow Carp* では DeviceID と `external_rev_id` は結びついており、`DeviceID: 0x1681` には `0x20 (B0/B1)` 、`DeviceID: 0x164D` には `0x1 (A0)` が対応している。  

 > 		 		if (adev->pdev->device == 0x1681)
 > 		-			adev->external_rev_id = adev->rev_id + 0x19;
 > 		+			adev->external_rev_id = 0x20;
 > 		 		else
 > 		 			adev->external_rev_id = adev->rev_id + 0x01;
 > 		 		break;
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: support B0&B1 external revision id for yellow carp](https://lists.freedesktop.org/archives/amd-gfx/2021-October/070389.html) {{< /quote >}}

ステッピングによる違いは、AMDGPUドライバーのコードを探る限り、主にディスプレイ部だと思われる。  
*Yellow Carp* のディスプレイコントローラ/エンジンである DCN3.1 周りのコードには、*B0/B1* のみを対象にした部分、*B0/B1 でない* 場合のパターンを想定した部分があり、*A0* と *B0/B1* で一部機能や設定が異なる  

 > 			/* Legacy path, avoid if possible. */
 > 			if (enc->ctx->asic_id.hw_internal_rev != YELLOW_CARP_B0) {
 > 				REG_GET(RDPCSTX_PHY_CNTL6, RDPCS_PHY_DPALT_DISABLE,
 > 					&dp_alt_mode_disable);
 > 			} else {
 > 				/*
 > 				 * B0 phys use a new set of registers to check whether alt mode is disabled.
 > 				 * if value == 1 alt mode is disabled, otherwise it is enabled.
 > 				 */
 > 				if ((enc10->base.transmitter == TRANSMITTER_UNIPHY_A) ||
 > 				    (enc10->base.transmitter == TRANSMITTER_UNIPHY_B) ||
 > 				    (enc10->base.transmitter == TRANSMITTER_UNIPHY_E)) {
 > 					REG_GET(RDPCSTX_PHY_CNTL6, RDPCS_PHY_DPALT_DISABLE,
 > 						&dp_alt_mode_disable);
 > 				} else {
 > 					REG_GET(RDPCSPIPE_PHY_CNTL6, RDPCS_PHY_DPALT_DISABLE,
 > 						&dp_alt_mode_disable);
 > 				}
 > 			}
 > {{< quote >}} [dcn31_dio_link_encoder.c « dcn31 « dc « display « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/display/dc/dcn31/dcn31_dio_link_encoder.c?h=next-20220128#n619) {{< /quote >}}

 > 			if (dc->ctx->asic_id.chip_family == FAMILY_YELLOW_CARP &&
 > 			    dc->ctx->asic_id.hw_internal_rev == YELLOW_CARP_B0 &&
 > 			    !dc->debug.dpia_debug.bits.disable_dpia) {
 > 				/* YELLOW CARP B0 has 4 DPIA's */
 > 				pool->base.usb4_dpia_count = 4;
 > 			}
 >
 > {{< quote >}} [dcn31_resource.c « dcn31 « dc « display « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/display/dc/dcn31/dcn31_resource.c?h=next-20220128#n2477) {{< /quote >}}

実際の *Yellow Carp APU* にはどちらの ID が割り当てられているかについて、DeviceID といった情報も公開しているような *Ryzen 6000U/H/HS/HX* 搭載製品レビューを見つけられなかったが、以前に取り上げたエンジニアサンプリング品の Linux Kernel ブートログはある。  
{{< link >}} [AMD Yellow Carp/Rembrandt APU ブートログ ―― LilacKD-RMB, 100-000000560-40_Y | Coelacanth's Dream](/posts/2021/12/14/yc-rmb-bootlog/) {{< /link >}}
そのエンジニアサンプリング、*100-000000560-40_Y* では GPU部の DeviceID が `0x1681` となっており、先に書いたように `DeviceID 0x1681` には *B0/B1 ステッピング* が結びつけられている。  

 > 		[    3.876317] amdgpu: HMM registered 512MB device memory
 > 		[    3.876352] amdgpu: SRAT table not found
 > 		[    3.876354] amdgpu: Virtual CRAT table created for GPU
 > 		[    3.876394] amdgpu: Topology: Add dGPU node [0x1681:0x1002]
 > 		[    3.876398] kfd kfd: amdgpu: added device 1002:1681
 > 		[    3.876417] amdgpu 0000:62:00.0: amdgpu: SE 1, SH per SE 2, CU per SH 6, active_cu_number 12
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}

実 SKU と近い仕様であるエンジニアサンプリング品の時点で *B0/B1 ステッピング* の DeviceID が割り当てられていることから、実際の製品に搭載されたものでも *B0/B1 ステッピング* になっているものと思われる。  
おそらく *Yellow Carp* は開発途中でディスプレイ部、あるいはそれ以外の部分も一部設計を変更、修正し、その前を *A0 ステッピング* 、以降を *B0/B1 ステッピング* としたのではないだろうか。  
*Yellow Carp* は最近の AMD APU では大きなアップデートが行われており、PCIe Gen4 や USB4 に対応したことはステッピングの更新と無関係ではないと推察される。  

まとめれば、*Yellow Carp APU* には 2つの DeviceID があるが、`0x164D` は *A0 ステッピング* に割り当てられており、*A0 ステッピング* と *B0/B1 ステッピング* にはディスプレイ部に違いが存在する。  
どちらも *Yellow Carp* であり、以前自分が推測したような別の APU が存在するということにはならない。  
そして実際に世に出回る主要な *Yellow Caro APU* は *B0/B1 ステッピング* だろう、となる。  

