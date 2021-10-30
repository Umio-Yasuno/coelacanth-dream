---
title: "Xe-LP/HP より大きな命令キャッシュを持つ Xe-HPG"
date: 2021-09-16T05:43:34+09:00
draft: false
tags: [ "DG2", "Xe-HPG", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

Intel oneAPI の深層学習ライブラリ、[oneAPI Deep Neural Network Library (oneDNN)](https://github.com/oneapi-src/oneDNN) に *{{< xe class="hp/hpg" >}}* をサポートするコミットが取り込まれた。  
そこには *{{< xe class="hp/hpg" >}}* のアーキテクチャの一部を示す記述が含まれている。  
*{{< xe class="hpg" >}}* については Intel Architecture Day 2021 で詳細が語られたが、まだ正式リリースがまだということもありそれは一部で、語られていない部分はまだまだある。*{{< xe class="hpg"  >}} アーキテクチャ* を採用する最初の GPU/SoC *DG2/Alchemist* は 2021Q1 でのリリースが予定されている。  
{{< link >}} [Intel Architecture Day 2021 個人的まとめ　―― 用語が整理された Xe GPU | Coelacanth's Dream](/posts/2021/08/26/intel-arch-day-2021-xe-gpu/) {{< /link >}}

 * [src: gpu: backport xe_hp+ gpu support · oneapi-src/oneDNN@e6d288e](https://github.com/oneapi-src/oneDNN/commit/e6d288ef1943f93a782a644e3aac8b2a500c9299#diff-83f47fe7c7dcf480d6939a5b6af7df4082a26ee534ac014ca9873ae12d852c96)

## {{< xe class="hpg" >}}-Core の L1命令キャッシュサイズ {#l1i-cache}

*{{< xe >}} アーキテクチャ* では複数の EU (Vector Engine, Matrix Engine) やロード/ストアユニットをまとめた {{< xe >}}-Core ごとに L1命令キャッシュを持つ。  
{{< xe >}}-Core は、以前は Sub-Slice(s) と呼ばれており、Intel GPU 向けのオープンソースソフトウェア、ドライバーではそのまま Sub-Slice(s) を使っている。  
*Tiger/Rocket/Alder Lake* 、*DG1* が採用する *{{< xe class="lp" >}} アーキテクチャ* では L1命令キャッシュのサイズは 48 KiB となっていた。  
今回追加されたコードによれば、*{{< xe class="hp" >}}* は同じ 48 KiB、そして *{{< xe class="hpg" >}}* はその倍のサイズとなる 96 KiB を L1命令キャッシュに持つとしている。  

L1命令キャッシュサイズは Sub-Slice あたりの EU数等に変化はあったものの、*Gen9 アーキテクチャ* から 48 KiB で通してきたためアーキテクチャの変更でも注目される点と言える。  

 > 		static size_t icache_size(ngen::HW arch) {
 > 		    switch (arch) {
 > 		        case gpu_gen9: return 48 * 1024;
 > 		        case gpu_xe_lp: return 48 * 1024;
 > 		        case gpu_xe_hp: return 48 * 1024;
 > 		        case gpu_xe_hpg: return 96 * 1024;
 > 		        default: return 0;
 > 		    }
 > 		}
 >
 > {{< quote >}} [oneDNN/gen_convolution.cpp at c7e18067cc5353f5ccbd63129dd7eb64fa4275c4 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/c7e18067cc5353f5ccbd63129dd7eb64fa4275c4/src/gpu/jit/conv/gen_convolution.cpp) {{< /quote >}}

自分は GPUアーキテクトでも無いため確かなことは言えないが、*{{< xe class="hpg" >}}* で L1命令キャッシュを倍にした意図を個人的に考えていく。  

まず、上記コードの各アーキテクチャにおける L1命令キャッシュサイズは、実行するカーネルのサイズと比較し、カーネルが L1命令キャッシュより大きかった場合にメッセージを出力する部分に使われている。  
その点で考えると深層学習、推論処理においてより巨大なカーネルを実行する際の性能改善のため、L1命令キャッシュサイズを倍増させたとも考えられるが、ゲーミング向けの *{{< xe class="hpg" >}}* で増やし、コンピューティング向けである *{{< xe class="hp" >}}* は 48 KiB で据え置きというのは少し違和感を覚える。一応 *{{< xe class="Hp" >}}* の表記キャッシュサイズは今後変更される可能性もあるが。  
ただ開発時期や製造プロセスが影響している可能性もある。  

L1命令キャッシュの倍増がグラフィクス、ゲーミング性能にどう影響を与えるかを考えれば、Intel は Architecture Day 2021 でアップスケーリング技術 {{< xe >}} SS (Super Sampling) を発表しており、画質向上のため複雑な処理を行う上で有効だと思われる。  
それ以外にも、多数のシェーダーを処理するゲームにおいても効果的なのかもしれない。  

## {{< xe class="hp" >}} と {{< xe class="hpg" >}} の共通点 {#hp-hpg}

以前、*{{< xe class="hp" >}}* では EU (Vector Engine) あたりのスレッド数が 1つ増えて 8スレッドとなり、また 4スレッドと設定することでスレッドあたりのレジスタファイルを 256エントリに増やす Large GRFモード をサポートすることが [intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler) への新たなコミットで明かされた。  
{{< link >}} [Intel Xe-HP EU に追加されるパイプラインと増加するスレッド/レジスタファイル | Coelacanth's Dream](/posts/2021/06/08/intel-xe_hp-thread-reg-pipe/) {{< /link >}}
今回 *{{< xe class="hpg" >}}* も Large GRFモードをサポートすることが明かされた。  

 > 		    // Assume 7 threads by default
 > 		    int32_t threads_per_eu[2] = {7, 7};
 > 		    switch (gpu_arch_) {
 > 		        case gpu::compute::gpu_arch_t::gen9:
 > 		        case gpu::compute::gpu_arch_t::xe_lp:
 > 		            threads_per_eu[0] = 7;
 > 		            threads_per_eu[1] = 7;
 > 		            break;
 > 		        case gpu::compute::gpu_arch_t::xe_hp:
 > 		        case gpu::compute::gpu_arch_t::xe_hpg:
 > 		            threads_per_eu[0] = 8; // 128 regs/thread
 > 		            threads_per_eu[1] = 4; // 256 regs/thread
 > 		            break;
 >
 > {{< quote >}} [oneDNN/device_info.cpp at e6d288ef1943f93a782a644e3aac8b2a500c9299 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/e6d288ef1943f93a782a644e3aac8b2a500c9299/src/gpu/compute/device_info.cpp#L107) {{< /quote >}}

ここは *{{< xe class="hp/hpg" >}}* で共通する部分となる。  
だが *{{< xe class="hpg" >}}* は 64-bit 精度のデータ型にはエミュレートで対応、対し *{{< xe class="hp" >}}* は FP64/Int64 を処理する専用パイプラインを持つとしている。  
そのため EU部において一部共通するが、パイプライン構成についてはそれぞれのターゲットに向けて最適化された別物となる。  

| {{< xe >}} EU |  |  | Issue Port 0 | IP 1 | IP 2 | IP 3 | IP 4? |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| {{< xe class="lp" >}} | Branch<br>(Control Flow) | Send | Short<br>(INT /FP) | Math | - | - | - |
| {{< xe class="hp" >}} | ? | Send | INT? (int8/16/32)<br> /Branch? | Math? | DPAS? | Long?<br>(Int64 /FP64) | FP?<br>(FP16/32, BF16) |
|                       | In-Order | Out-of-Order | In-Order | Out-of-Order | Out-of-Order | In-Order | In-Order |
{{< link >}} [intel-graphics-compiler/RegDeps.cpp at f5e9e3dcda319339b82f1171c7e55add46884036 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/f5e9e3dcda319339b82f1171c7e55add46884036/visa/iga/IGALibrary/IR/RegDeps.cpp#L100) {{< /link >}}

| {{< xe >}} GPU | {{< xe class="lp" >}} | {{< xe class="hpg" >}} | {{< xe class="hp" >}} | {{< xe class="hpc" >}} |
| :-- | :--: | :--: | :--: | :--: |
| Vector Engine | 256-bit? | 256-bit | ? | 512-bit |
| VE per SS (Sub-Slice) | 16 | 16 | ? | 8 |
| L1I$ per SS | 48 KB | 96 KB | 48 KB | ? |
| Matrix Engine | N/A | 1024-bit | ? | 4096-bit |
| Load/Store (SLM) | 128B? | ? | ? | 512B |
| L1D$/SLM per SS | 128 KB | ? | ? | 512 KB |
| Native FP64 | N/A | N/A | Y | Y |

{{< ref >}}
 * [Arm Mali GPU Best Practices Developer Guide](https://developer.arm.com/documentation/101897/0200/shader-code/instruction-caches)
 * [PowerPoint Presentation - GDC2017-Advanced-Shader-Programming-On-GCN.pdf](https://gpuopen.com/wp-content/uploads/2017/03/GDC2017-Advanced-Shader-Programming-On-GCN.pdf)
 * [Reducing Shader Bottlenecks | Apple Developer Documentation](https://developer.apple.com/documentation/metal/optimizing_performance_with_the_gpu_counters_instrument/reducing_shader_bottlenecks)
 * [Tips and Tricks: Ray Tracing Best Practices | NVIDIA Developer Blog](https://developer.nvidia.com/blog/rtx-best-practices/)
 * [oneDNN/emulation.hpp at c7e18067cc5353f5ccbd63129dd7eb64fa4275c4 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/c7e18067cc5353f5ccbd63129dd7eb64fa4275c4/src/gpu/jit/gemm/emulation.hpp)
 * [oneDNN/gen_convolution.cpp at c7e18067cc5353f5ccbd63129dd7eb64fa4275c4 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/c7e18067cc5353f5ccbd63129dd7eb64fa4275c4/src/gpu/jit/conv/gen_convolution.cpp)
 * [oneDNN/xe_lp_x8s8x_convolution.hpp at c7e18067cc5353f5ccbd63129dd7eb64fa4275c4 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/c7e18067cc5353f5ccbd63129dd7eb64fa4275c4/src/gpu/ocl/xe_lp_x8s8x_convolution.hpp)
 * [oneDNN/neo_structs.hpp at c7e18067cc5353f5ccbd63129dd7eb64fa4275c4 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/c7e18067cc5353f5ccbd63129dd7eb64fa4275c4/src/gpu/jit/ngen/npack/neo_structs.hpp)
 * [oneDNN/device_info.cpp at e6d288ef1943f93a782a644e3aac8b2a500c9299 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/e6d288ef1943f93a782a644e3aac8b2a500c9299/src/gpu/compute/device_info.cpp)

{{< /ref >}}


