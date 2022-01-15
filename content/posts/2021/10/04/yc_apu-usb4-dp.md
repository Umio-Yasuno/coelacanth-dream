---
title: "Yellow Carp APU は USB4 DPトンネリングをサポート"
date: 2021-10-04T23:55:02+09:00
draft: false
tags: [ "Yellow_Carp", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

Linux Kernel における AMD GPUドライバーに、*Yellow Carp/Rembrandt APU* の USB4 DPトンネリング機能をサポートするパッチが amd-gfx メーリングリストに投稿された。  

 * [[PATCH 00/23] USB4 DP tunneling](https://lists.freedesktop.org/archives/amd-gfx/2021-October/069794.html)

2021/08 には、AMD の USB4 ホストルーターをサポートするパッチが Linux-USB メーリングリストに投稿されており、時期を考えれば、*Yellow Carp* を想定したものと思われる。  
{{< link >}} [AMD の USB4 ホストルーターをサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/08/12/amd-usb4/) {{< /link >}}

USB4 DPIA (DisplayPort Input Adapter) の数は *Yellow Carp B0 Stepping* で 4基としており、これは最大画面出力数と一致する。実際にはボードの実装によると思うが、スペックとしては 4画面すべてを USB4 から出力することも可能かもしれない。  
{{< link >}} [RDNA 2 APU 「Yellow Carp」 をサポートするパッチが Linux Kernel に投稿される　―― DCN3.1 / VanGogh より大きいキャッシュ | Coelacanth's Dream](/posts/2021/06/03/yellow_carp-apu-linux-kernel/#dcn3_1) {{< /link >}}

 > 		+	if (dc->ctx->asic_id.chip_family == FAMILY_YELLOW_CARP &&
 > 		+	    dc->ctx->asic_id.hw_internal_rev == YELLOW_CARP_B0) {
 > 		+		/* YELLOW CARP B0 has 4 DPIA's */
 > 		+		pool->base.usb4_dpia_count = 4;
 > 		+	}
 > 		+
 >
 > {{< quote >}} [[PATCH 02/23] drm/amd/display: USB4 DPIA enumeration and AUX Tunneling](https://lists.freedesktop.org/archives/amd-gfx/2021-October/069796.html) {{< /quote >}}

また、*Yellow Carp APU* が搭載するディスプレイエンジン *DCN 3.1* は DP 2.0 をサポートするとされている。  
{{< link >}} [[Yellow Carp APU では DisplayPort 2.0 をサポートか | Coelacanth's Dream](/posts/2021/08/17/dcn31-dp2_0/)] {{< /link >}}
AMD APU では *Vega/GFX9 アーキテクチャ* の採用が続き、ディスプレイエンジン部、I/O部においても、*Renoir* から *Green Sardine/Cezanne* では特に変わらない等、機能面で大きな変更点は見られなかったが、  
*Yellow Carp/Rembrandt* では *RDNA 2/GFX10.3 アーキテクチャ* を採用し、ディスプレイ部、I/O部も同時に大きく進化することとなる。  
人感センサーといったディスプレイ機能に対する補助機能も搭載する兆しも見られ、より高性能、多機能な APU として *Yellow Carp/Rembradt* が登場することが期待される。  
{{< link >}} [AMD の次世代モバイルプラットフォームでは人感センサーをサポート | Coelacanth's Dream](/posts/2021/06/27/amd-hpd-next-gen-platform/) {{< /link >}}
