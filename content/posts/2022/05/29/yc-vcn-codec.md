---
title: "Yellow Carp/Rembrandt APU の対応コーデックについて"
date: 2022-05-29T12:40:18+09:00
draft: false
categories: [ "Diary", "AMD", "Hardware", "APU" ]
tags: [ "Yellow_Carp", "Rembrandt" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

雑記。  
*Yellow Carp/Rembrandt APU* に搭載されたメディアエンジン (VCN, Video Core Next) の対応コーデック情報について、過去に何度か触れてきた。  
対応コーデック情報は AMD 公式サイト、RadeonFeature、オープンソースドライバーのコード等から得られるが、*Yellow Carp/Rembrandt APU* の情報はそれぞれでわずかに食い違っていた。主に MPEG2, MPEG4, VC1, AV1 のデコード対応について。  
その点を [drm / amd](https://gitlab.freedesktop.org/drm/amd) レポジトリに issue を送ったところ、AMD の Alex Deucher 氏より実質的な回答となるパッチが返ってきた。  

結論から言えば、実際の動作については何の問題も無かったし、このパッチによる影響も特に無い。  

 * [[PATCH] drm/amdgpu: update VCN codec support for Yellow Carp](https://lists.freedesktop.org/archives/amd-gfx/2022-May/079667.html)

 > 		Supports AV1.  Mesa already has support for this and
 > 		doesn't rely on the kernel caps for yellow carp, so
 > 		this was already working from an application perspective.
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: update VCN codec support for Yellow Carp](https://lists.freedesktop.org/archives/amd-gfx/2022-May/079667.html) {{< /quote >}}

まず、Linux Kernel側の AMDGPUドライバーに記述されているコーデック情報は、libdrm uAPI の `amdgpu_query_video_caps_info` が呼ばれたときにコーデック自体への対応と対応サイズを返すのに使われる。  
`amdgpu_query_video_caps_info` を使うのは基本 UMD (User Mode Driver) である Mesa3D のみとなる。そして、以前にも触れたが、Mesa3D では取得した対応コーデック情報の内、サイズのみを使用している。  
{{< link >}} [AMD CES 2022 個人的まとめ | Coelacanth's Dream](/posts/2022/01/05/amd-ces-2022/#av1-dec) {{< /link >}}
libdrm のバージョン違い等で `amdgpu_query_video_caps_info` に対応していない場合でも、AMD APU/GPU ファミリーを元に対応サイズが返される。  

 > 		   case PIPE_VIDEO_CAP_MAX_WIDTH:
 > 		      if (codec != PIPE_VIDEO_FORMAT_UNKNOWN &&
 > 		            sscreen->info.dec_caps.codec_info[codec - 1].valid) {
 > 		         return sscreen->info.dec_caps.codec_info[codec - 1].max_width;
 > 		      } else {
 > 		         switch (codec) {
 > 		         case PIPE_VIDEO_FORMAT_HEVC:
 > 		         case PIPE_VIDEO_FORMAT_VP9:
 > 		         case PIPE_VIDEO_FORMAT_AV1:
 > 		            return (sscreen->info.family < CHIP_RENOIR) ?
 > 		               ((sscreen->info.family < CHIP_TONGA) ? 2048 : 4096) : 8192;
 > 		         default:
 > 		            return (sscreen->info.family < CHIP_TONGA) ? 2048 : 4096;
 > 		         }
 > 		      }
 >
 > {{< quote >}} [src/gallium/drivers/radeonsi/si_get.c · 244305493240ee9e0fc08a9b3da806d47a5cf257 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/244305493240ee9e0fc08a9b3da806d47a5cf257/src/gallium/drivers/radeonsi/si_get.c#L671-685) {{< /quote >}}

同時に RadeonFeature の情報も更新され、*Yellow Carp/Rembrandt APU* 、VCN 3.1 は MPEG4_AVC, HEVC, AV1 のデコードに対応し、MPEG2, MPEG4, VC1 のデコードには非対応ということが確定した。  

 * [RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeonvcnvideocorenexthardware)

AMD 公式サイトの **AMD Ryzen 5 6600U / 7 6800U** の Max Video Decode Bandwidth には MPEG2, VC1 が記載されているが、搭載製品が出回る頃には修正されると思う、多分。  

 * [AMD Ryzen™ 5 6600U | AMD](https://www.amd.com/en/product/11596)
 * [AMD Ryzen™ 7 6800U | AMD](https://www.amd.com/en/product/11591)

結局の所、問題も影響も無い自分の中の話題だったが、確かな情報を得たいときに RadeonFeature やオープンソースドライバーのコードを確認する自分にとってはコーデック情報が明確になって良かったと思う。  
[Tech Docs | AMD](https://www.amd.com/en/support/tech-docs?keyword=&page=0) からダウンロードできるドキュメントは公開されるまで時間が掛かり、公式サイトのスペックページは正直に言って不確かであることがそれなりにあるからだ。  
それに質問や報告する場が、<https://gitlab.freedesktop.org/drm/amd> や <https://gitlab.freedesktop.org/mesa/mesa/-/issues> のようにちゃんとあるのだから、まず聞いてみることが一番だと思った、オープンソースドライバーの情報を嘘だとか "釣り" として扱う前に。  
