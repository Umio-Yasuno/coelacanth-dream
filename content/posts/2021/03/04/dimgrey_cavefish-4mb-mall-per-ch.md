---
title: "Dimgrey Cavefish/Navi23 の Infinity Cache はメモリチャネルあたり 4MiB"
date: 2021-03-04T05:15:07+09:00
draft: false
tags: [ "Dimgrey_Cavefish", "RDNA_2", "Navi23" ]
# keywords: [ "", ]
categories: [ "AMD", "GPU", "Hardware" ]
noindex: false
# summary: ""
toc: false
---

*Sienna Cichlid/Navi21* 、*Navy Flounder/Navi22* が L3キャッシュに位置する *Infinity Cache* を、メモリチャネルあたり 8MiB のサイズを持つのに対し、  
*Dimgrey Cavefish/Navi23* のサイズを採ることが Linux Kernel (amd-gfx) へのパッチから判明した。  

{{< pindex >}}
 * [メモリチャネルあたり 4MiB の Infinity Cache](#mall-size)
 * [ディスプレイエンジン数も 1基少ない Dimgrey Cavefish](#disp-engine)
{{< /pindex >}}


## メモリチャネルあたり 4MiB の Infinity Cache {#mall-size}

判明したと言っても、パッチが投稿されたのは 2021/01/19、1ヶ月半近く前で、自分が今まで気付いていなかっただけだ。  

 * [[PATCH 3/3] drm/amd/display: Update dcn30_apply_idle_power_optimizations() code](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058679.html)

以下に引用した部分のコードは、DCN 3.02 関連のファイルに追加されたもので、そして DCN 3.02 は *Dimgrey Cavefish/Navi23* に搭載されるディスプレイエンジンIPとなる。  
{{< link >}} [Linux Kernel に RDNA 2 GPU "Dimgrey Cavefish" をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2020/10/08/amd-dimgrey_cavefish-linux-kernel-patch/) {{< /link >}}
`caps.mall_size_per_mem_channel` に 4 が代入され、`caps.mall_size_total` を `dc->caps.mall_size_per_mem_channel * dc->ctx->dc_bios->vram_info.num_chans * 1048576` 、  
わかりやすく言い換えれば、*MALL* の総サイズを `メモリチャネルあたりの MALL サイズ * メモリチャネル数 * 1048576 [2^20]` で求めている。  

 >        --- a/drivers/gpu/drm/amd/display/dc/dcn302/dcn302_resource.c
 >        +++ b/drivers/gpu/drm/amd/display/dc/dcn302/dcn302_resource.c
 >        @@ -1316,7 +1316,9 @@ static bool dcn302_resource_construct(
 >         	dc->caps.max_cursor_size = 256;
 >         	dc->caps.min_horizontal_blanking_period = 80;
 >         	dc->caps.dmdata_alloc_size = 2048;
 >        -
 >        +	dc->caps.mall_size_per_mem_channel = 4;
 >        +	/* total size = mall per channel * num channels * 1024 * 1024 */
 >        +	dc->caps.mall_size_total = dc->caps.mall_size_per_mem_channel * dc->ctx->dc_bios->vram_info.num_chans * 1048576;
 >         	dc->caps.cursor_cache_size = dc->caps.max_cursor_size * dc->caps.max_cursor_size * 8;
 >
 > {{< quote >}} [[PATCH 3/3] drm/amd/display: Update dcn30_apply_idle_power_optimizations() code](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058679.html) {{< /quote >}}

*MALL* については *Memory Access at Last Level* の略であり、*Last Level Cache* の別名。オープンソースドライバーのコード中では *MALL* が主に使われている。マーケティング的には *Infinity Cache* と呼ばれる。  
{{< link >}} [Sienna Cichlid は MALL 機能をサポート | Coelacanth's Dream](/posts/2020/10/21/sienna_cichlid-support-mall/) {{< /link >}}

 >        // =====================================================================================================================
 >        // Helper function for calculating an SRD's "llc_noalloc" field (last level cache, aka the mall).
 >
 > {{< quote >}} [pal/gfx9Device.cpp at c9d1904b35ac5eb70bd4738a5bf5e537adbf5ab0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/c9d1904b35ac5eb70bd4738a5bf5e537adbf5ab0/src/core/hw/gfxip/gfx9/gfx9Device.cpp) {{< /quote >}}

そして、*Sienna Cichlid/Navi21* と *Navy Flounder/Navi22* が搭載する DCN 3.0 関連ファイルの同様の部分は 8 [MiB] となっており、これは発表済みの **Radeon RX 6000 シリーズ** の仕様と一致する。  
*Sienna Cichlid/Navi21* は GDDR6 256-bit に対して *MALL/Infinity Cache* は 128MiB、*Navy Flounder/Navi22* は 192-bit に対して 96MiB となっている。(厳密に言えば、**Radeon RX 6700 XT** == *Navy Flounder/Navi22* というのはまだ確認できていないが。だが恐らくそうだろう)  

 >        	dc->caps.mall_size_per_mem_channel = 8;
 >        	/* total size = mall per channel * num channels * 1024 * 1024 */
 >        	dc->caps.mall_size_total = dc->caps.mall_size_per_mem_channel * dc->ctx->dc_bios->vram_info.num_chans * 1048576;
 >
 > {{< quote >}} [drm/amd/display: Update idle optimization handling (4f8d4007) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/4f8d4007752e45b1cb5a9b649a2271565af7b550#4395e2a537b31b3470d7ff79bb73bfea155dab0e) {{< /quote >}}

以上のことから、*Dimgrey Cavefish/Navi23* のメモリチャネルあたりの *MALL/Infinity Cache* は、 *Sienna Cichlid/Navi21* 、 *Navy Flounder/Navi22* よりも半分のサイズとなる 4 [MiB] だと考えられる。  

また、*RDNA 2 アーキテクチャ* を採用する APU に [VanGogh](/tags/vangogh) がいるが、*VanGogh APU* が搭載する DCN 3.01 の関連ファイルには *MALL/Infinity Cache* の記述が無い。*RDNA 2* ではあるが *MALL/Infinity Cache* を省いていると思われる。  
APU がターゲットとする性能帯とコストから省いたのではないかと考えられる。  

## ディスプレイエンジン数も 1基少ない Dimgrey Cavefish {#disp-engine}

*Dimgrey Cavefish/Navi23* が *Sienna Cichlid/Navi21* 、*Navy Flounder/Navi22* より規模を減らしているのは *MALL/Infinity Cache* だけでなく、ディスプレイエンジン数の数も 1基少ない 5基となっている。  

 >      +static const struct resource_caps res_cap_dcn302 = {
 >      +		.num_timing_generator = 5,
 >      +		.num_opp = 5,
 >      +		.num_video_plane = 5,
 >      +		.num_audio = 5,
 >      +		.num_stream_encoder = 5,
 >      +		.num_dwb = 1,
 >      +		.num_ddc = 5,
 >      +		.num_vmid = 16,
 >      +		.num_mpc_3dlut = 2,
 >      +		.num_dsc = 5,
 >      +};
 >      +
 >
 > {{< quote >}} [[PATCH 49/50] drm/amd/display: Add support for DCN302 (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054588.html) {{< /quote >}}

Linux Kernel (amd-gfx) に *Dimgrey Cavefish/Navi23* をサポートする最初のパッチが投稿された時の記事でも触れたが、  
ディスプレイエンジンの数を減らすのは、*Polaris10* に対する *Polaris11/12* 、*Navi10* に対する *Navi14* のように、エントリー市場をターゲットとする GPU にはよく見られる設計点である。  
メモリチャネルあたりの *MALL/Infinity Cache* サイズを減らしたのも、*Dimgrey Cavefish/Navi23* はエントリー市場向けの *RDNA 2* GPU であり、製造コストの削減を意識した結果と思われる。  
あるいは、ターゲットの性能帯 {{< comple >}} 1080p と考えられる {{< /comple >}} では、メモリチャネルあたり 8 [MiB] の *MALL/Infinity Cache* を持っても効果が小さかった、4 [MiB] でも十分効果を発揮したため、最適なサイズ 4 [MiB] を選択した、という可能性もある。  

[^cache-rate]: [[画像] 「Radeon RX 6800」の秘密兵器はCPU由来の「Infinity Cache」だ (15/22) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1289/828/html/15_o.jpg.html)

