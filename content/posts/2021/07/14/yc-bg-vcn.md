---
title: "他 RDNA 2 GPU とは対応コーディックが異なる Yellow Carp APU と Beige Goby GPU"
date: 2021-07-14T16:49:33+09:00
draft: false
tags: [ "Yellow_Carp", "Beige_Goby", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
noindex: false
# summary: ""
---

*RDNA 2 アーキテクチャ* を採用し、マルチメディアエンジンには VCN 3.0 を搭載する *Yellow Carp APU* 、*Beige Goby GPU* だが、最近になって Linux Kernel における AMD GPU ドライバーに投稿されたパッチはそれら APU/GPU が、同じく VCN 3.0 を搭載する他 *RDNA 2 APU/GPU* から一部機能が削られていることを示している。  

 * [[PATCH 0/2] amdgpu video codec support addition and code optimization](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066547.html)
    * [[PATCH 1/2] amdgpu/nv.c - Added video codec support for Yellow Carp](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066548.html)
    * [[PATCH 2/2] amdgpu/nv.c - Optimize code for video codec support structure](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066549.html)

{{< pindex >}}
 * [AV1 等、一部コーディックのデコード機能が削られている Yellow Carp](#yc)
 * [エンコード機能を持たず、デコード機能も一部削られている Beige Goby](#bg)
 * [VCN 3.0](#vcn_3)
{{< /pindex >}}

## AV1 等、一部コーディックのデコード機能が削られている Yellow Carp {#yc}

*Yellow Carp APU* は、オープンソースドライバー上では *VanGogh APU* の次に登場した *RDNA 2 APU* だが、マルチメディアエンジンが対応するコーディックは *VanGogh* よりも少ない。  

 * [[PATCH 1/2] amdgpu/nv.c - Added video codec support for Yellow Carp](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066548.html)

下引用部は、*Sienna Cichlid* と *Yellow Carp* がハードウェアデコードに対応するコーディックを示したドライバー中のコード部。  
*Yellow Carp* では、*RDNA 2 アーキテクチャ* の特徴としてアピールされていた AV1 HWデコードをサポートせず[^rdna_2]、それ以外にも MPEG2・MPEG4・VC1 がサポートされていない。  

[^czn]: [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/)
[^rdna_2]: [RDNA 2 Architecture | One Gaming DNA | AMD](https://www.amd.com/en/technologies/rdna-2)

 > ##### Sienna Cichlid (sc_video_codecs_decode_array)
 >
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_MPEG2, 4096, 4906, 3)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_MPEG4, 4096, 4906, 5)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_MPEG4_AVC, 4096, 4906, 52)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_VC1, 4096, 4906, 4)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_HEVC, 8192, 4352, 186)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_JPEG, 4096, 4096, 0)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_VP9, 8192, 4352, 0)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_AV1, 8192, 4352, 0)},
 >
 > {{< quote >}} [[PATCH 2/2] amdgpu/nv.c - Optimize code for video codec support structure](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066549.html) {{< /quote >}}
 >
 > 		+/* Yellow Carp*/
 > 		+static const struct amdgpu_video_codec_info yc_video_codecs_decode_array[] = {
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_MPEG4_AVC, 4096, 4906, 52)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_HEVC, 8192, 4352, 186)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_VP9, 8192, 4352, 0)},
 > 		+	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_JPEG, 4096, 4096, 0)},
 > 		+};
 >
 > {{< quote >}} [[PATCH 1/2] amdgpu/nv.c - Added video codec support for Yellow Carp](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066548.html) {{< /quote >}}

これにより *Yellow Carp* は 8Kサイズの HWデコードには対応するが、対応コーディックの幅は前世代の *Renoir/Lucienne/Cezanne (Green Sardine) APU* [^czn] よりも狭まることとなる。  
だがそれら MPEG2・MPEG4・VC1 コーディックは現在ではほとんど使われておらず、H.264/MPEG-4 AVC・H.265/HEVC・VP9 といった広く普及し、主流となっているコーディックの対応は残されている。一部コーディックの削除は、活用する機会が少ない無駄な固定機能を省き、ダイサイズを最適化するため行ったのかもしれない。  

