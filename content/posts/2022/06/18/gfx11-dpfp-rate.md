---
title: "GFX11 では FP64 演算性能が FP32 の 1/32 に"
date: 2022-06-18T21:33:21+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", "RDNA_3", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Jay Foad 氏より LLVM に投稿されたパッチから、*GFX11* では FP32:FP64 の演算性能レートが変更され、従来の 16:1 から 32:1 になることがわかった。  
パッチはそれを確認するテストを追加するためのものとなっている。  
影響を受ける命令には FP64 フォーマットを扱う、演算命令、変換命令、比較命令がある。  

 > 		This mostly just tests that DPFP is 1/32 rate on GFX11, instead of 1/16
 > 		rate as on GFX10.
 >
 > {{< quote >}} [[AMDGPU] Add a GFX11 MCA test · llvm/llvm-project@d393538](https://github.com/llvm/llvm-project/commit/d393538c7f85a564f6f1c702aa2895b41f025da3) {{< /quote >}}

 * [[AMDGPU] Add a GFX11 MCA test · llvm/llvm-project@d393538](https://github.com/llvm/llvm-project/commit/d393538c7f85a564f6f1c702aa2895b41f025da3)
 * <https://reviews.llvm.org/rGd393538c7f85a564f6f1c702aa2895b41f025da3>

*GFX10* では `uOps Per Cycle` が 0.14、`IPC` が 0.14 であるのに対し、*GFX11* では `uOps Per Cycle` が 0.08 に、`IPC` が 0.07 となっている。  
また命令のレイテンシも、*GFX10* では基本 28サイクルなのに対し、*GFX11* では 38サイクルとなっている。  
逆数、平方根の命令 `V_{RCP,RSQ,SQRT}_F64_E32` は 40サイクルで変わらない。  

 > 		# CHECK:      Iterations:        1		# CHECK:      Iterations:        1
 > 		# CHECK-NEXT: Instructions:      28		# CHECK-NEXT: Instructions:      28
 > 		# CHECK-NEXT: Total Cycles:      205	      |	# CHECK-NEXT: Total Cycles:      384
 > 		# CHECK-NEXT: Total uOps:        29		# CHECK-NEXT: Total uOps:        29
 > 		
 > 		# CHECK:      Dispatch Width:    1		# CHECK:      Dispatch Width:    1
 > 		# CHECK-NEXT: uOps Per Cycle:    0.14	      |	# CHECK-NEXT: uOps Per Cycle:    0.08
 > 		# CHECK-NEXT: IPC:               0.14	      |	# CHECK-NEXT: IPC:               0.07
 > 		# CHECK-NEXT: Block RThroughput: 29.0		# CHECK-NEXT: Block RThroughput: 29.0
 >
 > {{< quote >}} L: [llvm-project/gfx10-double.s at d393538c7f85a564f6f1c702aa2895b41f025da3 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/d393538c7f85a564f6f1c702aa2895b41f025da3/llvm/test/tools/llvm-mca/AMDGPU/gfx10-double.s) <br> / R: [llvm-project/gfx11-double.s at d393538c7f85a564f6f1c702aa2895b41f025da3 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/d393538c7f85a564f6f1c702aa2895b41f025da3/llvm/test/tools/llvm-mca/AMDGPU/gfx11-double.s) {{< /quote >}}

*RDNA 1/GFX10 アーキテクチャ*、*RDNA 2/GFX10.3 アーキテクチャ* では FP32:FP64 の演算性能レートは 16:1 (32x ALU Unit に対して 2x DP Unit)となっていた。*Vega20/gfx906* のような一部のサーバー向け GPU や *Carrizo/gfx801* APU の FP32:FP64 = 2:1 (`HalfRate64Ops`) となっているアーキテクチャを除けば、*RDNA 1/2* の演算性能レートは *GCN 系アーキテクチャ* と基本同じである。  

{{< figure src="/image/2020/11/17/amd-cdna-rdna-simd.webp" title="CDNA/RDNA SIMD" caption="画像出典: [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) (Page 6), <br>&emsp; [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) (Page 9)" >}}

*RDNA 1/2* はゲーミング向けのアーキテクチャという言葉から、コンピュート性能は低いという印象を持たれることがあるが、演算性能レートについては変わっていない。  
それが *RDNA 3* と目される *GFX11* では変更される。変更には、よりグラフィクス処理で求められる FP32 演算性能を重視し、WGP (CU) の実装面積を減らし、効率を向上させる目的があるのではないかと考えられる。  
一方 *CDNA 系アーキテクチャ* では *CDNA 2/Aldebaran/gfx90a* からは `FullRateFP64` に対応し、FP32:FP64 = 1:1 となっている。  
*CDNA* と *RDNA* の大きな違いには、HWレイトレーシングの有無、行列演算命令 (MFMA, MAI) の有無、FP64 演算性能があるが、次世代では FP64演算性能の差がさらに広がることとなる。  

他社のグラフィクス、ゲーミング向け GPU では、NVIDIA TU102 では FP32:FP64 = 32:1[^tu102]、NVIDIA GA102 では FP32:FP64 = 64:1[^ga102] となっている。  
Intel GPU では、*Gen9 アーキテクチャ* では FP32:FP64 = 4:1、*Gen11 アーキテクチャ* からはネイティブでの対応が外され、エミュレートでの対応となっている。{{< xe class="hpg" >}} もエミュレートでの対応となっているが、{{< xe class="hp/hpc" >}} ではネイティブで対応しており、EU に専用の実行ポートを備える。[^intel-gpu]  
そのためアーキテクチャ、SIMDユニットの構成による所もあるが、最新のグラフィクス、ゲーミング向け GPU として *GFX11* は以前 FP64演算性能が確保されている方だとも言える。  

[^intel-gpu]: [Xe-LP/HP より大きな命令キャッシュを持つ Xe-HPG | Coelacanth's Dream](/posts/2021/09/16/intel-xe_hpg-icache/)

[^tu102]: [nvidia-ampere-ga-102-gpu-architecture-whitepaper-v2.pdf](https://www.nvidia.com/content/PDF/nvidia-ampere-ga-102-gpu-architecture-whitepaper-v2.pdf)
[^ga102]: [NVIDIA Turing Architecture In-Depth | NVIDIA Technical Blog](https://developer.nvidia.com/blog/nvidia-turing-architecture-in-depth/)

{{< ref >}}
 * [Intel® Processor Graphics Xᵉ-LP API Developer and Optimization Guide](https://www.intel.com/content/www/us/en/developer/articles/guide/lp-api-developer-optimization-guide.html)
 * [Re: What processors have an internal Gen11 GPU *with* FP64 (double precision) enabled? - Intel Communities](https://community.intel.com/t5/Graphics/What-processors-have-an-internal-Gen11-GPU-with-FP64-double/m-p/691217)
 * <https://github.com/oneapi-src/oneDNN/blob/df399b0d8bffcb0f59aa8753344dd254f4421ac5/src/gpu/jit/gemm/emulation.hpp#L42-L54>
 * `FeatureFP64` - <https://github.com/intel/intel-graphics-compiler/blob/d82cdf69941156e976205c64724801be7080e29a/IGC/VectorCompiler/lib/GenXCodeGen/GenX.td>
{{< /ref >}}
