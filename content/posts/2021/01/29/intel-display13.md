---
title: "Linux Kernel に 「Intel Display13」をサポートする最初のパッチが投稿される"
date: 2021-01-29T23:33:20+09:00
draft: false
tags: [ "Gen13", "XE_LPD" ]
# keywords: [ "", ]
categories: [ "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel (intel-gfx) に、*Display13* をサポートするパッチが投稿された。  
*Display13* は今後の Intelプラットフォームに採用されるディスプレイIP となり、現バージョンである *Display12* {{< comple >}} Tiger Lake, Rocket Lake, DG1, Alder Lake に採用されている {{< /comple >}} から自然に進化したものと説明されている。  
また、*Display13* は便宜的な名前であり、今後のパッチで Gen13 といったものに変わるのではないかと思われる。  

 * [[Intel-gfx] [PATCH 00/18] Preliminary Display13 support](https://lists.freedesktop.org/archives/intel-gfx/2021-January/259621.html)

*Display13* ではプレーンあたりの容量がこれまでの 32KB から 128KB に拡張され、これにより 16K解像度に対応するようになった。  

 >        +	int max_horizontal_pixels = 8192;
 >        +	int max_stride_bytes;
 >        +
 >        +	if (HAS_DISPLAY13(i915)) {
 >        +		/*
 >        +		 * The stride in bytes must not exceed of the size
 >        +		 * of 128K bytes. For pixel formats of 64bpp will allow
 >        +		 * for a 16K pixel surface.
 >        +		 */
 >        +		max_stride_bytes = 131072;
 >        +		if (cpp == 8)
 >        +			max_horizontal_pixels = 16384;
 >        +	} else {
 >        +		/*
 >        +		 * "The stride in bytes must not exceed the
 >        +		 * of the size of 8K pixels and 32K bytes."
 >        +		 */
 >        +		max_stride_bytes = 32768;
 >        +	}
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 05/18] drm/i915/display13: Support 128k plane stride](https://lists.freedesktop.org/archives/intel-gfx/2021-January/259627.html) {{< /quote >}}

Intel VT-d (Virtualization Technology for Directed I/O ) が有効な場合、ディスプレイに必要なメモリ帯域が 5% 増加するため、それに対応するパッチも含まれている。  

 * [[Intel-gfx] [PATCH 11/18] drm/i915/display13: Required bandwidth increases when VT-d is active](https://lists.freedesktop.org/archives/intel-gfx/2021-January/259629.html)

これは最初のパッチで触れられていた、*Display13* の機能としてあるが、今回の一連のパッチには含まれていない、  
*Tiled Surfaces* を GGTT (Global graphics translation table) にマッピングするようになったことによるものではないかと思われる。*Tiled Surfaces* は、フレームバッファーの各ピクセルをメモリにマッピングする際に用いられる方式のこと。  
そのように変更した理由としては、GPU 仮想技術やそれによる仮想デスクトップに活用するためではないかということが考えられる。  

パワードメインの関係性が以下のように変更された。  
映像出力ポートに対応する Display Pipe (A-D) が A と B-D のグループに分けられている。  
これは、最低限必要な Pipe A とそれ以外に分けることで、パワーゲーディングの適用を簡易にする目的があるのではないかと思われる。また、パイプごとに適用することも可能としている。  


 >        +/*
 >        + * Display13 Power Domains
 >        + *
 >        + * Previous platforms required that PG(n-1) be enabled before PG(n).  That
 >        + * dependency chain turns into a dependency tree on Display13:
 >        + *
 >        + *       PG0
 >        + *        |
 >        + *     --PG1--
 >        + *    /       \
 >        + *  PGA     --PG2--
 >        + *         /   |   \
 >        + *       PGB  PGC  PGD
 >        + *
 >        + * Power wells must be enabled from top to bottom and disabled from bottom
 >        + * to top.  This allows pipes to be power gated independently.
 >        + */
 >        +
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 07/18] drm/i915/display13: Add Display13 power wells](https://lists.freedesktop.org/archives/intel-gfx/2021-January/259628.html) {{< /quote >}}

また、Watermark {{< comple >}} ディスプレイの解像度等に応じて決定されるメモリのタイミング {{< /comple >}} の数が、従来の 32個から 256個へと大幅に増やされた。  
より最適なタイミングを設定することができ、電力効率を向上させられる。  

パッチのコメントにあったように、*Display13* での変更点はそう巨大なものではなく、自然な進化と言えるが、その分着実にディスプレイ部の性能と省電力機能を強化しているとも表せられる。  

{{< ref >}}

 * [Volume 5: Memory Views - intel-gfx-prm-osrc-skl-vol05-memory_views.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-skl-vol05-memory_views.pdf)
 * [Cover_Sheet - intel-gfx-prm-osrc-icllp-vol12-displayengine_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol12-displayengine_0.pdf)

{{< /ref >}}