ただ、普及しつつある AV1 HWデコードを省いたのは少し疑問に思え、またそれ以上に惜しいと感じる部分だ。 Intel Gen12 GPUアーキテクチャを採用する [Tiger Lake](/tags/tiger_lake)・[Rocket Lake](/tags/rocket_lake)・[Alder Lake](/tags/alder_lake) は AV1 HWデコードに対応しているため、それらと *Yellow Carp* を比較した時、弱点になるとも言える。  
{{< link >}} [Intel Gen12 GPU は AV1コーディックのHWデコードをサポート | Coelacanth's Dream](/posts/2020/07/09/intel-gen12-av1-decode/) {{< /link >}}

また、*VanGogh* が GPU L2キャッシュ 1MiB を持つのに対し、*Yellow Carp* はその倍のサイズ、2MiB を持つとされている。  
このことから *Yellow Carp* は、*VanGogh* よりも大きい規模のメモリインターフェイス、GPUコアを持つ可能性が考えられるが、その一方でマルチメディアエンジン部は *VanGogh* のが {{< comple >}} どれだけそのコーディックが使われているかはともかく {{< /comple >}} 多機能だということになる。コード中では、*VanGogh* は *Sienna Cichlid* と同じコーディックに対応するとしている。  
{{< link >}} [RDNA 2 APU 「Yellow Carp」 をサポートするパッチが Linux Kernel に投稿される　―― DCN3.1 / VanGogh より大きいキャッシュ | Coelacanth's Dream](/posts/2021/06/03/yellow_carp-apu-linux-kernel/#yc-cache) {{< /link >}}
設計時期、設計に掛けられる期間によるものか、あるいは実装コストか、AV1 HWデコードを削ってその分を GPUコア部に割いた方が有益と判断したか。  

可能性をただただ並べても仕方ないが、オープンソースドライバーでは将来の GPU をサポートするためのパッチが公開されている分、今後変更されることがある、ということだけは留意したい。  

## エンコード機能を持たず、デコード機能も一部削られている Beige Goby {#bg}

*Beige Goby* が HWエンコード機能を持たないことは、AMD GPUドライバーにサポートを追加する最初のパッチで既に示されていたが、HWデコードに関しても、*Yellow Carp* 同様 MPEG2・MPEG4・VC1 コーディック、それと JPEG にも対応しないことが明かされた。  
{{< link >}} [新たな RDNA 2 GPU、「Beige Goby」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/05/13/amd-beige_goby/#vcn3) {{< /link >}}

 * [amdgpu/nv.c - Added codec query for Beige Goby · torvalds/linux@b3a2446](https://github.com/torvalds/linux/commit/b3a24461f9fb1579c3335c63d1e039bc5a6eda53#diff-29095eeae87881c774274edb9172812d9b05002a942494ba525a9198d56bacf0)

対応コーディックの削減については、上記 *Yellow Carp* と同じ理由しか自分には思い付かないため省略する。  

*Yellow Carp* とは異なり、JPEG HWデコードのサポートも外されていることは、*Renoir APU* 、*Navi1x GPU* に搭載されている前世代の VCN 2.0 から、IPブロックを VCN と JPEGデコーダーとで分けられたため、JPEGデコーダーを省いた構成は以前から想定していたのではないかと思われる。  
省いたこと自体からは、*Beige Goby* が dGPU としては切り詰めた構成を採っている、という印象を受ける。  

 > 		/* Beige Goby*/
 > 		static const struct amdgpu_video_codec_info bg_video_codecs_decode_array[] = {
 > 			{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_MPEG4_AVC, 4096, 4906, 52)},
 > 			{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_HEVC, 8192, 4352, 186)},
 > 			{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_VP9, 8192, 4352, 0)},
 > 		};
 >
 > {{< quote >}} [linux/nv.c at b3a24461f9fb1579c3335c63d1e039bc5a6eda53 · torvalds/linux](https://github.com/torvalds/linux/blob/b3a24461f9fb1579c3335c63d1e039bc5a6eda53/drivers/gpu/drm/amd/amdgpu/nv.c#L319) {{< /quote >}}

[^jpeg-dec]: [drm/amdgpu: add JPEG IP block type · torvalds/linux@8d1b04a](https://github.com/torvalds/linux/commit/8d1b04a6a1dc654d36ab51211fba7af86f0940e6)

[Sienna Cichlid/Navi21](/tags/sienna_cichlid)・[Navy Flounder/Navi22](/tags/navy_flounder)・[Dimgrey Cavefish/Navi23](/tags/dimgrey_cavefish)・[Beige Goby](/tags/beige_goby)、[VanGogh APU](/tags/vangogh)・[Yellow Carp APU](/tags/yellow_carp) ――、オープンソースドライバーに *RDNA 2 アーキテクチャ* を採用する GPU/APU のサポートが増えるにつれ、規模以外で他とは異なる部分が見えてきた。  
*Yellow Carp APU* は他よりも進んだ DCN 3.1 を採用し、そして *Yellow Carp* と *Beige Goby* のマルチメディアエンジンは一部機能が削られている。  
一方で [LLVM](/tags/llvm) にも *GFX10.3/gfx103x* のサポートが追加されてきているが、現時点で ISA、機能が削られているもの {{< comple >}} 特にハードウェアレイトレーシングに関する命令等 {{< /comple >}} は見当たらない。  
こうした違いからは、GPUコア部のアーキテクチャを統一し、シェーダーコンパイラー等において性能最適化に関わる部分は変えず、代わりにマルチメディアエンジンやディスプレイエンジンといった固定機能部を変更することでターゲット帯に合わせて最適化するという方針が見えてくる。  
これまで通り GPUコア部の規模の変更に加えて固定機能部を変更することで、省電力機能、ダイサイズといった面でより進んだ最適化が可能になると考えられる。  

## VCN 3.0 {#vcn_3}

| VCN 3.0<br>(D = Decode, E = Encode) | Sienna Cichlid,<br>Navy Flounder,<br>Dimgrey Cavefish,<br>VanGogh | Beige Goby | Yellow Carp |
| :-- | :--: | :--: | :--: |
| MPEG2 | D | - | - |
| MPEG4 | D | - | - |
| VC1   | D | - | - |
| MPEG4 AVC | D/E | D | D/E |
| HEVC | D/E | D | D/E |
| VP9 | D | D | D |
| JPEG | D | - | D |
| AV1 | D | - | - |

{{< ref >}}
 * [RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeonvcnvideocorenexthardware)
{{< /ref >}}
