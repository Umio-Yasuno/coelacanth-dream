---
title: "RDNA 3 アーキテクチャの CU の若干の考察"
date: 2022-11-09T00:04:07+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", "RadeonSI", "RADV" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

オープンソースドライバーや LLVM のコードから、まだ詳細が公開されていない *RDNA 3 アーキテクチャ* の CU に関する若干の考察を個人的にしてみる。  
当然ながら、AMD よりホワイトペーパーといった *RDNA 3 アーキテクチャ* の詳細を記述した資料が公開された場合、ここでの考察は否定される可能性がある。  
また、*RadeonSI (OpenGL)* ドライバーや LLVM は AMD の開発者が直接開発を進め、パッチを公開しており、公式資料以外では少ない信頼できる情報源だと考えているが、同時にコードは更新し続けるものであるため、あくまでも現時点での情報を用いた考察となる。  

## Unified Compute Unit {#cu}
先日マージされた AMD の Marek Olšák 氏によるパッチの中で、ピーク FP32 性能 (GFlops) の計算が以下のように変更された。  
ここでのピーク FP32 性能は、主にプロファイラ向けの情報となる。  

 >         -   info->max_gflops = info->num_cu * 128 * info->max_gpu_freq_mhz / 1000;
 >         +   info->max_gflops = (info->gfx_level >= GFX11 ? 256 : 128) * info->num_cu * info->max_gpu_freq_mhz / 1000;
 >
 > {{< quote >}} [amd: add cosmetic gfx10 and gfx11 changes (8b66c0ac) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d) {{< /quote >}}

ピーク FP32 性能 (GFlops) は CU 数とピーク動作クロック (GHz, MHz/1000)、そして CU あたりの SIMDレーン数あるいは SP (Shader Processor) 数と FMA (ADD+MUL) の 2 ops を乗算して求められる。  
この変更から言えるのは、まず *RDNA 3 アーキテクチャ* の `libdrm` API 経由で返される CU 数は製品仕様と同じ数であること。[^rx7900xtx]  
それと CU あたりの演算性能 (Flops) は、従来の倍となる 256 Flops となるということだろう。  

