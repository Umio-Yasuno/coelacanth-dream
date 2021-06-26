---
title: "GPUID: gfx1013 は RDNA/Navi10ベースの APU で、レイトレーシング命令をサポートしている?"
date: 2021-06-06T06:10:39+09:00
draft: false
tags: [ "LLVM", "GFX10", "RDNA", "gfx1013" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU", "APU" ]
noindex: false
# summary: ""
---

先日正体不明の GPUID: *gfx1013* を LLVM に追加するパッチを取り上げたが、そのパッチに進展があった。 だがそれも *gfx1013* の正体を明かしてくれるどころか、謎を深める奇妙なものだった。  
{{< link >}} [RDN/GFX10.1世代に新たな GPUID、gfx1013? | Coelacanth's Dream](/posts/2021/06/04/llvm-gpuid-gfx1013/) {{< /link >}}

{{< pindex >}}
 * [gfx1013 は APU に割り当てられている?](#apu)
 * [レイトレーシング命令をサポート?](#rt)
{{< /pindex >}}

## gfx1013 は APU に割り当てられている? {#apu}
パッチを投稿した [bcahoon (Brendon Cahoon)](https://reviews.llvm.org/p/bcahoon/) 氏がレビューを受けて追加したコメント、コードに反映した内容によると、*gfx1013* は APU に割り当てられた GPUID だという。[^gfx1013-apu]  

[^gfx1013-apu]: [⚙ D103663 [AMDGPU] Add gfx1013 target](https://reviews.llvm.org/D103663#2800972)

 > 		     ``gfx1013``                 ``amdgcn``   APU   - cumode          - Absolute      - *rocm-amdhsa* *TBA*
 > 		                                                    - wavefrontsize64   flat          - *pal-amdhsa*
 > 		                                                    - xnack             scratch       - *pal-amdpal*  .. TODO::
 > 		
 > 		                                                                                                        Add product
 > 		                                                                                                        names.
 >
 > {{< quote >}} <https://reviews.llvm.org/D103663#inline-982973> {{< /quote >}}

*RDNA/GFX10.1/Navi1x* 世代はこれまで dGPU しか存在せず、オープンソースドライバーにおいてはそれらのみをサポートしていたが、今回で、コンパイラバックエンドである LLVM部とはいえ *RDNA/GFX10.1* 世代の APU サポートが開始されたことになる。  

## レイトレーシング命令をサポート? {#rt}
同様にコメントとパッチの内容から読み取れるものだが、*gfx1013* は *RDNA 2/GFX10.3* 世代でサポートしているレイトレーシング処理を行うための命令 `image_bvh_intersect_ray/image_bvh64_intersect_ray` 、サンプラーを用いずにコンポーネントから最大 4個のサンプルをロードする `image_msaa_load` をサポートしているようだ。  
繰り返しになるがそれら命令は *RDNA 2/GFX10.3* 世代でサポートされた命令であり、GPUIDのフォーマット上では *RDNA/GFX10.1* 世代となる *gfx1013* がサポートしているのはやはり奇妙な話だ。  
また、前回はどう活用されるのか不明とした `GFX_10A` 命令フォーマット (`GFX10_AEncoding`) だが、それら 2種の命令サポートの有無を判定するのに用いられている。  
*RDNA 2/GFX10.3* 世代のサポート時には `GFX10_BEncoding` が追加されており、それが命令サポートの判定に用いられていた。また、*RDNA 2/GFX10.3* 世代では `V_MAC_LEGACY_F32/V_MAD_LEGACY_F32` 命令が削除され、`V_FMAC_LEGACY_F32/V_FMA_LEGACY_F32` に置き換わられており、その置き換え処理にも用いられている。[^legacy-inst]  

[^legacy-inst]: [[AMDGPU] Add MC layer support for v_fmac_legacy_f32 · llvm/llvm-project@edc37ba](https://github.com/llvm/llvm-project/commit/edc37baca6d6e4f28b7f4136e3263d3f1c3199f1#diff-e8cf20be79c2bb674c6c52704423cac97ac06963f6e2a6c9e77d027375080c1d)

レイトレーシングをサポートする AMD APU が存在しない訳ではなく、**Xbox Series S|X** 、**PS5** といったゲーム機向け APU/SoC が該当するものとして思い浮かぶが、そういったものは公開されていない情報が多く、現時点では他オープンソースドライバー等に AMD のソフトウェアエンジニア方によるサポートは追加されていないため、自分が持ち合わせている情報もほとんど無く、判断するには乏しい。  
仮にそうだとしても今になって LLVM にサポートが追加されるというタイミングに対する疑問が生まれる。  
*RDNA 2* APU である [VanGogh](/tags/vangogh)、[Yellow Carp](/tags/yellow_carp) もレイトレーシング命令自体はサポートしているものと思われるが、それらは同時に *RDNA 2/GFX10.3* 世代においてドット積命令等、他の命令/機能もサポートしているため、 *gfx1013* からは外れると考えられる。  

*gfx1013* の詳細が一部明らかにされはしたが、やはり対応命令、機能的には *Navi10/gfx1010* をベースに、レイトレーシング命令を追加でサポートした APU の正体は何なのか、という謎は残る。  

{{< ref >}}
 * ["RDNA 2" Instruction Set Architecture: Reference Guide - RDNA2_Shader_ISA_November2020.pdf](https://developer.amd.com/wp-content/resources/RDNA2_Shader_ISA_November2020.pdf)
{{< /ref >}}
