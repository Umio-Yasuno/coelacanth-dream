---
title: "RADV + RDNA 2 で NGGカリングがデフォルトで有効に"
date: 2021-09-29T06:17:06+09:00
draft: false
tags: [ "RDNA_2", "NGG", "RADV" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

以前、*RADV* では *RDNA 2/GFX10.3* 世代で *NGGカリング/プリミティブカリング* をデフォルトで有効化するつもりがあることに触れたが、[Phoronix](https://www.phoronix.com/) の Michael Larabel 氏による検証で安定したパフォーマンス向上が確認できたとし、[Timur Kristóf](https://gitlab.freedesktop.org/Venemo) 氏によってそれを適用するパッチ/マージリクエストが投稿されている。  
{{< link >}} [RadeonSI ドライバーから非同期コンピュートによるプリミティブカリング機能が削除 | Coelacanth's Dream](/posts/2021/09/21/radeonsi-remove-prim-discard/) {{< /link >}}

 * [radv: Enable NGG culling by default on GFX10.3 (!13086) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13086)

## NGG/NGGカリング {#nggc}

*NGG/プリミティブシェーダー* は *Navi14* を除く *RDNA/GFX10* 世代の GPU でデフォルトで有効化されているが、*NGGカリング* は条件が増え、*RDNA 2/GFX10.3* とそれ以降の世代の GPU、かつ RenderBackend (RB) を 2基以上持つ GPU がデフォルトで有効化の対象になる。  
*RDNA/GFX10* 世代は *NGGカリング* がデフォルトで有効化されないが、環境変数 `RADV_PERFTEST=nggc` をセットすることで有効化できる。  

 > 		+   device->use_ngg_culling =
 > 		+      device->use_ngg &&
 > 		+      device->rad_info.max_render_backends > 1 &&
 > 		+      (device->rad_info.chip_class >= GFX10_3 ||
 > 		+       (device->instance->perftest_flags & RADV_PERFTEST_NGGC)) &&
 > 		+      !(device->instance->debug_flags & RADV_DEBUG_NO_NGGC);
 > 		+
 >
 > {{< quote >}} [radv: Enable NGG culling by default on GFX10.3 (!13086) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13086) {{< /quote >}}

判定部からして RB が 1基の場合は有効化が不可能となっているが、この点は以前から変わらない。  
現時点ではまだ対象となる RB 1基の *RDNA 2/GFX10.3* APU/GPU は登場していないが、小規模な *Zen 2 + RDNA 2 APU* [VanGogh](/rags/vangogh) が該当する可能性はある。[^dcc]  

[^dcc]: [ac/surface: Expose modifiers capable of DCC image stores first (!13056) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13056/diffs?commit_id=39f5be7d6ed133659160a791c9c752b99f8bcc28)

シェーダーベースのカリング処理は、効率が Pixel Shader (PS) のスループットに依存するとされており、*RADV* では GPU の規模、構成から PS への最大入力パラメーター数を決定している。[^ps-param]  
RB 1基の場合、*NGGカリング* を有効化できないようにされているのも、効率が低く、かえって性能に悪影響が与える可能性があるからなのかもしれない。  

[^ps-param]:  [src/amd/vulkan/radv_shader.c · 576d56780cad1e26dd401ef09318a85dd64841f3 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/576d56780cad1e26dd401ef09318a85dd64841f3/src/amd/vulkan/radv_shader.c#L899)

 > 		-   if (max_render_backends < 2)
 > 		-      return false; /* Don't use NGG culling on 1 RB chips. */
 > 		-   else if (max_render_backends / max_se == 4)
 > 		+   if (max_render_backends / max_se == 4)
 > 		       max_ps_params = 6; /* Sienna Cichlid and other GFX10.3 dGPUs. */
 > 		    else
 > 		       max_ps_params = 4; /* Navi 1x. */
 >
 > {{< quote >}} [radv: Enable NGG culling by default on GFX10.3 (!13086) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13086) {{< /quote >}}

### Parameter Cache (PC) {#pc}

*NGG* は、Pixel Shader への入力用オンチップバッファ、パラメーターキャッシュ (Parameter Cache, PC) を搭載している。  
コードから、パラメーターキャッシュの規模は *RDNA 2 dGPU (Sienna Cichlid/Navi21, Navy Flounder/Navi22, Dimgrey Cavefish/Navi23)* と *RDNA 1 dGPU (Navi10/Navi12)* で同じものとなっている。  
それでいて上記 `max_ps_params` は *RDNA 2 dGPU* の方が大きく設定されているため、*RDNA 1 -> RDNA 2* で *NGG* 部が改良されていると考えられる。  

 > 		   if (info->chip_class >= GFX9 && info->has_graphics) {
 > 		      unsigned pc_lines = 0;
 > 		
 > 		      switch (info->family) {
 > 		      case CHIP_VEGA10:
 > 		      case CHIP_VEGA12:
 > 		      case CHIP_VEGA20:
 > 		         pc_lines = 2048;
 > 		         break;
 > 		      case CHIP_RAVEN:
 > 		      case CHIP_RAVEN2:
 > 		      case CHIP_RENOIR:
 > 		      case CHIP_NAVI10:
 > 		      case CHIP_NAVI12:
 > 		      case CHIP_SIENNA_CICHLID:
 > 		      case CHIP_NAVY_FLOUNDER:
 > 		      case CHIP_DIMGREY_CAVEFISH:
 > 		         pc_lines = 1024;
 > 		         break;
 > 		      case CHIP_NAVI14:
 > 		      case CHIP_BEIGE_GOBY:
 > 		         pc_lines = 512;
 > 		         break;
 > 		      case CHIP_VANGOGH:
 > 		      case CHIP_YELLOW_CARP:
 > 		         pc_lines = 256;
 > 		         break;
 > 		      default:
 > 		         assert(0);
 > 		      }
 >
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 29f264f25804eeea962f21c29c39050c3fc1663d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/29f264f25804eeea962f21c29c39050c3fc1663d/src/amd/common/ac_gpu_info.c#L1013-1042) {{< /quote >}}

*RadeonSI (OpenGL)* ドライバーにも *NGGカリング* は実装されており、デフォルト有効化の条件は *RADV* と概ね同じだが、ワークステーション用途を想定してか、マーケティングネームに *Pro* が入っているかが条件に追加されている (`is_pro_graphics`)。  

 > 		   sscreen->use_ngg = !(sscreen->debug_flags & DBG(NO_NGG)) &&
 > 		                      sscreen->info.chip_class >= GFX10 &&
 > 		                      (sscreen->info.family != CHIP_NAVI14 ||
 > 		                       sscreen->info.is_pro_graphics);
 > 		   sscreen->use_ngg_culling = sscreen->use_ngg &&
 > 		                              sscreen->info.max_render_backends >= 2 &&
 > 		                              !((sscreen->debug_flags & DBG(NO_NGG_CULLING)) ||
 > 		                                LLVM_VERSION_MAJOR <= 11 /* hangs on 11, see #4874 */);
 >
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/blob/f00d3e29094942cf8a35c76646b2cfd82f4b3f8a/src/gallium/drivers/radeonsi/si_pipe.c#L1242-1250> {{< /quote >}}

*RadeonSI* では、*Navi14* を除く *RDNA/GFX10* とそれ以降の世代かつ、*Pro* カードで *NGG* がデフォルトで有効化され、*NGGカリング* はそれに加え RB が 2基以上の場合にデフォルトで有効化される。  

