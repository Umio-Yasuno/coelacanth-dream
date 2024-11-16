---
title: "gfx940 で新たにサポートされる命令と XF32フォーマット"
date: 2022-03-19T16:59:40+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "gfx940", "gfx90a", "CDNA_2", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Stanislav Mekhanoshin 氏より、新たな CDNA系 GPUID *gfx940* のサポートに向けたさらなるパッチが LLVM に投稿されている。  
それらパッチによれば、*gfx940* では行列演算命令、MFMA (Matrix-Fused-Multiply-Add) 系命令として新たな命令がいくつか追加され、またいくつかは *Aldebaran/gfx90a/CDNA 2* からサポートが外される。  

前提情報として、AMD GPU における MFMA命令名は `V_MFMA_{CD}_{M}x{N}x{K}_{AB}` のフォーマットからなり、`{AB}` は入力データフォーマット、`{CD}` は一時的な結果と最終的な出力データフォーマットを示し、`{M,N,K}` は行列のサイズを示す。  
`{M,N}` は値に {4,16,32} のどれかを取り、`{K}` は {1,2,4,8,16,32} から取る。  

{{< pindex >}}
 * [削除される MFMA系命令](#removed)
 * [追加される MFMA系命令](#added)
    * [XF32](#xf32)
    * [SMFMAC](#smfmac)
 * [V_MOV_B64](#v_mov_b64)
{{< /pindex >}}

## 削除される MFMA系命令 {#removed}
*gfx940* では MFMA系命令の従来の対応範囲から、`V_MFMA_I32_{32X32X8,16X16X16}_I8` と `V_MFMA_F32_{*}_BF16` が外される。  
それら命令は *Arcturus/gfx908/CDNA 1* の世代からサポートされていた。  
`V_MFMA_I32_{32X32X4,16X16X4,4X4X4}_I8` と、*Aldebaran/gfx90a/CDNA 2* からサポートしている `V_MFMA_F32_{*}BF16_1K` はサポートが残されている。  

* [[AMDGPU] Disable some MFMA instructions on gfx940 · llvm/llvm-project@4570527](https://github.com/llvm/llvm-project/commit/4570527e7210b4f379f20af36ba4026ddafd852f)

## 追加される MFMA系命令 {#added}
*gfx940* では新たに 4個の MFMA命令、`V_MFMA_I32_{32X32X16,16X16X32}_I8` と `V_MFMA_F32_{32X32X4,16X16X8}_XF32` に対応する。  
また、行列演算命令 (MAI, Matrix Arithmetic Instructions) として新たに SMFMAC 命令が追加された。  

* [[AMDGPU] New gfx940 mfma instructions · llvm/llvm-project@27439a7](https://github.com/llvm/llvm-project/commit/27439a764230e5eb54568b2fc053a20c9005970f)
* [[AMDGPU] Support gfx940 smfmac instructions · llvm/llvm-project@6e3e14f](https://github.com/llvm/llvm-project/commit/6e3e14f600afa1fa64a699df97c8bbac6d0f8b5a)

### XF32 {#xf32}
命令名の XF32 サフィックスは TF32 データフォーマットであることを示す。  
TF32 そのままを命令名に使用しなかった理由には、ハードウェア仕様によるものと Stanislav Mekhanoshin 氏は説明している。[^xf32]  
TF32 は 19-bits 長のデータフォーマットであり、データのダイナミックレンジは FP32 と同じ 8-bits、精度は FP"16" と同じ 10-bits となっている。(1-bit は符号 [sign] bit )  

TF32 データフォーマットには、NVIDIA GPU では *NVIDIA A100* からサポートしており、Intel GPU でも *Ponte Vecchio ({{< xe class="hpc" >}})* がサポートする。[^xe-hpc-tf32]  
*Ponte Vecchio ({{< xe class="hpc" >}})* は他に QF, BF8 といったデータフォーマットをサポートするとしているが、詳細はまだ不明。  
{{< link >}} [Xe-HPC のサポートが intel-graphics-compiler、oneDNN に追加される | Coelacanth's Dream](/posts/2021/10/29/xe-hpc-onednn/#icache) {{< /link >}}

*Aldebaran/gfx90a/CDNA 2* では TF32 データフォーマットへのサポートが行われなかったが、*gfx940* でサポートされ、他社 GPU と揃うことが明らかとなった。  

[^xf32]: [[AMDGPU] New gfx940 mfma instructions · llvm/llvm-project@27439a7](https://github.com/llvm/llvm-project/commit/27439a764230e5eb54568b2fc053a20c9005970f)
[^xe-hpc-tf32]: [Accelerating the Convergence of HPC and AI at Exascale - Intel Communities](https://community.intel.com/t5/Blogs/Products-and-Solutions/HPC/Accelerating-the-Convergence-of-HPC-and-AI-at-Exascale/post/1334987)

### SMFMAC {#smfmac}
*gfx940* でサポートされる SMFMAC 命令には、`V_SMFMAC_F32_{16X16X32,32X32X16}_F16`、`V_SMFMAC_F32_{16X16X32,32X32X16}_BF16`、`V_SMFMAC_I32_{16X16X64,32X32X32}_I8` の 6個がある。  
ただ正直、SMFMAC 命令が具体的にどういった命令で、MFMA 命令とどう異なるのかは、自分ではまだ読み取れていない。MFMA 命令から `idx` を指定する入力オペランドが追加されてはいるが。  

 > 		+    case Intrinsic::amdgcn_smfmac_f32_16x16x32_f16:
 > 		+    case Intrinsic::amdgcn_smfmac_f32_32x32x16_f16:
 > 		+    case Intrinsic::amdgcn_smfmac_f32_16x16x32_bf16:
 > 		+    case Intrinsic::amdgcn_smfmac_f32_32x32x16_bf16:
 > 		+    case Intrinsic::amdgcn_smfmac_i32_16x16x64_i8:
 > 		+    case Intrinsic::amdgcn_smfmac_i32_32x32x32_i8: {
 > 		+      // vdst, srcA, srcB, srcC, idx
 > 		+      OpdsMapping[0] = getAGPROpMapping(MI.getOperand(0).getReg(), MRI, *TRI);
 > 		+      OpdsMapping[2] = getVGPROpMapping(MI.getOperand(2).getReg(), MRI, *TRI);
 > 		+      OpdsMapping[3] = getVGPROpMapping(MI.getOperand(3).getReg(), MRI, *TRI);
 > 		+      OpdsMapping[4] = getAGPROpMapping(MI.getOperand(4).getReg(), MRI, *TRI);
 > 		+      OpdsMapping[5] = getVGPROpMapping(MI.getOperand(5).getReg(), MRI, *TRI);
 > 		+      break;
 > 		+    }
 >
 > {{< quote >}} [[AMDGPU] Support gfx940 smfmac instructions · llvm/llvm-project@6e3e14f](https://github.com/llvm/llvm-project/commit/6e3e14f600afa1fa64a699df97c8bbac6d0f8b5a) {{< /quote >}}

## V_MOV_B64 {#v_mov_b64}
*gfx940* には `V_MOV_B64` 命令のサポートも追加されている。  
`V_MOV_{*}` 命令はデータの移動を行う基本的な命令であり、命令名のサフィックスはデータのサイズ (ビット幅) を示す。  
*Aldebaran/gfx90a/CDNA 2* で `FullRateFP64` が実装されたが、`V_MOV_B64` はサポートされていなかった。`PackedFP32Ops` の一部として `V_PK_MOV_B32` 命令のサポートはしていたため、`V_MOV_B64` のサポートが *gfx940* にずれ込んだ何らかの理由があるのかもしれないが、まあ不明である。  

* [[AMDGPU] Add v_mov_b64 gfx940 opcode · llvm/llvm-project@e7b362d](https://github.com/llvm/llvm-project/commit/e7b362d75d2a3fdf67550b88738c708a33eec3cc)

まだ *gfx940* のサポートは途中だが、*RDNA 2/GFX10.3* 世代と同様に `FeatureMadMacF32Insts` (`V_{MAD,MAC,MADAK,MADMK}_F32`) の削除が行われていることと合わせ、対応命令の追加だけでなく整理も *gfx940* で行われているような印象を受ける。  
{{< link >}} [新たな CDNA系の GPUID 「gfx940」 | Coelacanth's Dream](/posts/2022/03/01/llvm-gfx940/) {{< /link >}}

{{< ref >}}
* [Accelerating the Convergence of HPC and AI at Exascale - Intel Communities](https://community.intel.com/t5/Blogs/Products-and-Solutions/HPC/Accelerating-the-Convergence-of-HPC-and-AI-at-Exascale/post/1334987)
* ["AMD Instinct MI200" Instruction Set Architecture: Reference Guide - CDNA2_Shader_ISA_4February2022.pdf](https://developer.amd.com/wp-content/resources/CDNA2_Shader_ISA_4February2022.pdf)
* [dgemm を使用した行列の乗算](https://jp.xlsoft.com/documents/intel/mkl/11.3/mkl113_tutorial_c/GUID-36BFBCE9-EB0A-43B0-ADAF-2B65275726EA.htm)
{{< /ref >}}
