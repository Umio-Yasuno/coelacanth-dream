---
title: "Navi21 /Sienna Cichlid のメモリは GDDR6 か HBM2 か、新たに出てきたパッチを読む"
date: 2020-07-27T16:04:07+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2", "Navi21" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

前回の記事で *Navi21 /Sienna Cichlid* の専用メモリのタイプは、*Navi10 /Navi14* 同様に GDDR6 ではないかと書いたが、それが揺らぐようなパッチが出てきた。  
{{< link >}} [AMD Navi21 /Sienna Cichlid 情報近況　―― GDDR6と省電力とサーバ向けと 【2020/07/21】 | Coelacanth's Dream](/posts/2020/07/21/amd-sienna_cichlid-recent-info-2020-07-21/) {{< /link >}}

それが以下のパッチで、主な内容としては UMC(Unified Memory Controller)[^unified-memory-controller] Ver 8.7 にて ECC で検知されたメモリエラーをレポートする機能をサポートするものと推察される。  
{{< link >}} [[PATCH] drm/amdgpu: add support for umc 8.7 ras functions](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051898.html) {{< /link >}}

[^unified-memory-controller]: [drm/amdgpu: add umc v6_1_1 IP headers](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=03c9963f47a9efe204983fb0ea022814f8ce0084)

そして気になるのが以下の部分。  

 >       +/* HBM  Memory Channel Width */
 >       +#define UMC_V8_7_HBM_MEMORY_CHANNEL_WIDTH	128
 >       +/* number of umc channel instance with memory map register access */
 >       +#define UMC_V8_7_CHANNEL_INSTANCE_NUM		2
 >       +/* number of umc instance with memory map register access */
 >       +#define UMC_V8_7_UMC_INSTANCE_NUM		8
 >       +/* total channel instances in one umc block */
 >       +#define UMC_V8_7_TOTAL_CHANNEL_NUM	(UMC_V8_7_CHANNEL_INSTANCE_NUM * UMC_V8_7_UMC_INSTANCE_NUM)
 >       +/* UMC regiser per channel offset */
 >       +#define UMC_V8_7_PER_CHANNEL_OFFSET_SIENNA	0x400
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: add support for umc 8.7 ras functions](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051898.html) {{< /quote >}}

UMC v8.7 については初見だが、`UMC_V8_7_PER_CHANNEL_OFFSET_SIENNA` から *Navi21 /Sienna Cichlid* に関連付けられたものと思われる。[Vega20](/tags/vega20)、[Arcturus](/tags/arcturus) に関連付けられているのは UMC v6.1 であり、バージョンナンバーが一気に飛んでいることとなる。しかし、ECCに対応した製品を出す計画が無かっただけで、内部的には *Navi1x* 系にもUMC のバージョンが割り振られていたのであれば自然と考えられる。  

して、上に引用した部分には *Navi21 /Sienna Cihclid* が HBM系のメモリを採用するかのように記述されている。  

各マクロ名と値については、まず HBM系メモリは 128-bit のメモリチャネルを 8本束ね、1024-bit のインターフェイスを構成する。  
`UMC_V8_7_HBM_MEMORY_CHANNEL_WIDTH` はそのままメモリチャネルの幅、`UMC_V8_7_UMC_INSTANCE_NUM` は束ねる本数に対応していると考えられる。  
`UMC_V8_7_CHANNEL_INSTANCE_NUM` は、UMC v6.1 のファイルと比較すると、

 >       /* HBM  Memory Channel Width */
 >       #define UMC_V6_1_HBM_MEMORY_CHANNEL_WIDTH	128
 >       /* number of umc channel instance with memory map register access */
 >       #define UMC_V6_1_CHANNEL_INSTANCE_NUM		4
 >       /* number of umc instance with memory map register access */
 >       #define UMC_V6_1_UMC_INSTANCE_NUM		8
 >       /* total channel instances in one umc block */
 >       #define UMC_V6_1_TOTAL_CHANNEL_NUM	(UMC_V6_1_CHANNEL_INSTANCE_NUM * UMC_V6_1_UMC_INSTANCE_NUM)
 >       /* UMC regiser per channel offset */
 >       #define UMC_V6_1_PER_CHANNEL_OFFSET_VG20	0x800
 >       #define UMC_V6_1_PER_CHANNEL_OFFSET_ARCT	0x400
 >
 > {{< quote >}} [umc_v6_1.h\amdgpu\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/umc_v6_1.h?h=amd-staging-drm-next-sienna_cichlid&id=4cf781c24c3bc8cc50f8013143aa20b26e9217e8) {{< /quote >}}

UMC v6.1 では値が `4` となっている。  
比較できるファイルがこれしかないため、根拠とするには少し乏しいが、*Vega20* のスペックから、インターフェイス数を表しているのかもしれない。  

短絡的になれば、*Navi21 /Sienna Cichlid* は HBM2 2インターフェイス(2048-bit) に対応……とも考えられるが、[前回](/posts/2020/07/21/amd-sienna_cichlid-recent-info-2020-07-21/#sienna-gddr6) 書いたように、GDDR6メモリを採用するような記述も見られる。  

まさかの GDDR6 と HBM2 の両対応なんて考えが浮かんだりもするが、今の所それを判定するための文もない。  
{{< link >}} [[PATCH] drm/amdgpu: enable umc 8.7 functions in gmc v10](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051907.html) {{< /link >}}

まだ結論を出すことはできないが、考察の跡をつけ、今後のためとしたい。  

{{< ref >}}

 * [Navi21 /Sienna Cichlid のメモリは GDDR6 か HBM2 か、新たに出てきたパッチを読む | Coelacanth's Dream](/posts/2020/07/27/what-sienna_cichlid-hbm2/)

{{< /ref >}}
