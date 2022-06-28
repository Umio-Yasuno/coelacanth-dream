---
title: "GFX11/RDNA 3 では行列積和演算命令、WMMA (Wave Matrix Multiply-accumulate) をサポート"
date: 2022-06-29T05:49:48+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

最近になって AMD の開発者より LLVM に *GFX11* の新機能、新命令のサポートを追加するパッチが続いて投稿されている。  
そして今回、AMD の Joe Nash 氏より、*GFX11* の新命令として `WMMA (Wave Matrix Multiply-accumulate)` のサポートを追加するパッチが投稿された。  

 * [⚙ D128756 [AMDGPU] gfx11 WMMA instruction support](https://reviews.llvm.org/D128756)

## WMMA (Wave Matrix Multiply-accumulate) {#wmma}
`WMMA` は行列の積和演算を行う命令となる。  

 > 		// WMMA (Wave Matrix Multiply-Accumulate) intrinsics
 > 		//
 > 		// These operations perform a matrix multiplication and accumulation of
 > 		// the form: D = A * B + C .
 >
 > {{< quote >}} [⚙ D128756 [AMDGPU] gfx11 WMMA instruction support](https://reviews.llvm.org/D128756) {{< /quote >}}

`WMMA` 命令には以下の 6個が存在し、*CDNA 系アーキテクチャ* でサポートされている `MFMA (Matrix-Fused-Multiply-Add)` 命令と同様のフォーマットとすると、`V_WMMA_{CD}_{M}x{N}x{K}_{AB}` と置いた場合、`{AB}` は入力データフォーマット、`{CD}` は加算する行列と最終的な出力データフォーマットを、`{M,N,K}` は行列のサイズを示す。  
入力データフォーマットには `F16,BF16,IU8,IU4`、出力データフォーマットには `F32,F16,BF16,I32` をサポートしていると考えられる。  

`MFMA` と `WMMA` の違いには、`MFMA` は入力出力共に `F64` フォーマットをサポートし、`WMMA` は `F64` をサポートしないが入力データフォーマットに `IU4` をサポートしている点、  
`MFMA` は複数の行列サイズに対応するが `WMMA` は 16x16x16 に固定されている点が挙げられる。  

 > 		defm V_WMMA_F32_16X16X16_F16   : VOP3P_Real_WMMA <0x040>;
 > 		defm V_WMMA_F32_16X16X16_BF16  : VOP3P_Real_WMMA <0x041>;
 > 		defm V_WMMA_F16_16X16X16_F16   : VOP3P_Real_WMMA <0x042>;
 > 		defm V_WMMA_BF16_16X16X16_BF16 : VOP3P_Real_WMMA <0x043>;
 > 		defm V_WMMA_I32_16X16X16_IU8   : VOP3P_Real_WMMA <0x044>;
 > 		defm V_WMMA_I32_16X16X16_IU4   : VOP3P_Real_WMMA <0x045>;
 >
 > {{< quote >}} [⚙ D128756 [AMDGPU] gfx11 WMMA instruction support](https://reviews.llvm.org/D128756) {{< /quote >}}

*GFX11* では Wave32 と Wave64 の両方に対応するが、`WMMA` 命令では Wave のサイズによって割り当てられるレジスタリソースが異なっている。  

 > 		class VOPProfileWMMA<VOPProfile P, string Suffix, RegisterOperand _Src01RC64, bit _HasClamp, bit _HasOpSel> : VOP3P_Profile<P> {
 > 		  let DstRC = !if(!eq(Suffix, "_w32"), VDst_256, VDst_128);
 > 		  let Src0RC64 = _Src01RC64;
 > 		  let Src1RC64 = _Src01RC64;
 > 		  let Src2RC64 = !if(!eq(Suffix, "_w32"), VISrc_256_f64, VISrc_128_f32);
 > 		  let HasClamp = _HasClamp;
 > 		  let HasOpSel = _HasOpSel;
 > 		  let IsWMMA = 1;
 > 		}
 >
 > {{< quote >}} [⚙ D128756 [AMDGPU] gfx11 WMMA instruction support](https://reviews.llvm.org/D128756) {{< /quote >}}

NVIDIA GPU 向けの CUDA にも Tesor Core が対応する命令、それを利用するための API として `WMMA (Warp Matrix Multiply Accumulate)` はあり、言葉としては Warp か Wave かの違いとなる。  
AMDGPU *GFX11* における `WMMA` は現状行列のサイズが 16x16x16 で固定されているのに対し、NVIDIA CUDA における `WMMA` はアーキテクチャにもよるが、16x16x16 以外に 32x8x16 や 8x8x4 (FP64)、8x8x128 (B1) にも対応するといった違いもある。  

Intel GPU では、`DPAS (Dot Product Accumulate Systolic)` 命令、XMX ({{< xe >}} Matrix eXtension) とも呼ぶユニットが NVIDIA Tensor Core に相当する。[^intel-gpu]  
Fused EU 構成を採用する *{{< xe class="hp/hpg" >}}* では内部的に行列ブロックを 2つ分解して実行する `DPASW (Wide)` 命令もサポートしている。[^intel-dpas]  

[^intel-gpu]: [Intel® Iris® Xe GPU Architecture](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-gpu-optimization-guide/top/xe-arch.html)
[^intel-dpas]: [DG2-G12, DG2 L3 banks, SIMD width | Coelacanth's Dream](/posts/2022/01/16/xe_hpg-hpc-eu-inst/#dpas_w)

機能や名称から、*GFX11* の `WMMA` 命令は NVIDIA GPU をある程度意識して追加された命令のように思える。  
*CDNA 系アーキテクチャ* がサポートする `MFMA` 命令は miSIMD (Machine Intelligence SIMD) で実行する方式だったが、*GFX11* でも同様のユニットを用いるかは不明。  
ただ今回のパッチでは通常のベクタレジスタ (VGPR) を指定しているため、miSIMD と専用の AccVGPR を用いる *CDNA 系アーキテクチャ* とは異なる方式であることが可能性として考えられる。  
*CDNA 1/gfx908/MI100/Arcturus* では SIMD と VGPR、miSIMD と AccVGPR が分けて実装されている。  
*CDNA 2/gfx90a/MI200/Aldebaran* からは VGPR と AccVGPR が統合され、`MFMA` 命令を実行するにはオフセットを指定して分割する必要があるが、それぞれのレジスタサイズは柔軟に変更可能となった。  
{{< link >}} [CDNA 2 アーキテクチャでは Unified Register Files を実装 | Coelacanth's Dream](/posts/2021/07/01/aldebaran-unified-vgpr/) {{< /link >}}

{{< ref >}}
 * [Programming Guide :: CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#wmma)
 * ["AMD Instinct MI200" Instruction Set Architecture: Reference Guide - CDNA2_Shader_ISA_4February2022.pdf](https://developer.amd.com/wp-content/resources/CDNA2_Shader_ISA_4February2022.pdf)
 * [Tensorコアを使ってみた - Fixstars Tech Blog /proc/cpuinfo](https://proc-cpuinfo.fixstars.com/2018/10/tensorcore/)
{{< /ref >}}
