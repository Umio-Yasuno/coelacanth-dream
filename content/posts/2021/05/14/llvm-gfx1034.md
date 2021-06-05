---
title: "LLVM に GPUID: gfx1034 が追加される"
date: 2021-05-14T23:38:44+09:00
draft: false
tags: [ "Beige_Goby", "RDNA_2", "LLVM" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD GPU とはコンパイラバックエンドとしての関わりを持つ LLVM に *GPUID: gfx1034* のサポートを追加するパッチが投稿された。  

 * [[AMDGPU] Add gfx1034 target · llvm/llvm-project@464e4dc](https://github.com/llvm/llvm-project/commit/464e4dc50f4e6e058e12a7020385d5bf29fd1df6)

GPUID は gfx の後に続く数字が Major, Minor, Stepping の並びで構成されており、*gfx0134* の場合は `Major: 10, Minor: 3, Stepping: 4` となっている。  
*RDNA アーキテクチャ* の GPUID には `Major: 10, Minor: 1`  (gfx101x) が割り当てられ、*RDNA 2 アーキテクチャ* には基本 `Major: 10, Minor: 3` (gfx103x) が割り当てられている。  
タイミングを考えれば先日 Linux Kernel (amd-gfx) にパッチが投稿された *Beige Goby* の GPUID が *gfx1034* ではないかと思われるが、まだはっきりとそう示されてはいない。  
{{< link >}} [新たな RDNA 2 GPU、"Beige Goby" をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/05/13/amd-beige_goby/) {{< /link >}}

パッチの内容としては、既存のコード内に *gfx1034* のパターンを追加したものがほとんどであり、*gfx1034* に向けた特定の機能サポートなどは見受けられない。  
言い換えれば、ベースにした製品が既に発表されている *Sienna Cichlid/Navi21/gfx1030* 、*Navy Flounder/Navi22/gfx1031* と、命令/ISAレベルでは同機能ということでもある。  

[llvm-project/AMDGPUUsage.rst](https://github.com/llvm/llvm-project/blob/main/llvm/docs/AMDGPUUsage.rst) ドキュメントに追加された内容では、*gfx1034* は dGPU とされている。  
Linux Kernel (amd-gfx) への *Beige Goby* サポートに向けたパッチでも、APU であることを示すフラグといった、APU に関する記述は無かったため、dGPU であることは *Beige Goby* と *gfx1034* とで一致する。  
また、*gfx1033* は APU であることが同ドキュメントには記述されている。


 > 		     ``gfx1034``                 ``amdgcn``   dGPU  - cumode          - Absolute      - *pal-amdpal*  *TBA*
 > 		                                                    - wavefrontsize64   flat
 > 		                                                                        scratch                       .. TODO::
 > 		
 > 		                                                                                                        Add product
 > 		                                                                                                        names.
 >
 > {{< quote >}} [[AMDGPU] Add gfx1034 target · llvm/llvm-project@464e4dc](https://github.com/llvm/llvm-project/commit/464e4dc50f4e6e058e12a7020385d5bf29fd1df6#diff-e60878666958a89628598e321bbd8dca134904a69313d8afb07aa6771945fc3f) {{< /quote >}}



| RDNA 2 | GPUID |
| :-- | :--: |
| Sienna Cichlid/Navi21 | gfx1030 |
| Navy Flounder/Navi22 |  gfx1031 |
| Dimgrey Cavefish/Navi23 | gfx1032? |
| VanGogh | gfx1033? |
| Beige Goby | gfx1034? |
