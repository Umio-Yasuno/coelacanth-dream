---
title: "Cyan Skilfish APU のディスプレイエンジンをサポートするパッチが投稿される　―― NAVI12_LITE"
date: 2021-09-28T05:37:21+09:00
draft: false
tags: [ "Cyan_Skilfish", "RDNA", "Linux_Kernel", "gfx1013" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

RDNA APU *Cyan Skilfish* のディスプレイエンジン/コントローラー **DCN 2.01** をサポートするパッチが amd-gfx メーリングリストに投稿された。  

 * [[PATCH 00/02] cyan skillfish display support](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069481.html)
    * [[PATCH 01/02] drm/amdgpu: add cyan_skillfish asic header files](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069482.html)
    * [[PATCH 02/02] drm/amd/display: add cyan_skillfish display support](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069483.html)
    * [[PATCH 02/02 v2] drm/amd/display: add cyan_skillfish display support](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069486.html)

*Cyan Skilfish* をサポートするパッチは以前に投稿されていたが、その時はディスプレイエンジンのサポートを含まれていなかった。  
{{< link >}} [Linux Kernel に RDNA APU 「Cyan Skilfish」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/07/21/amd-cyan_skilfish-rdna-apu/) {{< /link >}}
今回サポートが追加されたことで、ディスプレイ出力が有効化された *Cyan Skilfish* ベースの製品が出てくる可能性が見えてきた。  

また、*Cyan Skilfish* のメディアエンジンについてはまだ有効化、サポートはされていない。  

{{< pindex >}}
 * [DCN 2.01](#dcn201)
 * [NAVI12_LITE](#navi12_lite)
{{< /pindex >}}

## DCN 2.01 {#dcn201}

*Navi1x* はディスプレイエンジンに **DCN 2.0** を搭載しており、*Cyan Skilfish* のはそこからステッピングが進んだバージョンとなる。  
**DCN 2.01** は最大同時画面出力数が 2画面となっており、他 APU ではほとんどが 4画面となっていることを考えれば少ない方だと言える。  

 > 		+       case CHIP_CYAN_SKILLFISH:
 > 		+               adev->mode_info.num_crtc = 2;
 > 		+               adev->mode_info.num_hpd = 2;
 > 		+               adev->mode_info.num_dig = 2;
 > 		+               break;
 >
 > {{< quote >}} [[PATCH 02/02] drm/amd/display: add cyan_skillfish display support](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069483.html) {{< /quote >}}

メモリには、ステイトの `.dram_speed_mts` の最大値から GDDR6 〜14Gbps、`.num_chans` から 256-bit (16ch * 16-bit) を想定していると思われる。  

 > 		+                       {
 > 		+                               .state = 3,
 > 		+                               .dscclk_mhz = 400.0,
 > 		+                               .dcfclk_mhz = 1000.0,
 > 		+                               .fabricclk_mhz = 250.0,
 > 		+                               .dispclk_mhz = 1200.0,
 > 		+                               .dppclk_mhz = 1200.0,
 > 		+                               .phyclk_mhz = 810.0,
 > 		+                               .socclk_mhz = 1254.0,
 > 		+                               .dram_speed_mts = 14000.0,
 > 		+                       },
 > 		+                       {
 > 		+                               .state = 4,
 > 		+                               .dscclk_mhz = 400.0,
 > 		+                               .dcfclk_mhz = 1000.0,
 > 		+                               .fabricclk_mhz = 750.0,
 > 		+                               .dispclk_mhz = 1200.0,
 > 		+                               .dppclk_mhz = 1200.0,
 > 		+                               .phyclk_mhz = 810.0,
 > 		+                               .socclk_mhz = 1254.0,
 > 		+                               .dram_speed_mts = 14000.0,
 > 		+                       }
 > {{< quote >}} [[PATCH 02/02] drm/amd/display: add cyan_skillfish display support](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069483.html) {{< /quote >}}

 > 		+       .channel_interleave_bytes = 256,
 > 		+       .num_banks = 8,
 > 		+       .num_chans = 16,
 > 		+       .vmm_page_size_bytes = 4096,
 >
 > {{< quote >}} [[PATCH 02/02] drm/amd/display: add cyan_skillfish display support](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069483.html) {{< /quote >}}

## NAVI12_LITE {#navi12_lite}

今回のパッチで明らかになった訳ではなく、自分が今まで気付いていなかっただけだが、他に取り上げている人もいなかった。  

AMD の GPU ASIC には、`external_rev_id/eChipRev` という ID が GPUファミリー内でそれぞれ割り当てられており、*Cyan Skilfish* のそれは `0x82 (130)` となっている。  

 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		+		adev->cg_flags = 0;
 > 		+		adev->pg_flags = 0;
 > 		+		adev->external_rev_id = adev->rev_id + 0x82;
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 15/29] drm/amdgpu: add chip early init for cyan_skillfish](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066818.html) {{< /quote >}}

そして、後に変更が加えられメインラインにマージされなかった内容ではあるが、2019/06 に投稿、公開された *Navi10* をサポートするパッチの一部によれば、`external_rev_id/eChipRev: 0x82 (130)` は *NV_NAVI12_LITE_P_A0* と一致する。  

 > 		+#define FAMILY_NV 143 /* DCN 2*/
 > 		+
 > 		+enum {
 > 		+	NV_NAVI10_P_A0      = 1,
 > 		+	NV_NAVI12_P_A0      = 10,
 > 		+	NV_NAVI14_M_A0      = 20,
 > 		+	NV_NAVI21_P_A0      = 40,
 > 		+	NV_NAVI10_LITE_P_A0 = 0x80,
 > 		+	NV_NAVI10_LITE_P_B0 = 0x81,
 > 		+	NV_NAVI12_LITE_P_A0 = 0x82,
 > 		+	NV_NAVI21_LITE_P_A0 = 0x90,
 > 		+	NV_UNKNOWN          = 0xFF
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH 316/459] drm/amd/display: Add DCN2 and NV ASIC ID](https://lists.freedesktop.org/archives/amd-gfx/2019-June/035497.html) {{< /quote >}}

何らかの理由で *NAVI12_LITE* を *Cyan Skilfish* というコードネームに変更したか、新たに付けたと思われる。  
ただ、*Cyan Skilfish (gfx1013)* はレイトレーシング命令を除いて、*Navi10 (gfx1010)* をベースにした範囲の命令に対応しており、*Navi12 (gfx1011)/ Navi14 (gfx1012)* がサポートする一部ドット積命令 (`dot[1,2,5-7]-insts`) には対応していない。  
{{< link>}} [HWレイトレーシングをサポートする RDNA APU 「Cyan Skilfish (gfx1013)」 | Coelacanth's Dream](/posts/2021/08/01/cyan_skilfish-apu-gfx1013/#gfx1013) {{< /link >}}
そのため LITE と付いてはいるが *Cyan Skilfish* が元々 *NAVI12_LITE* だったというのは若干違和感を覚える。  

とはいえ *LITE* と付く AMD APU/GPU については、メインラインに組み込まれなかったように、オープンソースドライバーでは対象にしていない GPU であり、情報に乏しい。  
そこを超えて情報を集める気は今はあまり無く、また何かしら楽しむためには線引きすることが大切だと思うようになった。  
