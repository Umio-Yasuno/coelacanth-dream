---
title: "RadeonSI で gfx1036、gfx1037 のサポートが進められる"
date: 2022-02-27T15:11:49+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "GC_10_3_6", "GC_10_3_7", "RadeonSI", "RDNA_2" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD のソフトウェアエンジニア [Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏より、*RadeonSI (OpenGL)* ドライバーに *GC_10_3_6 (gfx1036)* 、*GC_10_3_7 (gfx1037)* のサポートを追加するパッチ、マージリクエストが投稿されている。  

* [amd: add support for gfx1036 and gfx1037 chips, update addrlib (!15155) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/15155)

## GC_10_3_6 (gfx1036), GC_10_3_7 (gfx1037) {#gc}
APU/GPU としてのファミリーは AMDGPUドライバー同様、`GC_10_3_6` `GC_10_3_7` とされ、チップとしてのファミリーはそれぞれ *gfx1036* , *gfx1037* として識別される。  
{{< link >}} [新たな RDNA 2 APU の IPブロック ―― GC 10.3.7, DCN 3.1.6, VCN 3.1.1 | Coelacanth's Dream](/posts/2022/02/17/amdgpu-new-rdna_2-ip/) {{< /link >}}
{{< link >}} [さらなる RDNA 2 APU の IPブロック ―― GC 10.3.6, DCN 3.1.5, VCN 3.1.2 | Coelacanth's Dream](/posts/2022/02/17/rdna_2-ip-gc_10_3_6/) {{< /link >}}

 > 		 #define FAMILY_NV      0x8F
 > 		 #define FAMILY_VGH     0x90
 > 		 #define FAMILY_YC      0x92
 > 		+#define FAMILY_GC_10_3_6  0x95
 > 		+#define FAMILY_GC_10_3_7  0x97
 > 		
 > 		 // AMDGPU_FAMILY_IS(familyId, familyName)
 > 		 #define FAMILY_IS(f, fn)     (f == FAMILY_##fn)
 > 		@@ -111,6 +113,10 @@
 > 		
 > 		 #define AMDGPU_YELLOW_CARP_RANGE 0x01, 0xFF
 > 		
 > 		+#define AMDGPU_GFX1036_RANGE    0x01, 0xFF
 > 		+
 > 		+#define AMDGPU_GFX1037_RANGE    0x01, 0xFF
 >
 > {{< quote >}} [amd: add support for gfx1036 and gfx1037 chips (85caa4be) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/85caa4be69897570d0b03600798b7f6ad5720f6e?merge_request_iid=15155) {{< /quote >}}

現時点では、`FAMILY_GC_10_3_6` と `FAMILY_GC_10_3_7` ともに `CHIP_GFX1036` として認識される。  

 > 		+   case FAMILY_GC_10_3_6:
 > 		+      identify_chip(GFX1036);
 > 		+      break;
 > 		+   case FAMILY_GC_10_3_7:
 > 		+      identify_chip2(GFX1037, GFX1036);
 > 		+      break;
 >
 > {{< quote >}} [amd: add support for gfx1036 and gfx1037 chips (85caa4be) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/85caa4be69897570d0b03600798b7f6ad5720f6e?merge_request_iid=15155) {{< /quote >}}

`ac_get_llvm_processor_name()` において、GPUID が他 *RDNA 2 APU/dGPU* とまとめて *gfx1030* とされているが、これは ISAレベルではそれらに差が無いからではないかと思われる。  

 > 		@@ -179,6 +179,7 @@ const char *ac_get_llvm_processor_name(enum radeon_family family)
 > 		    case CHIP_BEIGE_GOBY:
 > 		    case CHIP_VANGOGH:
 > 		    case CHIP_YELLOW_CARP:
 > 		+   case CHIP_GFX1036:
 > 		       return "gfx1030";
 > 		    default:
 >
 > {{< quote >}} [amd: add support for gfx1036 and gfx1037 chips (85caa4be) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/85caa4be69897570d0b03600798b7f6ad5720f6e?merge_request_iid=15155) {{< /quote >}}

今回、*gfx1036* と *gfx1037* 、ひいては `CHIP_GFX1036` サポート追加にあたって明かされた情報は少なく、他 *RDNA 2 APU* と同様に、RB+ (Render Backend Plus) をサポートし、Pixel Shader のバッファとして用いられる PC Lines (Paramater Cache Line) は 256本とされている。  
{{< link >}} [RADVドライバーに NGGカリングが実装 | Coelacanth's Dream](/posts/2021/07/26/radv-nggc/#pc) {{< /link >}}

また、今回で AMDGPUドライバーだけでなく、Mesa3D (*RadeonSI, RADV*) においても従来のようなコードネームがやはり廃止されることが確定した。  
*RDNA 2 dGPU* で用いられたような 色+魚 のコードネームも、ランダムな組み合わせで決定したという話があり、それであれば DeviceID を検出に用いない *RadeonSI, RADV* においても、IPバージョンや GPUID を元にした名前でサポートを進めた方が効率的なのだろう。  
