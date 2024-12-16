---
title: "GL1キャッシュがバッファへと変わる AMD RDNA 4 アーキテクチャ"
date: 2024-12-16T07:13:15+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX12", "RDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

読み取り専用キャッシュとして RDNA 1 アーキテクチャから導入され、RDNA 3 アーキテクチャからは 256KiB (RDNA 1/2 は 128KiB) に増量された GL1キャッシュ (Graphics L1キャッシュ) だが、RDNA 4 からはキャッシュではなくバッファへと変わる。  

## GL1キャッシュ {#gl1c}
GL1キャッシュは Shader Array 内に配置されており、Shader Array 内の WGP (CU)、RenderBackend (ROP) で共有される読み取り専用のキャッシュとなる。  
RenderBackend は Shader Array、GL1キャッシュと結び付けられており、そのため RDNA 系アーキテクチャではメモリバス幅は減らしても RenderBackend はフルスペック、という製品構成が可能となっていた。  

## GL1バッファ {#gl1-buffer}
それが RDNA 4 アーキテクチャではキャッシュからバッファへと変わる。  
RDNA 系アーキテクチャはキャッシュ周りの変更が多く、RDNA 1 では GL1キャッシュの導入、RDNA 2 では L3キャッシュ (Infinity Cache) の導入、RDNA 3 では L0 データキャッシュと GL1キャッシュの増量、そして RDNA 4 で GL1キャッシュのバッファ化となる。  
また L2キャッシュラインサイズも RDNA 4 では変更され、RDNA 1/2/3 の 128Byte から 256Byte となる。  

RDNA 4 アーキテクチャが GL1キャッシュを持たないことは 2024/05/02 に投稿された AMD の Marek Olšák 氏による RadeonSI ドライバーへのマージリクエストで、  
バッファへと変わることは 2024/07/12 に投稿された AMD の Pierre van Houtryve 氏による、LLVM 内のドキュメントに RDNA 4/GFX12 のメモリーモデルを追加するプルリクエストでオープンにされている。  

 * [[AMDGPU] Document & Finalize GFX12 Memory Model by Pierre-vh · Pull Request #98599 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/98599)
   * <https://github.com/llvm/llvm-project/blob/main/llvm/docs/AMDGPUUsage.rst#memory-model-gfx12>
 * [amd: add initial code for gfx12 (!29007) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/29007)

 >            if (info->gfx_level >= GFX10 && info->gfx_level < GFX12)
 >               fprintf(f, "    l1_cache_size = %i KB\n", DIV_ROUND_UP(info->l1_cache_size, 1024));
 >         
 >
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/main/src/amd/common/ac_gpu_info.c) {{< /quote >}}

LLVM のドキュメントでは、GL1バッファは Shader Array 内のクライアントから見て、L2キャッシュへのブリッジとして機能するとしている。  

バッファ化によって Shader Array の構造も従来から変わったと考えられる。  
RadeonSI ドライバーと RADV ドライバーで使われる共有部分にて、`tcc_rb_non_coherent` の判定に `info->gfx_level < GFX12` が追加されている。  
`tcc_rb_non_coherent` はL2キャッシュブロック (テクスチャキャッシュ、TCC) と RenderBackend の数が一致しない、上記で挙げたような RDNA アーキテクチャで可能になった構成であることを示す変数となる。  
RDNA 4/GFX12 とそれ以降でないことが条件に追加されたことは、RDNA 4 では L2キャッシュに RenderBackend を接続した以前の構造に戻った可能性を示している。  

 >            info->tcc_rb_non_coherent = info->gfx_level < GFX12 &&
 >                                        !util_is_power_of_two_or_zero(info->num_tcc_blocks) &&
 >                                        info->num_rb != info->num_tcc_blocks;
 >
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/main/src/amd/common/ac_gpu_info.c) {{< /quote >}}


読み取り専用キャッシュをバッファにすることのメリットに関して、自分に断定できることはない。  
強いて言うのであれば、キャッシュ制御に必要なハードウェア機能の削減、ソフトウェア的にキャッシュコヒーレンシを維持するための命令を発行する必要が無いとかだろうか？  

 * [コンピュータアーキテクチャの話(353) GPUにおける1次キャッシュのコヒーレンシ | TECH+（テックプラス）](https://news.mynavi.jp/techplus/article/architecture-353/)
