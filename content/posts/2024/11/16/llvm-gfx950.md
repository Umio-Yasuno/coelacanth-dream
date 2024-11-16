---
title: "LLVM に CDNA 系の GPU ID、gfx950 が追加"
date: 2024-11-16T07:45:04+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "LLVM", "GFX9", "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

LLVM に *gfx950* と新命令のサポートを追加するプルリクエストが AMD の Matt Arsenault 氏によって公開されている。  
*gfx950* は *CDNA 3* の GPU ID である *gfx94x* と基本的な機能、ISA は共通しており、同様に *CDNA* 系の GPU ID だと考えられる。  
ただ今回のプルリクエストでは FP4 と FP6 のデータフォーマットをサポートする命令は見られず、*MI350/CDNA 4* ではなく、*MI300/CDNA 3* のマイナーチェンジ版か *MI325X* に割り当てられた GPU ID の可能性が高いように思う。  

 * [AMD 、データセンター向け AI イノベーションを加速し業界をリード AMD Instinct GPU の新たなロードマップを発表](https://www.amd.com/ja/newsroom/press-releases/2024-06-02-amd-ai-amd-instinct-gpu.html)

 >         +def FeatureISAVersion9_5_Common : FeatureSet<
 >         +  !listconcat(FeatureISAVersion9_4_Common.Features,
 >         +  [FeatureFP8Insts,
 >         +   FeatureFP8ConversionInsts,
 >         +   FeatureCvtFP8VOP1Bug,
 >         +   FeatureGFX950Insts
 >         +   ])>;
 >         +
 >
 > {{< quote >}} [AMDGPU: Add gfx950 subtarget definitions by arsenm · Pull Request #116307 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116307) {{< /quote >}}

 * [AMDGPU: Add gfx950 subtarget definitions by arsenm · Pull Request #116307 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116307)
 * [AMDGPU: Add subtarget features for minimum3/maximum3 instructions by arsenm · Pull Request #116308 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116308)

## 追加される行列演算命令と削除される TF32 のサポート {#mfma}
`V_MFMA_F32_16X16X32_F16, V_MFMA_F32_32X32X16_F16` 命令が *gfx950* で追加された。  
`A (16x32) * B (32x16) + C (16x16)` と `A (32x16) * B (16x32) + C (32x32)` 形式の行列演算命令は *CDNA 3 (gfx940)* では、入力にサポートするデータフォーマットが BF8 か FP8 のみだったため、そこに FP16 が追加されたと見ることができる。  

 * [AMDGPU: Add first gfx950 mfma instructions by arsenm · Pull Request #116312 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116312)

*gfx950* では何故か XF32 フォーマットの行列演算命令のサポートが削除されている。  
XF32 フォーマットは TF32 フォーマットを指し、フォーマット名が異なるのはハードウェアの仕様によるものと過去に説明されていた。  
TF32 (XF32) は 19-bits 長のデータフォーマットであり、FP32 から仮数部 (精度) を FP16 と同じ 10-bits に減らしたものとなる。  

*CDNA 3* では TF32 フォーマットの行列のピーク性能は FP32 よりも高いものとなっていたため (NVIDIA GPU ほどのピーク性能差ではないが)、サポートする意義はあったと思うが。  

 * [gfx940 で新たにサポートされる命令と XF32フォーマット | Coelacanth's Dream](/posts/2022/03/19/amd-gfx90a-gfx940-diff/#xf32)

 >         +    case GK_GFX950:
 >         +      Features["gfx950-insts"] = true;
 >         +      [[fallthrough]];
 >              case GK_GFX942:
 >              case GK_GFX941:
 >              case GK_GFX940:
 >                Features["fp8-insts"] = true;
 >                Features["fp8-conversion-insts"] = true;
 >         -      Features["xf32-insts"] = true;
 >         +      if (Kind != GK_GFX950)
 >         +        Features["xf32-insts"] = true;
 >                [[fallthrough]];
 >
 > {{< quote >}} [AMDGPU: Add gfx950 subtarget definitions by arsenm · Pull Request #116307 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116307) {{< /quote >}}

## `V_PRNG_B32`
確率的丸め (Stochastic Rounding) のための命令、`V_PRNG_B32` が *gfx950* で追加された。  
詳しくないため今回検索して初めて知ったが、機械学習において最近注目されているアプローチらしい。  
丸め処理によって小さな値が潰れてしまうのを防ぐことができる。  
AWS の機械学習専用チップ、AWS Trainium にもハードウェア実装されている。  

 * [AMDGPU: Add v_prng_b32 instruction for gfx950 by arsenm · Pull Request #116310 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116310)
    * [Stochastic Rounding Implicitly Regularizes Tall-and-Thin Matrices - Instant Read & Key Insights](https://linnk.ai/insight/%E6%95%B0%E5%AD%A6/%E7%A2%BA%E7%8E%87%E7%9A%84%E4%B8%B8%E3%82%81%E3%81%8C%E8%83%8C%E3%81%AE%E9%AB%98%E3%81%84%E8%A1%8C%E5%88%97%E3%82%92%E6%9A%97%E9%BB%99%E7%9A%84%E3%81%AB%E6%AD%A3%E5%89%87%E5%8C%96%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%82%92%E7%A4%BA%E3%81%99-z8t928zI/)
 * [独自設計チップ AWS Trainium 搭載 Amazon EC2 Trn1 インスタンスで ML トレーニングを高速実行（基礎編） | Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/aws-trainium-amazon-ec2-trn1-ml-training-part1/)

## `V_CVT_F32_BF16`
BF16 フォーマットを FP32 に変換する `V_CVT_F32_BF16` 命令が *gfx950* で追加されている。  
*CDNA 3 (gfx94x)* では BF8 を FP32 に変換する命令や、BF16 の行列を入力に取り、FP32 で出力する行列演算命令はサポートしていたが、単純に BF16 を FP32 に変換する命令はサポートしていなかった。  

 * [AMDGPU: Add V_CVT_F32_BF16 for gfx950 by arsenm · Pull Request #116311 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116311/files)

## 160 KB の LDS {#lds}
*gfx950* では LDS (Local Data Share) のサイズが 160 KB とされている。  
従来の AMD GPU は LDS サイズが CU あたり 64 KB となっており、*RDNA* 系の GPU における WGP モードでも最大 128 KB だったため、*gfx950* の LDS は大幅に増やされていることがわかる。  

LDS は CU ごとに持つ高速なスクラッチメモリであり、サイズが大きくなれば活用できる状況が増え、性能が向上する可能性がある。  
また、コンパイラは要求されるベクタレジスタ数に対してハードウェアのベクタレジスタが不足している場合、LDS に一部を割り当てる場合もある。  

 * [AMDGPU: Increase the LDS size to support to 160 KB for gfx950 by arsenm · Pull Request #116309 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116309/)
 * [Register pressure in AMD CDNA2™ GPUs - amd-lab-notes - AMD GPUOpen](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-register-pressure-readme/)