[^rx7900xtx]: [AMD Radeon™ RX 7900 XTX | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-7900xtx#product-specs)

しかし、**RX 7900 XTX** の製品仕様で Stream Processors 数は、CU あたり 64基とした SP 6144基となっている。  
また一方で、同ソースファイル (`src/amd/common/ac_gpu_info.c`) 内の、CU あたりの SIMDユニット数 (`num_simd_per_compute_unit`) や Workgroup あたりの LDS (Local Data Share) サイズ (`lds_size_per_workgroup`) は変更されずに据え置かれている。  
CU あたりの SIMDユニット数については LLVM でも *RDNA 系アーキテクチャ* は 2基となっている。[^llvm-eu_per_cu]  

[^llvm-eu_per_cu]: [llvm-project/AMDGPUBaseInfo.cpp at 7425077e31c9b505103a98299a728bc496bd933c · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/7425077e31c9b505103a98299a728bc496bd933c/llvm/lib/Target/AMDGPU/Utils/AMDGPUBaseInfo.cpp#L780-L789)

 >            info->num_simd_per_compute_unit = info->gfx_level >= GFX10 ? 2 : 4;
 >         
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d/src/amd/common/ac_gpu_info.c#L1319) {{< /quote >}}
 >
 >            /* LDS is 64KB per CU (4 SIMDs), which is 16KB per SIMD (usage above
 >             * 16KB makes some SIMDs unoccupied).
 >             *
 >             * LDS is 128KB in WGP mode and 64KB in CU mode. Assume the WGP mode is used.
 >             */
 >            info->lds_size_per_workgroup = info->gfx_level >= GFX10 ? 128 * 1024 : 64 * 1024;
 >         
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d/src/amd/common/ac_gpu_info.c#L1319) {{< /quote >}}

CU あたりの SIMDユニット数は 2基のままで、CU あたりの演算性能は従来の倍になっている。  
そうなると、ベクタレジスタは SIMDユニットごとに持つため、演算性能に対するベクタレジスタは減っていることとなる。  
*Navi31/gfx1100* と *Navi32/gfx1101* は、SIMDユニットあたりでは従来から 1.5倍のベクタレジスタを持つが、これは減った分を補うためのものという見方もできる。  
また、*Navi33/gfx1102* と *Phoenix APU/gfx1103* は追加のベクタレジスタを持たないため、演算性能に対するベクタレジスタは従来の半分となる。  

 >            if (info->family == CHIP_GFX1100 || info->family == CHIP_GFX1101)
 >               info->num_physical_wave64_vgprs_per_simd = 768;
 >            else if (info->gfx_level >= GFX10)
 >               info->num_physical_wave64_vgprs_per_simd = 512;
 >            else
 >               info->num_physical_wave64_vgprs_per_simd = 256;
 >         
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d/src/amd/common/ac_gpu_info.c#L1313-1318) {{< /quote >}}

*RDNA 1/2 アーキテクチャ* における LDS には、WGP 内の CU 2基で LDS 128KB (2x64KB) を共有し、ある CU から見た時に近い LDS と遠い LDS が存在する WGP mode と、CU 内の LDS 64KB のみを使う CU mode がある。  
*RDNA 3 アーキテクチャ* で LDS と WGP/CU mode がどうなっているかは不明であり、発表時にも LDS には触れられていなかった。  

### Cache {#cache}

先に amd-gfx メーリングリストの中で AMD の Liu, Aaron 氏によって明かされていたが[^rdna_3-cache]、AMD の Marek Olšák 氏によるパッチでは *RDNA 3 アーキテクチャ* の L1ベクタキャッシュ (RDNA L0ベクタキャッシュ) を 32KiB、GL1データキャッシュ (RDNA L1データキャッシュ) を 256KiB としている。  
L1ベクタキャッシュは CU ごとに持ち、GL1データキャッシュは Shader Array ごとに持つ。  
ベクタレジスタと異なり、それらキャッシュサイズは *RDNA 3 アーキテクチャ* 全体で共通するのだろう。  

 >         +   if (info->gfx_level >= GFX11)
 >         +      info->l1_cache_size = 32768;
 >         +   else
 >         +      info->l1_cache_size = 16384;
 >
 > {{< quote >}} [amd: add cosmetic gfx10 and gfx11 changes (8b66c0ac) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d) {{< /quote >}}
 >
 >             if (info->gfx_level >= GFX10) {
 >                fprintf(f, "    l0_cache_size = %i KB\n", DIV_ROUND_UP(info->l1_cache_size, 1024));
 >         -      fprintf(f, "    l1_cache_size = %i KB\n", 128);
 >         +      fprintf(f, "    l1_cache_size = %i KB\n", info->gfx_level >= GFX11 ? 256 : 128);
 >             } else {
 >                fprintf(f, "    l1_cache_size = %i KB\n", DIV_ROUND_UP(info->l1_cache_size, 1024));
 >             }
 >         
 > {{< quote >}} [amd: add cosmetic gfx10 and gfx11 changes (8b66c0ac) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/8b66c0ac7605b1f0e0f7af4cff1c8e0381b16b4d) {{< /quote >}}

[^rdna_3-cache]: [L0ベクタキャッシュと GL1データキャッシュが増やされる RDNA 3/GFX11 APU | Coelacanth's Dream](/posts/2022/09/02/gfx11-l0c-gl1c/)

### VOPD (Dual issue wave32) {#vopd}
*RDNA 3 アーキテクチャ* では `VOPD (Dual issue wave32)` 命令に対応し、イベントでは *64 Dual Issue Stream Processors*, *Dual issue SIMD units* として発表していた。  
各レーンあるいは SP は Dual issue に対応し、4 Flops 相当の演算性能を持つと考えれば、SP数と演算性能については説明が付く。  
CU あたりの SIMDユニット数も合わせて考えると、*64 Dual Issue Stream Processors* と発表されていたが、CU 内部は *2x Dual issue SIMD32 units* となっているのではないだろうか。  

ここで疑問に思うのは Wave モードの違いである。LLVM へのパッチによれば `VOPD` 命令は Wave32 にのみ対応し、Wave64 ではサポートしない。[^llvm-vopd]  
また `VOPD` 命令は限られた範囲の対応する命令を最大 2種類組み合わせる方式であるため、すべての命令を `VOPD` 命令に変換するといったこともできないように思われる。  
また、これはコンパイラ側の対応で対処が可能であるため実際に問題になることは少ないと思うが、`VOPD` 命令以外を Wave32 で発行した場合は Dual issue SIMD unit の実質半分は休んだ状態となるのだろうか。  

[^llvm-vopd]: [GFX11 でサポートされる VOPD (Dual issue wave32) 命令の範囲と特徴 | Coelacanth's Dream](/posts/2022/06/21/gfx11-vopd-instruction/)

*RDNA 1/2 アーキテクチャ* では Wave64 を 2xWave32 に分割する方式を用いていたが、*RDNA 3 アーキテクチャ* も同様かは不明。  
仮に *Dual issue SIMD unit* が Wave64 を 1サイクルで発行可能とすると、Wave64 に対する Wave32 (Dual issue) の利点はさらなる実行レイテンシの削減と言えるかもしれない。  
*RDNA 1 アーキテクチャ* で新たに Wave32 に対応したことについて、AMD は利点の 1つとして低レイテンシをアピールしていた。  
*RDNA 1/2 アーキテクチャ* の Wave64 を 2xWave32 に分割する方式では、発行可能なスカラ命令は 1回に制限していたため、*RDNA 3 アーキテクチャ* で Wave64 を 1サイクルで発行可能としても実行方式の変更による影響は小さいように思える。  


{{< ref >}}
 * [AMD，新世代GPU「Radeon RX 7000」シリーズを発表。第1弾製品はRadeon RX 7900 XTXとRadeon RX 7900 XTで12月13日発売](https://www.4gamer.net/games/660/G066019/20221104001/)
 * [ASCII.jp：AMD「Radeon RX 7000シリーズ」の発表内容を、もう少し深く読み解く (1/5)](https://ascii.jp/elem/000/004/111/4111884/)
 * [radv: Speculatively tune RT pipelines for GFX11. (!19288) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/19288)
{{< /ref >}}
