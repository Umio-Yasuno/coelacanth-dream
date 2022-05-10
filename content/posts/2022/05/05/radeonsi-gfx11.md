---
title: "RadeonSI ドライバーで GFX11 のサポートが進められる"
date: 2022-05-05T12:48:07+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU", "APU" ]
tags: [ "GFX11", "RadeonSI", "NGG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Marek Olšák 氏より、*RadeonSI (OpenGL)* ドライバーに *GFX11* 世代のサポートを追加するマージリクエストが投稿、公開されている。  
内容としては *GFX11* で従来から変更のあった部分、機能の改良や拡張、あるいは廃止になった部分に対するものが主であり、*GFX11* で実装される新機能のサポートといったものはまだ公開されていないように見えた。  

 * [amd: add gfx11 support (!16328) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/16328)

{{< pindex >}}
 * [命令キャッシュラインサイズ](#inst-cache-line-size)
 * [NGG](#ngg)
 * [VRS](#vrs)
 * [DCC](#dcc)
 * [VCN4](#vcn4)
 * [Chip, Family](#chip_family)
{{< /pindex >}}

## 命令キャッシュラインサイズ {#inst-cache-line-size}

*GFX11* 世代では命令キャッシュラインサイズの変更が行われ、*RDNA 1/2* 世代の 64 bytes から 128 bytes に拡大された。  
命令キャッシュは SQC (Sequencer Cache) であり、複数の CU で共有される。  

キャッシュラインサイズは、*Aldebaran/gfx90a/MI200* を除いた *GFX9/Vega* 世代では、命令キャッシュとスカラデータキャッシュが 32 bytes、L1ベクタキャッシュと L2キャッシュが 64 Bytes、  
*RDNA 1/2* 世代ではそこから、命令キャッシュとスカラデータキャッシュが 64 bytes、L0ベクタキャッシュと GL1キャッシュ、GL2キャッシュが 128 bytes となった。  

 > 		+   if (prefetch_distance) {
 > 		+      if (i.info->chip_class >= GFX11)
 > 		+         binary->rx_size = align(binary->rx_size + prefetch_distance * 64, 128);
 > 		+      else
 > 		+         binary->rx_size = align(binary->rx_size + prefetch_distance * 64, 64);
 > 		+   }
 >
 > {{< quote >}} [radeonsi/gfx11: instruction cache line size is 128 bytes (841d7614) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/841d76145f51ac33ca181cc5a74d3b6f1ab090e8?merge_request_iid=16328) {{< /quote >}}

*GFX11* では L2キャッシュ (TCC, Texture Cache Channel) のラインサイズは 128 bytes とされており[^gfx11-tcc]、SQC である L1命令キャッシュ (それと恐らくは L1スカラデータキャッシュも) のラインサイズのみを拡大した形となっている。  

[^gfx11-tcc]: [amd: add gfx11 support (!16328) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/16328/diffs?commit_id=c0b87b6d33f86f735af2c77f947aaa89d4d734c4)

| Cache Line Size               | L1i | L1k | L1d | GL1 | GL2 |
| :--------------               | :-: | :-: | :-: | :-: | :-: |
| GFX9/Vega (without Aldebaran) | 32B | 32B | 64B | -   | 64B |
| GFX10/RDNA 1, GFX10.3/RDNA 2    | 64B | 64B | 128B | 128B | 128B |
| GFX11                         | 128B | 128B? | 128B? | 128B? | 128B |

## NGG {#ngg}

*GFX11* では *GFX9/Vega* までのレガシーなグラフィクスパイプラインが廃され、NGG (Next Generation Geometry) のみをサポートする。  

 * [radeonsi/gfx11: add assert in legacy vs path (b2669445) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/b2669445aa29d6ce34e3e4ee92324ada40f69f7c?merge_request_iid=16328)
 * [radeonsi/gfx11: enable NGG-only draw paths (4e9915b5) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/4e9915b5e5c9ecc3c354a6dda916253aaf07138d?merge_request_iid=16328)

NGG は *GFX10/RDNA 1* からデフォルトで有効化されているが、 *Navi14 (GFX10/RDNA 1)* では無効化されており、またデバッグオプションによって切替可能な機能だった。  
*GFX10.3/RDNA 2* 世代ではすべてのチップでデフォルトで有効化されるという段階を踏み、*GFX11* 世代でレガシーグラフィクスパイプラインを廃し、NGG への完全な置き換えに至った。  
またこれまでは無効化されていた NGG Stream-Out (transform feedback) も *GFX11* ではデフォルトで有効化される。  

現時点で *GFX11* の NGG Culling はデフォルトで無効化されているが、これは *GFX10/RDNA 1* 以上の世代でも、RenderBackend (RB) が 2基以上という条件がデフォルトで有効化する条件に含まれており、GPU の規模によって効果に違いがあることが理由だろう。  
NGGカリングのようなシェーダーベースのカリング処理は、効率が Pixel Shader のスループットに依存するとされている。  
{{< link >}} [RADV + RDNA 2 で NGGカリングがデフォルトで有効に | Coelacanth's Dream](/posts/2021/09/29/radv-enable-nggc-default-on-rdna_2/#nggc) {{< /link >}}
内部での検証が進み、効果があると分かれば切り替えられると思われる。  

 > 		+   if (sscreen->info.chip_class >= GFX11) {
 > 		+      sscreen->use_ngg = true;
 > 		+      sscreen->use_ngg_streamout = true;
 > 		+      /* TODO: Disable for now. Investigate if it helps.  */
 > 		+      sscreen->use_ngg_culling = (sscreen->debug_flags & DBG(ALWAYS_NGG_CULLING_ALL)) &&
 > 		+                                 !(sscreen->debug_flags & DBG(NO_NGG_CULLING));
 > 		+   } else {
 > 		+      sscreen->use_ngg = !(sscreen->debug_flags & DBG(NO_NGG)) &&
 > 		+                         sscreen->info.chip_class >= GFX10 &&
 > 		+                         (sscreen->info.family != CHIP_NAVI14 ||
 > 		+                          sscreen->info.is_pro_graphics);
 > 		+      sscreen->use_ngg_streamout = false;
 > 		+      sscreen->use_ngg_culling = sscreen->use_ngg &&
 > 		+                                 sscreen->info.max_render_backends >= 2 &&
 > 		+                                 !(sscreen->debug_flags & DBG(NO_NGG_CULLING)) &&
 > 		+                                 LLVM_VERSION_MAJOR >= 12; /* hangs on 11, see #4874 */
 > 		+   }
 >
 > {{< quote >}} [radeonsi/gfx11: enable NGG-only draw paths (4e9915b5) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/4e9915b5e5c9ecc3c354a6dda916253aaf07138d?merge_request_iid=16328) {{< /quote >}}

Pixel Shader への入出力用バッファ、パラメーターキャッシュ (Parameter Cache, PC) の規模は 1024、*Sienna Cichlid/Navi21* 等と同じ規模となっている。  

 > 		+   if (info->chip_class >= GFX11) {
 > 		+      info->pc_lines = 1024;
 > 		+      info->pbb_max_alloc_count = 255; /* minimum is 2, maximum is 256 */
 >
 > {{< quote >}} [radeonsi/gfx11: scattered register deltas (9fecac09) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/9fecac091f3159eb50a3e3dea2312218bb87d8c1?merge_request_iid=16328) {{< /quote >}}

## VRS {#vrs}

*GFX10.3/RDNA 2* 世代における VRS (Variable Rate Shading) は、粒度 (coarse) 4 pixels (2x2) までのサポートだったが、*GFX11* では 16 pixels (4x4) に対応する。  

 * [radeonsi/gfx11: VRS changes (18a560dd) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/18a560dd45ae3157b2283cb35f86b9e3303565c7?merge_request_iid=16328)
 
## DCC {#dcc}

*GFX11* では DCC (Delta Color Compression) が改良され、これまではフォーマットのサポートが部分的だったが *GFX11* ですべてのフォーマットをサポートするようになった。  
また、`always_allow_dcc_stores` フラグは、dGPU では性能低下のリスクがあることから、*GFX10.3/RDNA 2* 世代の APU でのみデフォルトで有効化されていたが、*GFX11* では dGPU, APU 問わずデフォルトで有効化される。  

 * [radeonsi/gfx11: always allow DCC stores (f81dafdb) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/f81dafdb75081b2af260f5d3082d7601e2a941a1?merge_request_iid=16328)
 * [radeonsi/gfx11: enable arbitrary DCC format reinterpretation (2ea6b525) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/2ea6b5253516aea3945aa11c5200d6eedbd32a41?merge_request_iid=16328)

 > 		   /* DCC stores have 50% performance of uncompressed stores and sometimes
 > 		    * even less than that. It's risky to enable on dGPUs.
 > 		    */
 > 		   sscreen->always_allow_dcc_stores = !(sscreen->debug_flags & DBG(NO_DCC_STORE)) &&
 > 		                                      (sscreen->debug_flags & DBG(DCC_STORE) ||
 > 		                                       sscreen->info.chip_class >= GFX11 || /* always enabled on gfx11 */
 > 		                                       (sscreen->info.chip_class >= GFX10_3 &&
 > 		                                        !sscreen->info.has_dedicated_vram));
 >
 > {{< quote >}} [src/gallium/drivers/radeonsi/si_pipe.c · f81dafdb75081b2af260f5d3082d7601e2a941a1 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/f81dafdb75081b2af260f5d3082d7601e2a941a1/src/gallium/drivers/radeonsi/si_pipe.c#L1294-1301) {{< /quote >}}
 >
 > 		+   /* All formats are compatible on GFX11. */
 > 		+   if (sscreen->info.chip_class >= GFX11)
 > 		+      return true;
 > 		+
 >
 > {{< quote >}} [radeonsi/gfx11: enable arbitrary DCC format reinterpretation (2ea6b525) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/2ea6b5253516aea3945aa11c5200d6eedbd32a41?merge_request_iid=16328) {{< /quote >}}

## VCN4 {#vcn4}

VCN4 では MPEG1/2, MPEG4, VC1 の HWデコードをサポートをしないため、それぞれの部分に `chip_class` が *GFX11* 以上の場合は `false` を返す記述が追加されている。  

 > 		       case PIPE_VIDEO_FORMAT_MPEG12:
 > 		-         return profile != PIPE_VIDEO_PROFILE_MPEG1;
 > 		+         if (sscreen->info.chip_class >= GFX11)
 > 		+            return false;
 > 		+         else
 > 		+            return profile != PIPE_VIDEO_PROFILE_MPEG1;
 > 		       case PIPE_VIDEO_FORMAT_MPEG4:
 > 		-         return 1;
 > 		+         if (sscreen->info.chip_class >= GFX11)
 > 		+            return false;
 > 		+         else
 > 		+            return true;
 >
 > {{< quote >}} [radeonsi/gfx11: update codec support for gfx11 (b8b88fb0) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/b8b88fb0c2490452ff230548c555bda2b61895d4?merge_request_iid=16328) {{< /quote >}}
 >
 > 		       case PIPE_VIDEO_FORMAT_VC1:
 > 		-         return true;
 > 		+         if (sscreen->info.chip_class >= GFX11)
 > 		+            return false;
 > 		+         else
 > 		+            return true;
 >
 > {{< quote >}} [radeonsi/gfx11: update codec support for gfx11 (b8b88fb0) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/b8b88fb0c2490452ff230548c555bda2b61895d4?merge_request_iid=16328) {{< /quote >}}

だがそれよりも上の部分で、`[MPEG12, MPEG4, VC1]` かつ `family` が *Beige Goby/Navi24* 以上 (*Beige Goby/Navi24, Yellow Carp/Rembrandt, GFX1036*) の場合 `false` を返すようになっているため、不要な気もする。  

 > 		   switch (param) {
 > 		   case PIPE_VIDEO_CAP_SUPPORTED:
 > 		      if (codec < PIPE_VIDEO_FORMAT_MPEG4_AVC &&
 > 		          sscreen->info.family >= CHIP_BEIGE_GOBY)
 > 		         return false;
 >
 > {{< quote >}} [src/gallium/drivers/radeonsi/si_get.c · b8b88fb0c2490452ff230548c555bda2b61895d4 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/b8b88fb0c2490452ff230548c555bda2b61895d4/src/gallium/drivers/radeonsi/si_get.c#L604-613) {{< /quote >}}

VCN4 では AV1デコーダー部が従来のものからアップデートされたような記述がある。  
ただ、AMDGPUドライバーへの VCN4 に関するパッチによれば、現時点ではデコーダーが対応するピクセルサイズは VCN3 から変更はない。[^vcn4-codec]  

[^vcn4-codec]: [[PATCH 10/11] drm/amdgpu: add vcn_4_0_0 video codec query](https://lists.freedesktop.org/archives/amd-gfx/2022-May/078569.html)

 > 		+#define RDECODE_AV1_VER_0  0
 > 		+#define RDECODE_AV1_VER_1  1
 > 		+
 >
 > {{< quote >}} [radeonsi/vcn: update av1 decoding to support vcn4 (88e84200) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/88e842002b6f02ae891f93bc4e21553ed20c7657?merge_request_iid=16328) {{< /quote >}}
 >
 > 		    case CHIP_GFX1102:
 > 		       dec->jpg.direct_reg = true;
 > 		       dec->addr_gfx_mode = RDECODE_ARRAY_MODE_ADDRLIB_SEL_GFX11;
 > 		+      dec->av1_version = RDECODE_AV1_VER_1;
 > 		       break;
 >
 > {{< quote >}} [radeonsi/vcn: update av1 decoding to support vcn4 (88e84200) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/88e842002b6f02ae891f93bc4e21553ed20c7657?merge_request_iid=16328) {{< /quote >}}

## Chip, Family {#chip_family}

GC (Graphics Compute) IP のバージョンと、LLVM 等で用いられる GPU/GFX/TargetID は近いフォーマットながら一致は以前からしていなかったが、この世代でもそれは同様らしい。  

現時点で AMDGPUドライバーには *GC 11.0.{0, 1, 2}* のサポートを追加するパッチが投稿されているが、対応する GFX ID の対応は以下の表のようになっている。  

| GC IP ver | GFX ID  |      |
| :-------- | :-----: | :--: |
| 11.0.0    | gfx1100 | dGPU |
| 11.0.1    | gfx1103 | APU  |
| 11.0.2    | gfx1102 | dGPU |
| 11.0.3?   | gfx1101? | dGPU? |

下引用部で、チップの判定に使われる `eRevisionId` の範囲がそのままの並びになっていないが、*RadeonSI* では GFX ID をベースにした名前を用いていることと、先に書いたように GC IP と GFX IP が必ず一致はしないことが関係していると考えられる。  

 > 		+#define AMDGPU_GFX1100_RANGE    0x01, 0x10
 > 		+#define AMDGPU_GFX1101_RANGE    0x20, 0xFF
 > 		+#define AMDGPU_GFX1102_RANGE    0x10, 0x20
 > 		+
 > 		+#define AMDGPU_GFX1103_RANGE    0x01, 0xFF
 > 		+
 >
 > {{< quote >}} [amd: add gfx11 support (!16328) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/16328/diffs?commit_id=8e6d599238881bc63590457813dc8532cb5462ad) {{< /quote >}}

{{< ref >}}
 * [[PATCH] drm/amdkfd: add asic support for GC 11.0.2](https://lists.freedesktop.org/archives/amd-gfx/2022-May/078624.html)
 * [[PATCH] drm/amdkfd: add asic support for GC 11.0.2](https://lists.freedesktop.org/archives/amd-gfx/2022-May/078624.html)
 * [AMD PowerPoint- White Template - RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf)
{{< /ref >}}
