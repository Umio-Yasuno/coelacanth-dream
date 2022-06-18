---
title: "GFX11 では FP64 演算性能が FP32 の 1/32 に"
date: 2022-06-18T21:33:21+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Jay Foad 氏より LLVM に投稿されたパッチから、*GFX11* では FP32:FP64 の演算性能レートが変更され、従来の 16:1 から 32:1 になることがわかった。  
パッチはそれを確認するテストを追加するためのものとなっている。  

 > 		This mostly just tests that DPFP is 1/32 rate on GFX11, instead of 1/16
 > 		rate as on GFX10.
 >
 > {{< quote >}} [[AMDGPU] Add a GFX11 MCA test · llvm/llvm-project@d393538](https://github.com/llvm/llvm-project/commit/d393538c7f85a564f6f1c702aa2895b41f025da3) {{< /quote >}}

 * [[AMDGPU] Add a GFX11 MCA test · llvm/llvm-project@d393538](https://github.com/llvm/llvm-project/commit/d393538c7f85a564f6f1c702aa2895b41f025da3)
 * <https://reviews.llvm.org/rGd393538c7f85a564f6f1c702aa2895b41f025da3>

*RDNA 1/GFX10 アーキテクチャ*、*RDNA 2/GFX10.3 アーキテクチャ* では FP32:FP64 の演算性能レートは 16:1 となっていた。*Vega20/gfx906* のような一部のサーバー向け GPU や *Carrizo/gfx801* APU の FP32:FP64 = 2:1 となっているアーキテクチャを除けば、*RDNA 1/2* の演算性能レートは *GCN 系アーキテクチャ* と基本同じである。  

{{< figure src="/image/2020/11/17/amd-cdna-rdna-simd.webp" title="CDNA/RDNA SIMD" caption="画像出典: [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) (Page 6), <br>&emsp; [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) (Page 9)" >}}

*RDNA 1/2* はゲーミング向けのアーキテクチャという言葉から、コンピュート性能は低いと見られることがあるが、演算性能レートについては変わらなかった。  
それが *RDNA 3* と目される *GFX11* では変更される。変更には、より FP32 演算性能を重視し、WGP (CU) の実装面積を減らす目的があるのではないかと考えられる。  
一方 *CDNA 系アーキテクチャ* では *CDNA 2/Aldebaran/gfx90a* からは `FullRateFP64` に対応し、FP32:FP64 = 1:1 となっている。  
*CDNA* と *RDNA* の大きな違いには、HWレイトレーシングの有無、行列演算命令 (MFMA, MAI) の有無、FP64 演算性能があるが、次世代では FP64演算性能の違いがさらに広がることとなる。  

他社のグラフィクス、ゲーミング向け GPU では、NVIDIA TU102 では FP32:FP64 = 32:1[^tu102]、NVIDIA GA102 では FP32:FP64 = 64:1[^ga102] となっている。  
Intel GPU では、*Gen9 アーキテクチャ* では FP32:FP64 = 4:1、*Gen11 アーキテクチャ* からはネイティブでの対応が外され、ソフトウェアエミュレートでの対応となっている。{{< xe class="hpg" >}} もエミュレートでの対応となっているが、{{< xe class="hp/hpc" >}} ではネイティブで対応している。[^intel-gpu]  

[^intel-gpu]: [Xe-LP/HP より大きな命令キャッシュを持つ Xe-HPG | Coelacanth's Dream](/posts/2021/09/16/intel-xe_hpg-icache/)

[^tu102]: [nvidia-ampere-ga-102-gpu-architecture-whitepaper-v2.pdf](https://www.nvidia.com/content/PDF/nvidia-ampere-ga-102-gpu-architecture-whitepaper-v2.pdf)
[^ga102]: [NVIDIA Turing Architecture In-Depth | NVIDIA Technical Blog](https://developer.nvidia.com/blog/nvidia-turing-architecture-in-depth/)
