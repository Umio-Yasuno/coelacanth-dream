---
title: "RadeonSI/RADV に Beige Goby GPU と Yellow Carp APU をサポートする最初のパッチが投稿される　―― Yellow Carp は VanGogh に続く RDNA 2 APU に"
date: 2021-05-19T23:13:13+09:00
draft: false
tags: [ "Beige_Goby", "Yellow_Carp", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU", "APU" ]
noindex: false
# summary: ""
---

AMD の GPUドライバー開発者である [Marek Olšák](https://github.com/marekolsak) 氏により、オープンソースドライバー [RadeonSI](/tags/radeonsi) (OpenGL)、[RADV](/tags/radv) (Vulkan) に向けた、*Beige Goby GPU* 、 *Yellow Carp APU* をサポートする最初のパッチが投稿された。  

 * [amd: add more fish (!10878) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10878/)

*Beige Goby* は先日 Linux Kernel (amd-gfx) へのパッチと同時に登場した新たな *RDNA 2* GPU のコードネーム。  
{{< link >}} [新たな RDNA 2 GPU、"Beige Goby" をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/05/13/amd-beige_goby/) {{< /link >}}
*Yellow Carp* は 2020/12 に投稿された SMUv13 (System Management Unit) に関するパッチの中で突然登場した APU のコードネームであり、*Yellow Carp* をサポートするための巨大なパッチ群はまだ投稿されていない。  
{{< link >}} [新たな AMD APU、"Yellow Carp"　―― SMU は v13 に | Coelacanth's Dream](/posts/2020/12/08/amd-yellow-carp-apu/) {{< /link >}}

今回は最初のパッチということもあり、各種ヘッダファイルに *Beige Goby* を判定するためのマクロや既存のコードにパターンを追加する初期的な対応であり、*Beige Goby* に限っては新しい話はない。  
しかし *Yellow Carp* となると話は変わり、AMD のオープンソースドライバーから約半年振りに名前が出されたかと思えば、*VanGogh APU* に続く *RDNA 2* APU であることがパッチの内容から明かされた。  
前述のパッチでは、*Yellow Carp* は APU であることが分かるくらいで、GPUコア部の世代については伏せられていた。  

 > 		   case CHIP_SIENNA_CICHLID:
 > 		   case CHIP_NAVY_FLOUNDER:
 > 		   case CHIP_DIMGREY_CAVEFISH:
 > 		   case CHIP_BEIGE_GOBY:
 > 		   case CHIP_VANGOGH:
 > 		   case CHIP_YELLOW_CARP:
 > 		      return "gfx1030";
 >
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10878/diffs#diff-content-fbbe09bc487101e3f2f96c0b8fe543c915fa99bf> {{< /quote >}}

*Yellow Carp APU* の GPUファミリーと主に判別に使われる `eRevisionId` が、 *VanGogh* 同様に最初から `0x01, 0xFF` と範囲が埋められているが、 *Raven APU (gfx902)* に対する *Raven2 APU (gfx909)* 、*Renioir APU (gfx90c)* のように、APU の GPU部が細かく派生する予定は今の所ないのだろうか。  

 > 		define AMDGPU_BEIGE_GOBY_RANGE         0x46, 0x50
 > 		
 > 		#define AMDGPU_VANGOGH_RANGE    0x01, 0xFF
 > 		
 > 		#define AMDGPU_YELLOW_CARP_RANGE 0x01, 0xFF
 >
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10878/diffs#diff-content-c1f3b41aade2b5733c9e6855b5288f64c3048688> {{< /quote >}}

また、*Dimgrey Cavefish/Navi23* と *VanGogh* に存在する、DCC (Delta Color Compression) 機能に関するハードウェアのバグが *Yellow Carp* にも存在するとのことで、これは開発時期を知るヒントになるかもしれない。 *Beige Goby* がそこに含まれないのは少し不思議に思うが、少し遅れて開発されたのだろうか？  


 > 		   /* Whether chips are affected by the image load/sample/gather hw bug when
 > 		    * DCC is enabled (ie. WRITE_COMPRESS_ENABLE should be 0).
 > 		    */
 > 		   info->has_image_load_dcc_bug = info->family == CHIP_DIMGREY_CAVEFISH ||
 > 		                                  info->family == CHIP_VANGOGH ||
 > 		                                  info->family == CHIP_YELLOW_CARP;

*Yellow Carp APU* について AMD の開発者より明かされた情報はまだ少なく、*VanGogh APU* に続く *RDNA 2* APU となり、SMU は *VanGogh* の SMUv11.5 よりメジャーバージョンが進んだ SMUv13 を搭載する、くらいだ。組み合わされる CPU や各種IPもまだはっきりしない。  
だが、*VanGogh* も自分の記事を見返したら最初のパッチが投稿されたのは 2020/09 で、未だ正式発表はされていない。明かされていない情報は数多くあるだろう。  
そういうことで *Yellow Carp APU* についても、変わらず気長に待つことにする。  
それに UMD (User Mode Driver) にパッチが投稿開始されたことから、Linux Kernel (amd-gfx) へのパッチ群も公開される日が近いかもしれない。  


