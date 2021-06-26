---
title: "LLVM に新たな RDNA 2 APU の GPUID、gfx1035 が追加される"
date: 2021-06-24T11:03:47+09:00
draft: false
tags: [ "LLVM", "gfx1035", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

AMD GPU とは GPGPUコンパイラやシェーダーコンパイラのバックエンドとしての関わりを持つ LLVM に、新たな AMD GPUターゲット、 *GPUID: gfx1035* を追加するパッチが投稿、公開されている。  

 * [⚙ D104804 [AMDGPU] Add gfx1035 target](https://reviews.llvm.org/D104804)

パッチを投稿したのは [aakanksha555 ( Aakanksha Patil)](https://reviews.llvm.org/p/aakanksha555/) 氏。氏は過去に *gfx1034* を追加するパッチも投稿している。  
{{< link >}} [LLVM に GPUID: gfx1034 が追加される | Coelacanth's Dream](/posts/2021/05/14/llvm-gfx1034/) {{< /link >}}

## 2つ目の RDNA 2 APU

パッチの内容によれば、*gfx1035* は APU であり、*RDNA 2/GFX10.3* 世代の GPUID であることからもわかるように、他 *RDNA 2* GPU と同様の ISA、機能を備えている。  

 > 		     ``gfx1035``                 ``amdgcn``   APU   - cumode          - Absolute      - *pal-amdpal*  *TBA*
 > 		                                                    - wavefrontsize64   flat
 > 		                                                                        scratch                       .. TODO::
 > 		                                                                                                        Add product
 > 		                                                                                                        names.
 >
 > {{< quote >}} [⚙ D104804 [AMDGPU] Add gfx1035 target](https://reviews.llvm.org/D104804) {{< /quote >}}

*RDNA 2* 世代の APU は 2つ目であり、1つ目には *gfx1033* が当たる。 *gfx1033* も ISA、機能は同様。  

 > 		     ``gfx1033``                 ``amdgcn``   APU   - cumode          - Absolute      - *pal-amdpal*  *TBA*
 > 		                                                    - wavefrontsize64   flat
 > 		                                                                        scratch                       .. TODO::
 > 		
 > 		                                                                                                        Add product
 > 		                                                                                                        names.
 >
 > {{< quote >}} [⚙ D104804 [AMDGPU] Add gfx1035 target](https://reviews.llvm.org/D104804) {{< /quote >}}

Linux の Kernelドライバー、User Modeドライバー ([RadeonSI](/tags/radeonsi), [RADV](/tags/radv)) においても 2つの *RDNA 2* APU が確認されており、[VanGogh](/tags/vangogh)、[Yellow Carp](/tags/yellow_carp) がそうだ。  
タイミングで考えれば、 *gfx1033* が *VanGogh* 、*gfx1035* が *Yellow Carp* に割り当てられているのだろう。  

先日には、*Navi10/gfx1010* をベースにした APU でありながら、 *RDNA 2/GFX10.3* の目玉機能の一つであるレイトレーシング命令をサポートするという奇妙な GPUID、 [gfx1013](/tags/gfx1013) が現れたが、今回 *gfx1035* のサポートが追加されたことからも、やはり *Yellow Carp* に *gfx1013* は割り当てられず、  
他のオープンソースドライバー関連のソフトウェアにもまだサポートが追加されていない APU が *gfx1013* ではないかと思われる。  
{{< link >}} [GPUID: gfx1013 は RDNA/Navi10ベースの APU で、レイトレーシング命令をサポートしている? | Coelacanth's Dream](/posts/2021/06/06/gfx1013-apu-rt/) {{< /link >}}

| GPUID | Arch | Codename |
| :-- | :--: | :--: |
| *gfx90c* | Vega | Renoir/Lucienne/Cezanne (Green Sardine)[^green_sardine]
| *gfx1010* | RDNA | Navi10 |
| *gfx1011* | RDNA | Navi12 |
| *gfx1012* | RDNA | Navi14 |
| *gfx1013* | RDNA | ?????<br>(APU, Navi10-base, Support RayTracing Inst)[^gfx1013] |
| *gfx1030* | RDNA 2 | Sienna Cichlid/Navi21 |
| *gfx1031* | RDNA 2 | Navy Flounder/Navi22 |
| *gfx1032* | RDNA 2 | Dimgrey Cavefish/Navi23 ? |
| *gfx1033* | RDNA 2 | VanGogh? (APU) |
| *gfx1034* | RDNA 2 | Beige Goby? |
| *gfx1035* | RDNA 2 | Yellow Carp? (APU) |

[^green_sardine]: [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/)
[^gfx1013]: [gfx1013 | Coelacanth's Dream](/tags/gfx1013/)
