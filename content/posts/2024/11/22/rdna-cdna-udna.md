---
title: "AMD RDNA と CDNA の違いを考える"
date: 2024-11-22T14:07:10+09:00
draft: false
categories: [ "Hardware", "AMD", "Note" ]
tags: [ "RDNA", "CDNA", "UDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

2024/09/09 付けで公開された Tom's Hardware の Paul Alcorn 氏による、AMD の Jack Huynh 氏へのインタビュー記事にて、*RDNA アーキテクチャ* と *CDNA アーキテクチャ* を統合させた *UDNA アーキテクチャ* の構想が初めて語られた。  

 * [AMD announces unified UDNA GPU architecture — bringing RDNA and CDNA together to take on Nvidia's CUDA ecosystem | Tom's Hardware](https://www.tomshardware.com/pc-components/cpus/amd-announces-unified-udna-gpu-architecture-bringing-rdna-and-cdna-together-to-take-on-nvidias-cuda-ecosystem)

現在、AMD GPU のアーキテクチャは大きく分けてグラフィクス、ゲーム向けの *RDNA 系アーキテクチャ* と、サーバー、HPC 向けの *CDNA 系アーキテクチャ* の 2つが存在する。  
*UDNA アーキテクチャ* でどのようにその 2つが統合されるかは全く語られてはいないが、統合する上で意識する必要のある差異について一度個人的に整理してみることにした。  
記事内の予想に関しては完全に素人によるものであるため、信用しないでほしい。  

## WMMA, MFMA {#matrix}
*RDNA* も *CDNA* も行列積和演算命令をサポートするが、*RDNA* は WMMA (Wave Matrix Multiply Accumulate)、*CDNA* は MFMA (Matrix fused-multiply-add) と異なる命令になっている。  
2つのとしては、*CDNA* は MFMA 命令を処理するための Matrix Core Unit (miSIMD, Machine Intelligence SIMD) と専用のベクタレジスタ AccVGPR (Accumulation Vector general-purpose register) を持つが、
*RDNA* はそれらを持たない構成になっていることが挙げられる。  
*RDNA (RDNA 3)* の WMMA 命令は通常のベクタレジスタを使用し、内部的にドット積命令を複数呼ぶ仕様となっている。  

 * [How to accelerate AI applications on RDNA 3 using WMMA - AMD GPUOpen](https://gpuopen.com/learn/wmma_on_rdna3/)
 * [AMD matrix cores (amd-lab-notes) - AMD GPUOpen](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-matrix-cores-readme/)

なお、AccVGPR は *CDNA 1/MI100/gfx908/Arcturus* では完全に MFMA 命令専用のベクタレジスタだったが、
*CDNA 2/MI200/gfx90a/Aldebaran* ではベクタレジスタ全体を VGPR と AccVGPR の 2つのプールに分割する方式となり、また MFMA 命令の入出力も一部 VGPR を指定するようになった。  
*CDNA 3/MI300/gfx94x* でも VGPR と AccVGPR に分割する方式は変わっていないが、ドキュメントを読む限りでは MFMA 命令の入出力に関してはどちらでもよく、ベクタレジスタの制限は無くなったように見える。  

 * ["AMD Instinct MI200" Instruction Set Architecture: Reference Guide - instinct-mi200-cdna2-instruction-set-architecture.pdf](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/instruction-set-architectures/instinct-mi200-cdna2-instruction-set-architecture.pdf)
 * ["AMD Instinct MI300" Instruction Set Architecture: Reference Guide - amd-instinct-mi300-cdna3-instruction-set-architecture.pdf](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/instruction-set-architectures/amd-instinct-mi300-cdna3-instruction-set-architecture.pdf)

AMD の開発者が行列演算に関してサーバー向けを第一に考えているのであれば、*UDNA アーキテクチャ* では MFMA 命令を採用する可能性が高いように思う。  
しかし *RDNA* から見れば追加の Matrix Core Unit に追加のベクタレジスタと、必要なハードウェアリソースが増えることとなる。  
そのため、物理ベクタレジスタファイルサイズや Matrix Core Unit のスループットがゲーム向けとサーバー向けで異なることになるかもしれない。  
既に *RDNA 3* では従来通りの物理ベクタレジスタを持つものと、それよりも 1.5倍多いもので分かれている。  
VGPR と AccVGPR の 2つのプールに分割する方式については、MFMA 命令の制限が *CDNA 3* で無くなった (ように見える) ことから引き継ぐ必要はないかもしれないが、レジスタ割り当ての最適化に影響する可能性もある。  

## キャッシュ {#cache}
AMD の Jack Huynh 氏はインタビューの中で、*RDNA* 側で犯したミスとして、メモリ階層、サブシステムの変更を挙げており、理由は最適化のマトリクスをリセットする必要があったこととしている。  
ただ、これについて具体的に何を指すのか、自分にはわかっていない。  

 > So, one of the things we want to do is ...we made some mistakes with the RDNA side; each time we change the memory hierarchy, the subsystem, it has to reset the matrix on the optimizations. I don't want to do that.
 > {{< quote >}} [AMD announces unified UDNA GPU architecture — bringing RDNA and CDNA together to take on Nvidia's CUDA ecosystem | Tom's Hardware](https://www.tomshardware.com/pc-components/cpus/amd-announces-unified-udna-gpu-architecture-bringing-rdna-and-cdna-together-to-take-on-nvidias-cuda-ecosystem) {{< /quote >}}

*RDNA* で Shader Array ごとに持つ GL1 キャッシュを導入し、*RDNA 2* ではさらに Infinity Cache を導入したことなのか、キャッシュ帯域の変更のことなのか、メモリスコープを指定する方式の変更のことなのか、設計レベルの話なのか。  
メモリスコープの指定方式に関しては、*RDNA 1/2* では GLC (Globally Coherent), DLC (Device Level Coherent), SLC (System Level Coherent) の各ビットの組み合わせでスコープが異なり複雑だったが、*RDNA 3* でかなり簡略化された。  
*RDNA 4/GFX12* では CU, SE (Shader Engine), Device/Agent, System を指定する、よりシンプルな方式となっている。  

 * <https://github.com/llvm/llvm-project/blob/main/llvm/docs/AMDGPUUsage.rst#memory-model-gfx12>

 >            } else if (gfx_level >= GFX11) {
 >               /* GFX11 simplified it and exposes what is actually useful.
 >                *
 >                * GLC means device scope for loads only. (stores and atomics are always device scope)
 >                * SLC means non-temporal for GL1 and GL2 caches. (GL1 = hit-evict, GL2 = stream, unavailable in SMEM)
 >                * DLC means non-temporal for MALL. (noalloc, i.e. coherent bypass)
 >                *
 >                * GL0 doesn't have a non-temporal flag, so you always get LRU caching in CU scope.
 >                */
 >               if (access & ACCESS_TYPE_LOAD && scope_is_device)
 >                  result.value |= ac_glc;
 >         
 >               if (access & ACCESS_NON_TEMPORAL && !(access & ACCESS_TYPE_SMEM))
 >                  result.value |= ac_slc;
 >            } else if (gfx_level >= GFX10) {
 >               /* GFX10-10.3:
 >                *
 >                * VMEM and SMEM loads (SMEM only supports the first four):
 >                * !GLC && !DLC && !SLC means CU scope          <== use for normal loads with CU scope
 >                *  GLC && !DLC && !SLC means SA scope
 >                * !GLC &&  DLC && !SLC means CU scope, GL1 bypass
 >                *  GLC &&  DLC && !SLC means device scope      <== use for normal loads with device scope
 >                * !GLC && !DLC &&  SLC means CU scope, non-temporal (GL0 = GL1 = hit-evict, GL2 = stream)  <== use for non-temporal loads with CU scope
 >                *  GLC && !DLC &&  SLC means SA scope, non-temporal (GL1 = hit-evict, GL2 = stream)
 >                * !GLC &&  DLC &&  SLC means CU scope, GL0 non-temporal, GL1-GL2 coherent bypass (GL0 = hit-evict, GL1 = bypass, GL2 = noalloc)
 >                *  GLC &&  DLC &&  SLC means device scope, GL2 coherent bypass (noalloc)  <== use for non-temporal loads with device scope
 >                *
 >                * VMEM stores/atomics (stores are CU scope only if they overwrite the whole cache line,
 >                * atomics are always device scope, GL1 is always bypassed):
 >                * !GLC && !DLC && !SLC means CU scope          <== use for normal stores with CU scope
 >                *  GLC && !DLC && !SLC means device scope      <== use for normal stores with device scope
 >                * !GLC &&  DLC && !SLC means CU scope, GL2 non-coherent bypass
 >                *  GLC &&  DLC && !SLC means device scope, GL2 non-coherent bypass
 >                * !GLC && !DLC &&  SLC means CU scope, GL2 non-temporal (stream)  <== use for non-temporal stores with CU scope
 >                *  GLC && !DLC &&  SLC means device scope, GL2 non-temporal (stream)  <== use for non-temporal stores with device scope
 >                * !GLC &&  DLC &&  SLC means CU scope, GL2 coherent bypass (noalloc)
 >                *  GLC &&  DLC &&  SLC means device scope, GL2 coherent bypass (noalloc)
 >                *
 >                * "stream" allows write combining in GL2. "coherent bypass" doesn't.
 >                * "non-coherent bypass" doesn't guarantee ordering with any coherent stores.
 >                */
 > 
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/amd/common/ac_shader_util.c> {{< /quote >}}


 * [aco,llvm: rewrite how GLC, DLC, SLC are set, don't treat ACCESS_NON_READABLE as ACCESS_COHERENT (!22770) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/22770)

## Wave32/64 {#wave}
Wave (SIMD ユニットが実行するスレッドの塊) 内のスレッド数の違いについては、*RDNA* の登場から何度も語られてきたことであろうことから、過去の資料や記事に任せてここでは詳細を省く。

 * [AMD PowerPoint- White Template - RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf)
 * [RadeonSI ドライバーでは RDNA GPU も Wave64モードで各シェーダーを実行するように | Coelacanth's Dream](/posts/2020/07/02/radeonsi-shader-wave64-with-rdna/)
 * [RadeonSIドライバーで一部シェーダーを再び Wave32モードで実行するように | Coelacanth's Dream](/posts/2021/11/27/radeonsi-wave32-rework/)

*RDNA 1/2* では Wave32/64 のモードの違いによる性能特性は変わらないが、*RDNA 3* からは変わった。  
*RDNA 3* では CU 内の SIMD32 ユニットが増えたため、*RDNA 1/2* より多くのケースで Wave32 より Wave64 の方が優れた性能を示すとされている。  
また、*RDNA 3* では *RDNA 1/2* に実装されていた Sub-Vector Mode は削除された。  

 >            /* Gfx10: Pixel shaders without interp instructions don't suffer from reduced interpolation
 >             * performance in Wave32, so use Wave32. This helps Piano and Voloplosion.
 >             *
 >             * Gfx11: Prefer Wave64 to take advantage of doubled VALU performance.
 >             */
 >            if (sscreen->info.gfx_level < GFX11 && stage == MESA_SHADER_FRAGMENT && !info->num_inputs)
 >               return 32;
 >         
 >            /* Gfx10: There are a few very rare cases where VS is better with Wave32, and there are no
 >             * known cases where Wave64 is better.
 >             *
 >             * Wave32 is disabled for GFX10 when culling is active as a workaround for #6457. I don't
 >             * know why this helps.
 >             *
 >             * Gfx11: Prefer Wave64 because it's slightly better than Wave32.
 >             */
 >            if (stage <= MESA_SHADER_GEOMETRY &&
 >                (sscreen->info.gfx_level == GFX10 || sscreen->info.gfx_level == GFX10_3) &&
 >                !(sscreen->info.gfx_level == GFX10 && si_shader_culling_enabled(shader)))
 >               return 32;
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/gallium/drivers/radeonsi/si_state_shaders.cpp> {{< /quote >}}

 * [西川善司の3DGE：Radeon RX 7900 XTX/XTは何が変わったのか。大幅な性能向上を遂げたNavi 31世代の秘密を探る](https://www.4gamer.net/games/660/G066019/20221212087/)

統合にあたって重要なのは Wave32/64 モードより、CU 内の SIMD ユニット幅と命令の発行方式かもしれない。  
*RDNA 1/2* は SIMD32 で Wave32 を 1回で、Wave64 を 2回に分けて発行する方式であり、*RDNA 3* は 2xSIMD32 で Wave32 (VOPD, Dual issue) を 1回、Wave64 を 1回で発行する方式する。 (VOPD 命令以外を Wave32 で発行した場合は不明)  
そして *GCN, CDNA* は SIMD16 で Wave64 を 4回に分けて発行する方式となる。  
*RDNA* は低レイテンシを、*GCN, CDNA* は SIMD ユニットの高使用率を意識した実行方式となっている。  

実行方式は *UDNA* でどのように統合するのか、特に気になる点である。  
ゲーム向けとサーバー向けで、SIMD ユニット幅と CU 内の数を変えてしまっては統合する意味が薄れてしまうようにも思う。
