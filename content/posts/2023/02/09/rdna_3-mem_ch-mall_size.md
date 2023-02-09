---
title: "Navi31/Navi32/Navi33 のメモリチャネル数と MALL サイズ"
date: 2023-02-09T19:57:29+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "RDNA_3", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日、AMDGPU ドライバーのディスプレイエンジン、DCN (Display Core Next) のコード部に、*RDNA 3* GPU のメモリチャネル数と MALL (Memory Access at Last Level, Infinity Cache, L3キャッシュ) サイズが異なる SKU に対応する変更が適用された。  
パッチは amd-gfx メーリングリストには投稿されておらず、おそらくは AMD 内部でのレビューを経てブランチにコミットが追加されている。  

 * [drm/amd/display: adjust MALL size available for DCN32 and DCN321 (235fef6c) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/235fef6c7fd341026eee90cc546e6e8ff8b2c315)
 * [drm/amd/display: fix MALL size hardcoded for DCN321 (918d5166) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/918d5166439078364453f2eb5b4d8e75095a510e)
 * [[pull] amdgpu drm-next-6.3](https://lists.freedesktop.org/archives/amd-gfx/2023-February/089241.html)

MALL はメモリサイドキャッシュであり、メモリチャネルの一部を無効化した場合、その分使用可能な MALL サイズは減ることとなる。  
これまでは DCN が使用可能な MALL サイズを、DCN 3.2 は 64MiB、DCN 3.21 は 32MiB と直接コードに記述していたが (ハードコード)、パッチでは VBIOS から取得可能なメモリチャネル数を基に計算する方法に変更している。  
これによりメモリチャネル数を減らした結果、MALL サイズがハードコードされた値より小さくなった SKU もサポートされたと考えられる。  
また、ここではメモリチャネルあたりの MALL サイズを 4MiB としている。*RDNA 2* 世代では MALL サイズが 2種類あり、*Navi21, Navi22* はメモリチャネルあたり 8MiB、*Navi23, Navi24* は 4MiB としていたが、*RDNA 3* 世代では現状 4MiB で統一されているらしい。  

 >         +
 >         +unsigned int dcn32_calc_num_avail_chans_for_mall(struct dc *dc, int num_chans)
 >         +{
 >         +	/*
 >         +	 * DCN32 and DCN321 SKUs may have different sizes for MALL
 >         +	 *  but we may not be able to access all the MALL space.
 >         +	 *  If the num_chans is power of 2, then we can access all
 >         +	 *  of the available MALL space.  Otherwise, we can only
 >         +	 *  access:
 >         +	 *
 >         +	 *  max_cab_size_in_bytes = total_cache_size_in_bytes *
 >         +	 *    ((2^floor(log2(num_chans)))/num_chans)
 >         +	 *
 >         +	 * Calculating the MALL sizes for all available SKUs, we
 >         +	 *  have come up with the follow simplified check.
 >         +	 * - we have max_chans which provides the max MALL size.
 >         +	 *  Each chans supports 4MB of MALL so:
 >         +	 *
 >         +	 *  total_cache_size_in_bytes = max_chans * 4 MB
 >         +	 *
 >         +	 * - we have avail_chans which shows the number of channels
 >         +	 *  we can use if we can't access the entire MALL space.
 >         +	 *  It is generally half of max_chans
 >         +	 * - so we use the following checks:
 >         +	 *
 >         +	 *   if (num_chans == max_chans), return max_chans
 >         +	 *   if (num_chans < max_chans), return avail_chans
 >         +	 *
 >         +	 * - exception is GC_11_0_0 where we can't access max_chans,
 >         +	 *  so we define max_avail_chans as the maximum available
 >         +	 *  MALL space
 >         +	 *
 >         +	 */
 >
 > {{< quote >}} [drm/amd/display: adjust MALL size available for DCN32 and DCN321 (235fef6c) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/235fef6c7fd341026eee90cc546e6e8ff8b2c315) {{< /quote >}}

GPU がサポートするメモリチャネルとそこに接続された MALL を、DCN 部ですべて使用できる訳ではないとし、GPU ごとに使用可能なメモリチャネル数を判定するコードが追加されている。  
*Navi32 (GC 11.0.3, gfx1101)* はメモリチャネルを最大 16ch (256-bit?, 16ch x 16-bit) をサポートし、そこから減らした SKU の場合、DCN は 8ch 分の MALL が使用可能となっている。  
*Navi33 (GC 11.0.2, gfx1102)* は最大 8ch (128-bit) をサポート、そこから減らした SKU の場合 DCN は 4ch 分が使用可能となる。  

奇妙なのは *Navi31 (GC 11.0.0, gfx1100)* であり、メモリチャネル数を減らしていない場合でも DCN はすべてのメモリチャネル分の MALL を使用することはできない。  
さらに、判定部分では VBIOS から取得できるメモリチャネル数について最大 48ch となることを想定しており、その場合 DCN は 32ch 分を、そうでない場合は 16ch 分の MALL が使用可能とされている。  
現時点で発表されている *Navi31 (GC 11.0.0, gfx1100)* ベースの SKU、**RX 7900 XTX** は 24ch (384-bit)[^7900-xtx]、**RX 7900 XT** は 20ch (320-bit) の仕様となっている。[^7900-xt]  
最大 48ch というのはそれらより大きい値だが、これらの情報は Linux Kernel とその AMDGPU ドライバーがオープンソースで開発されているから公開されているのであり、当然ハードコードされた値が今後変更される可能性は充分にある。  

[^7900-xtx]: [AMD Radeon™ RX 7900 XTX | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-7900xtx#product-specs)
[^7900-xt]: [AMD Radeon™ RX 7900 XT | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-7900xt#product-specs)

AMDGPU ドライバーは、電源投入時に GPU 内部の PSP (Platform Security Processor) が VRAM に出力する GPU 情報、IP discovery table に対応しており、  
GPU 側が対応していれば MALL の情報、メモリコントローラあたりのサイズ、通常の倍のサイズ、もしくは半分を使用するかといった情報も出力されるが、DCN 部のコードでその値は使われていない。[^ip-discovery]  

[^ip-discovery]: [次世代 GPU IPブロックのサポートが進む AMDGPUドライバー ―― GFX11, GC 11.0, MES 11.0, IMU | Coelacanth's Dream](/posts/2022/04/30/amd-gc_11_0_0/#ip)

 >         +	int gc_11_0_0_max_chans = 48;
 >         +	int gc_11_0_0_max_avail_chans = 32;
 >         +	int gc_11_0_0_avail_chans = 16;
 >         +	int gc_11_0_3_max_chans = 16;
 >         +	int gc_11_0_3_avail_chans = 8;
 >         +	int gc_11_0_2_max_chans = 8;
 >         +	int gc_11_0_2_avail_chans = 4;
 >         +
 >         +	if (ASICREV_IS_GC_11_0_0(dc->ctx->asic_id.hw_internal_rev)) {
 >         +		return (num_chans == gc_11_0_0_max_chans) ?
 >         +			gc_11_0_0_max_avail_chans : gc_11_0_0_avail_chans;
 >         +	} else if (ASICREV_IS_GC_11_0_2(dc->ctx->asic_id.hw_internal_rev)) {
 >         +		return (num_chans == gc_11_0_2_max_chans) ?
 >         +			gc_11_0_2_max_chans : gc_11_0_2_avail_chans;
 >         +	} else { // if (ASICREV_IS_GC_11_0_3(dc->ctx->asic_id.hw_internal_rev)) {
 >         +		return (num_chans == gc_11_0_3_max_chans) ?
 >         +			gc_11_0_3_max_chans : gc_11_0_3_avail_chans;
 >         +	}
 >         +}
 >
 > {{< quote >}} [drm/amd/display: adjust MALL size available for DCN32 and DCN321 (235fef6c) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/235fef6c7fd341026eee90cc546e6e8ff8b2c315) {{< /quote >}}
