---
title: "Xe-HPC のサポートが intel-graphics-compiler、oneDNN に追加される"
date: 2021-10-29T18:02:53+09:00
draft: false
tags: [ "Xe-HPC", "Xe-HPG", "Gen12", "DG2" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

Intel GPU の OpenCLコンパイラ [intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler)、oneAPI における深層学習ライブラリ [oneDNN](https://github.com/oneapi-src/oneDNN) に {{< xe class="hpc" >}} のサポートが追加された。  

 * [support for DG2+PVC · intel/intel-graphics-compiler@574378f](https://github.com/intel/intel-graphics-compiler/commit/574378f2ba14ae4f872c78b7e104228d904a337d)
 * [gpu: jit: ngen: implement ngen Xe HPC support · oneapi-src/oneDNN@8c79e4e](https://github.com/oneapi-src/oneDNN/commit/8c79e4e352d716745d825cddd5a49539d73e7471)
 * [gpu: compute: add Xe HPC engine · oneapi-src/oneDNN@dac5847](https://github.com/oneapi-src/oneDNN/commit/dac5847295d587737133d4027a9c435383926fe3)
 * [gpu: ocl: implement ocl Xe HPC support · oneapi-src/oneDNN@43b0a06](https://github.com/oneapi-src/oneDNN/commit/43b0a063495944d31d3826482bc42bb2f31be7aa)

## {{< xe class="hpc" >}} の L1命令キャッシュサイズ {#icache}

*{{< xe >}} アーキテクチャ* では複数の EU (Vector Engine, Matrix Engine) やロード/ストアユニットをまとめた {{< xe >}}-Core ごとに L1命令キャッシュを持つ。  
{{< xe >}}-Core は、以前は Sub-Slice(s) と呼ばれており、Intel GPU 向けのオープンソースソフトウェア、ドライバーではそのまま Sub-Slice(s) を使っている。  

*Tiger/Rocket/Alder Lake* 、*DG1* が採用する *{{< xe class="lp" >}} アーキテクチャ* では L1命令キャッシュのサイズは 48 KiB となっていた。  
以前には、*{{< xe class="hp" >}}* は *{{< xe class="lp" >}}* と同じ 48 KiB、*{{< xe class="hpg" >}}* はその倍のサイズとなる 96 KiB を L1命令キャッシュに持つことを示す記述が oneDNN に追加されていた。  
{{< link >}} [Xe-LP/HP より大きな命令キャッシュを持つ Xe-HPG | Coelacanth's Dream](/posts/2021/09/16/intel-xe_hpg-icache/) {{< /link >}}
今回、同様の部分に *{{< xe class="hpc" >}}* の L1命令キャッシュサイズを示す記述が追加され、*{{< xe class="lp/hp" >}}* とも *{{< xe class="hpg" >}}* とも異なる 80 KiB とされている。  

 > 		static size_t icache_size(ngen::HW arch) {
 > 		    switch (arch) {
 > 		        case gpu_gen9: return 48 * 1024;
 > 		        case gpu_xe_lp: return 48 * 1024;
 > 		        case gpu_xe_hp: return 48 * 1024;
 > 		        case gpu_xe_hpg: return 96 * 1024;
 > 		        case gpu_xe_hpc: return 80 * 1024;
 > 		        default: return 0;
 > 		    }
 > 		}
 >
 > {{< quote >}} [oneDNN/gen_convolution.cpp at 8be125b5a3f6c55091bf322f75a98daad4be2b14 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/8be125b5a3f6c55091bf322f75a98daad4be2b14/src/gpu/jit/conv/gen_convolution.cpp#L37-L46) {{< /quote >}}

ただ、GPUアーキテクチャにおける命令キャッシュの増減について触れられることは少ないと感じており、単にサイズからどうこう言うことは自分にはできない。  
oneDNN では命令キャッシュの情報を、実行するカーネルの生サイズと比較してカーネルが命令キャッシュより大きかった場合にログに警告メッセージを出力する、といった使い方をしている。より大きなカーネルの実行性能を高めるために命令キャッシュを大きくした、という可能性はある。  

とはいえ、上記のコードの通りであれば、*{{< xe class="lp/hp" >}}*, *{{< xe class="hpg" >}}*, *{{< xe class="hpc" >}}* でそれぞれ {{< xe >}}-Core (Sub-Slice) あたりの命令キャッシュサイズは異なることとなり、目的があってサイズを調整していることは窺い知れる。  

| {{< xe >}} GPU | {{< xe class="lp" >}} | {{< xe class="hpg" >}} | {{< xe class="hp" >}} | {{< xe class="hpc" >}} |
| :-- | :--: | :--: | :--: | :--: |
| Vector Engine | 256-bit? | 256-bit | ? | 512-bit |
| VE per SS (Sub-Slice) | 16 | 16 | ? | 8 |
| L1I$ per SS | 48 KB | 96 KB? | 48 KB? | 80 KB? |
| Matrix Engine | N/A | 1024-bit | ? | 4096-bit |
| Load/Store per cycle (SLM) | 128 B?<br>(per Dataport) | ? | ? | 512 B |
| L1D$/SLM per SS | 128 KB | ? | ? | 512 KB |
| Native FP64 | N/A | N/A | Y | Y |

*{{< xe class="hpc" >}}* では GRF (General Register File) のサイズが 64 Bytes (512-bit) に拡張されており、従来のアーキテクチャでは 32 Bytes (256-bit, 8x 32-bit) だった。  
恐らく FP64演算性能の最適化のため、8x 64-bit の構成を採っていると思われる。  
同様のアーキテクチャ拡張は *AMD Aldebaran/MI200 GPU* でも施されており、FullRateFP64、PackedFP32 への対応と同時に行われている。  
{{< link >}} [LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か | Coelacanth's Dream](/posts/2021/02/19/llvm-gfx90a/) {{< /link >}}

 > 		    const int REG_SIZE_BITS = p >= Platform::XE_HPC ? 512 : 256;
 >
 > {{< quote >}} [intel-graphics-compiler/Messages.cpp at 2887a020540d935b2f0872b5bd56c0336bc19570 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2887a020540d935b2f0872b5bd56c0336bc19570/visa/iga/IGALibrary/IR/Messages.cpp#L32) {{< /quote >}}
 >
 > 		    const int GRF_BYTES = platform() >= Platform::XE_HPC ? 64 : 32;
 > 		    const int DEFAULT_SIMD = GRF_BYTES / 2;
 >
 > {{< quote >}} [intel-graphics-compiler/KernelParser.cpp at 5d1f5306aa6b3aac36f4f396ee2ccfbcdf24dbae · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/5d1f5306aa6b3aac36f4f396ee2ccfbcdf24dbae/visa/iga/IGALibrary/Frontend/KernelParser.cpp#L4362) {{< /quote >}}

EU あたりの最大スレッド数が 8スレッドとなり、また半分の 4スレッドに設定することでスレッドあたりのレジスタファイルを 256エントリに増やす Large GRFモードをサポートする点は *{{< xe class="hp/hpg/hpc" >}}* で共通する。  

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
 > 		        case gpu::compute::gpu_arch_t::xe_hpc:
 > 		            threads_per_eu[0] = 8; // 128 regs/thread
 > 		            threads_per_eu[1] = 4; // 256 regs/thread
 > 		            break;
 > 		        default: break;
 > 		    }
 >
 > {{< quote >}} [oneDNN/device_info.cpp at dac5847295d587737133d4027a9c435383926fe3 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/dac5847295d587737133d4027a9c435383926fe3/src/gpu/compute/device_info.cpp#L116-L131) {{< /quote >}}

*{{< xe class="hpc" >}}* ではデータタイプに、QF, BF8, TF32 を新しくサポートする。  
TF32 (Tensor Float32) は 19-bit長のデータフォーマットで、**NVIDIA A100 GPU** がサポートしているデータタイプだが、QF,BF8 については詳細不明。  
QF は Quadword Float、BF8 は BF16 (BFloat16) からダイナミックレンジと精度を減らしたフォーマットとは考えられるが。  

 > 		    GED_DATA_TYPE_qf,      ///< XE.HPC.A
 > 		    GED_DATA_TYPE_bf8,     ///< XE.HPC
 > 		    GED_DATA_TYPE_tf32,    ///< XE.HPC
 >
 > {{< quote >}} [intel-graphics-compiler/ged_enumerations.h at 2887a020540d935b2f0872b5bd56c0336bc19570 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2887a020540d935b2f0872b5bd56c0336bc19570/visa/iga/GEDLibrary/GED_external/build/autogen-ia32/ged_enumerations.h#L148-L150) {{< /quote >}}

### {{< xe class="hpc" >}}-XT と {{< xe class="hpc" >}}-XL {#xl-xt}

*{{< xe class="hpc" >}}* では XT と XL の 2種類があることがコメントで示されている。  
これまで公開されてきた *{{< xe class="hpc" >}} アーキテクチャ* を採用する **Ponte Vecchio** は Base Tile を 2基持ち、Co-EMIB でそれらを接続する 2-Tile 構成だったが、Raja Koduri 氏は 1-Tile バージョンを提供する予定があることを話している。[^1t-pvc]  
AMD風の命名法則に従えば、{{< xe class="hpc" >}}-XL が 1-Tile、{{< xe class="hpc" >}}-XT が 2-Tile 構成の **Ponte Vecchio** となるのかもしれない。  

 > 		    XE_HPC      = IGA_XE_VER_ORDINAL(1, 4), // XeHPC-XT, preserved (1, 3) for XeHPC-XL
 >
 > {{< quote >}} [intel-graphics-compiler/Types.hpp at 2887a020540d935b2f0872b5bd56c0336bc19570 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2887a020540d935b2f0872b5bd56c0336bc19570/visa/iga/IGALibrary/IR/Types.hpp#L52) {{< /quote >}}

[^1t-pvc]: [ASCII.jp：Ponte VecchioとIntel Arcに関する疑問をRaja Koduri氏が回答　インテル GPUロードマップ (1/3)](https://ascii.jp/elem/000/004/069/4069704/)

{{< ref >}}
 * [ASCII.jp：Ponte VecchioとIntel Arcに関する疑問をRaja Koduri氏が回答　インテル GPUロードマップ (1/3)](https://ascii.jp/elem/000/004/069/4069704/)
 * [コンピュータアーキテクチャの話(153) スコアボード方式による管理(1) | TECH+](https://news.mynavi.jp/article/architecture-153/)
 * [intel-graphics-compiler/SWSBSetter.cpp at 2887a020540d935b2f0872b5bd56c0336bc19570 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2887a020540d935b2f0872b5bd56c0336bc19570/visa/iga/IGALibrary/IR/SWSBSetter.cpp#L344)
 * [Intel® Processors with Intel® UHD Graphics](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-gpu-optimization-guide/top/gen-arch.html)
{{< /ref >}}
