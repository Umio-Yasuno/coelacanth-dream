---
title: "FP8 と BF8 に対応する AMD GFX940/CDNA 3 と BF8 に対応する Intel Xe-HPC"
date: 2022-07-26T13:37:09+09:00
draft: false
categories: [ "Hardware", "Intel", "AMD", "GPU" ]
tags: [ "gfx940", "CDNA_3", "Xe-HPC" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Stanislav Mekhanoshin (rampitec) 氏により、*GFX940/CDNA 3* で追加された FP8/BF8 フォーマット命令の対応を LLVM に追加するパッチが投稿された。パッチはすでに [llvm/llvm-project](https://github.com/llvm/llvm-project) の main ブランチに取り込まれている。  

 * [[AMDGPU] Support for gfx940 fp8 conversions · llvm/llvm-project@9fa5a6b](https://github.com/llvm/llvm-project/commit/9fa5a6b7e8a292ec91b844a622836d2990ef5796)
 * [[AMDGPU] Support for gfx940 fp8 mfma · llvm/llvm-project@2695f0a](https://github.com/llvm/llvm-project/commit/2695f0a688e9d26fcb0f3a4b686a2783f2eb145c)
 * [[AMDGPU] Support for gfx940 fp8 smfmac · llvm/llvm-project@523a99c](https://github.com/llvm/llvm-project/commit/523a99c0eb0331680905e9ef6fbdd114f4ee7a47)

*GFX940/CDNA 3* の FP8 フォーマット対応自体は AMD Financial Analyst Day にて発表されており、*GFX940/CDNA 3* は AI学習性能が *GFX90A/CDNA 2* の 8倍という比較において、用いるデータフォーマットは *GFX940/CDNA 3* が FP8、*GFX90A/CDNA 2* は FP16 となっていた。  

## FP8/BF8 {#fp8-bf8}
FP8/BF8 フォーマットを用いた演算命令は `MFMA (Matrix-Fused-Multiply-Add), SMFMAC` 命令と miSIMDユニットで対応し、通常の SIMDユニット側では変換命令にのみ対応する。  
FP8/BF8 フォーマットは 8-bit ということはわかるが、指数部 (range, exponent) と仮数部 (precision, mantissa) がそれぞれどうなっているかはパッチ内には記述されていない。  
8-bit の浮動小数点フォーマットには *NVIDIA H100 GPU* が先に対応しており、そして指数部 5-bit 仮数部 2-bit の E5M2 と指数部 4-bit 仮数部 3-bit の E4M3 の 2種類に対応している。  
*GFX940/CDNA 3* における `FP8/BF8` がそれらと同様かはまだ不明だが、仮にそうだとすると、精度を寄った E4M3 は `FP8` に、ダイナミックレンジを重視した E5M2 は `BF8` となるだろうか。  

行列のレイアウトは `MFMA` 命令では `16x16x32, 32x32x16` に、`SMFMAC` 命令では `16x16x64, 32x32x32` に対応している。  
アキュムレータと最終的な出力フォーマットは `F32` のみに対応する。  
命令名の末尾は `_{BF8,FP8}_{BF8,FP8}` からなる 4つのパターンがあるが、既存の命令名からそれぞれの入力フォーマットと考えられる。  
`MFMA, SMFMAC` 命令ではこれまで入力される 2個の行列は同じフォーマットだったが、`FP8` と `BF8` に限っては異なる組み合わせに対応するのだろう。  

 > 		+  defm V_MFMA_F32_16X16X32_BF8_BF8 : MAIInst<"v_mfma_f32_16x16x32_bf8_bf8", "F32_I64_X32",    int_amdgcn_mfma_f32_16x16x32_bf8_bf8>;
 > 		+  defm V_MFMA_F32_16X16X32_BF8_FP8 : MAIInst<"v_mfma_f32_16x16x32_bf8_fp8", "F32_I64_X32",    int_amdgcn_mfma_f32_16x16x32_bf8_fp8>;
 > 		+  defm V_MFMA_F32_16X16X32_FP8_BF8 : MAIInst<"v_mfma_f32_16x16x32_fp8_bf8", "F32_I64_X32",    int_amdgcn_mfma_f32_16x16x32_fp8_bf8>;
 > 		+  defm V_MFMA_F32_16X16X32_FP8_FP8 : MAIInst<"v_mfma_f32_16x16x32_fp8_fp8", "F32_I64_X32",    int_amdgcn_mfma_f32_16x16x32_fp8_fp8>;
 > 		+  defm V_MFMA_F32_32X32X16_BF8_BF8 : MAIInst<"v_mfma_f32_32x32x16_bf8_bf8", "F32_I64_X16",    int_amdgcn_mfma_f32_32x32x16_bf8_bf8>;
 > 		+  defm V_MFMA_F32_32X32X16_BF8_FP8 : MAIInst<"v_mfma_f32_32x32x16_bf8_fp8", "F32_I64_X16",    int_amdgcn_mfma_f32_32x32x16_bf8_fp8>;
 > 		+  defm V_MFMA_F32_32X32X16_FP8_BF8 : MAIInst<"v_mfma_f32_32x32x16_fp8_bf8", "F32_I64_X16",    int_amdgcn_mfma_f32_32x32x16_fp8_bf8>;
 > 		+  defm V_MFMA_F32_32X32X16_FP8_FP8 : MAIInst<"v_mfma_f32_32x32x16_fp8_fp8", "F32_I64_X16",    int_amdgcn_mfma_f32_32x32x16_fp8_fp8>;
 >
 > {{< quote >}} [[AMDGPU] Support for gfx940 fp8 mfma · llvm/llvm-project@2695f0a](https://github.com/llvm/llvm-project/commit/2695f0a688e9d26fcb0f3a4b686a2783f2eb145c#diff-547f0be0fdf8fd86523d5e13e9b40af7ff77e1882be85f4836d5374c99a39094) {{< /quote >}}

変換命令は `F32 <-> FP8,BF8` の変換に対応し、パックド命令にも対応している。  
それ以外のフォーマット、`F16, BF16, F64` との変換には対応していない。  

 > 		+let SubtargetPredicate = HasFP8Insts, mayRaiseFPException = 0,
 > 		+    SchedRW = [WriteFloatCvt] in {
 > 		+  let Constraints = "$vdst = $vdst_in", DisableEncoding = "$vdst_in" in {
 > 		+    defm V_CVT_PK_FP8_F32 : VOP3Inst<"v_cvt_pk_fp8_f32", VOP3_CVT_PK_F8_F32_Profile>;
 > 		+    defm V_CVT_PK_BF8_F32 : VOP3Inst<"v_cvt_pk_bf8_f32", VOP3_CVT_PK_F8_F32_Profile>;
 > 		+  }
 > 		+
 > 		+  // These instructions have non-standard use of op_sel. In particular they are
 > 		+  // using op_sel bits 2 and 3 while only having two sources. Therefore dummy
 > 		+  // src2 is used to hold the op_sel value.
 > 		+  let Constraints = "$vdst = $src2", DisableEncoding = "$src2" in {
 > 		+    defm V_CVT_SR_FP8_F32 : VOP3Inst<"v_cvt_sr_fp8_f32", VOP3_CVT_SR_F8_F32_Profile>;
 > 		+    defm V_CVT_SR_BF8_F32 : VOP3Inst<"v_cvt_sr_bf8_f32", VOP3_CVT_SR_F8_F32_Profile>;
 > 		+  }
 >
 > {{< quote >}} [[AMDGPU] Support for gfx940 fp8 conversions · llvm/llvm-project@9fa5a6b](https://github.com/llvm/llvm-project/commit/9fa5a6b7e8a292ec91b844a622836d2990ef5796) {{< /quote >}}

## Intel Xe-HPC {#xe-hpc}
[intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler) によれば、Intel GPU も *{{< xe class="hpc" >}} アーキテクチャ* で `BF8` に対応している。  
*{{< xe class="hpc" >}} A0* が対応するフォーマットとして `QF (Quarter Float)` も以下引用部にあるが、他の部分の記述から `QF` のフォーマット名は `BF8` にリネームされたことがわかる。  
*{{< xe class="hpc" >}}* のリビジョン更新と共に `QF` を `BF8` に変えた理由は明かされていないが、他ベンダーにおける 8-bit 浮動小数点フォーマットと統一するためという理由が想像できる。  
2022/07/07 付で、AI 向けの IPU (Intelligence Processing Unit) を設計している Graphcore より、AMD と Qualcomm と共に 8-bit 浮動小数点フォーマットの標準化を提案したというリリースが出されており、このことが関係しているのではないかと思われる。  

 * [GraphcoreとAMDがQualcomm Technologiesのサポートを得てAI向け8ビット浮動小数点形式を業界標準として提案](https://www.graphcore.ai/ja-jp/posts/graphcore-and-amd-propose-8-bit-fp-ai-standard-with-qualcomm-support)

*{{< xe class="hpc" >}}* では現状 `BF8` のみの対応とし、`FP8` の記述はない。  
変換命令については、*AMD GFX940/CDNA 3* では `F32 <-> FP8,BF8` のみだったが、*Intel {{< xe class="hpc" >}}* では `F16 (HF) <-> BF8 (QF)` のみの対応関係となっている。  
Intel GPU における `BF8` のフォーマットは E5M2 とされている。  

 > 		     BF16 = 9,   // bfloat16 (1, 8, 7)
 > 		     FP16 = 10,  // half (1, 5, 10)
 > 		+    BF8 = 11,   // bfloat8 (1, 5, 2)
 > 		     TF32 = 12,  // TensorFloat (1, 8, 10), 19 bits
 >
 > {{< quote >}} [Internal feature · intel/intel-graphics-compiler@2a6f568](https://github.com/intel/intel-graphics-compiler/commit/2a6f568e4fd813cd3f4c23b833a20daabc2c1369) {{< /quote >}}

 > 		    GED_DATA_TYPE_nf,      ///< 11, TGL, XE.HP, XE.HPG
 > 		    GED_DATA_TYPE_bf,      ///< XE.HP, XE.HPG, XE.HPC.A, XE.HPC
 > 		    GED_DATA_TYPE_qf,      ///< XE.HPC.A
 > 		    GED_DATA_TYPE_bf8,     ///< XE.HPC
 > 		    GED_DATA_TYPE_tf32,    ///< XE.HPC
 >
 > {{< quote >}} [intel-graphics-compiler/ged_enumerations.h at 56e33b2fdf33bfe545ae68fb62f3ec1583f3592a · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/56e33b2fdf33bfe545ae68fb62f3ec1583f3592a/visa/iga/GEDLibrary/GED_external/build/autogen-ia32/ged_enumerations.h#L146-L150) {{< /quote >}}
 >
 > 		// [Deprecated. Use bf8 version instead]
 > 		// Note: qf is renamed to bf8. Those cvt builtins are the same as
 > 		//       those cvt builtins between bf8 and hf (qf is deprecated)
 > 		//       Once all apps/tests move to use bf8 cvt, those can be deleted
 > 		//
 > 		// quarter float <--> half float conversion
 > 		//    qf : no igc type for qf yet. Use char as *opaque* type for it.
 > 		//
 > 		// hf -> qf conversion builtins (rte rounding mode)
 > 		char   __builtin_IB_hftoqf_1 (half   a) __attribute__((const));
 > 		char2  __builtin_IB_hftoqf_2 (half2  a) __attribute__((const));
 > 		char3  __builtin_IB_hftoqf_3 (half3  a) __attribute__((const));
 > 		char4  __builtin_IB_hftoqf_4 (half4  a) __attribute__((const));
 > 		char8  __builtin_IB_hftoqf_8 (half8  a) __attribute__((const));
 > 		char16 __builtin_IB_hftoqf_16(half16 a) __attribute__((const));
 > 		
 > 		// qf -> hf conversion builtins (precise conversion)
 > 		half   __builtin_IB_qftohf_1 (char   a) __attribute__((const));
 > 		half2  __builtin_IB_qftohf_2 (char2  a) __attribute__((const));
 > 		half3  __builtin_IB_qftohf_3 (char3  a) __attribute__((const));
 > 		half4  __builtin_IB_qftohf_4 (char4  a) __attribute__((const));
 > 		half8  __builtin_IB_qftohf_8 (char8  a) __attribute__((const));
 > 		half16 __builtin_IB_qftohf_16(char16 a) __attribute__((const));
 >
 > {{< quote >}} [intel-graphics-compiler/IGCBiF_Intrinsics_Dpas.cl at fc234c1eb2a9659097a114ce9ca1c71c6e8a0e57 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/fc234c1eb2a9659097a114ce9ca1c71c6e8a0e57/IGC/BiFModule/Implementation/IGCBiF_Intrinsics_Dpas.cl#L457-L481) {{< /quote >}}

{{< ref >}}
 * [AMD Financial Analyst Day 個人的まとめ | Coelacanth's Dream](/posts/2022/06/11/amd-financial-analyst-day/)
 * [西川善司の3DGE：Hopper世代のNVIDIA製GPU「GH100」のアーキテクチャを深掘りしてみる](https://www.4gamer.net/games/623/G062364/20220328110/)
 * [GraphcoreとAMDがQualcomm Technologiesのサポートを得てAI向け8ビット浮動小数点形式を業界標準として提案](https://www.graphcore.ai/ja-jp/posts/graphcore-and-amd-propose-8-bit-fp-ai-standard-with-qualcomm-support)
{{< /ref >}}
