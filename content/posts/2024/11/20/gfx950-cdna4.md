---
title: "gfx950 は FP6/FP4 フォーマットをサポート、MI350X/CDNA 4 か"
date: 2024-11-20T05:19:45+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "CDNA", "GFX9", "LLVM", "CDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日 AMD の Matt Arsenault 氏 によって公開された LLVM への *gfx950* 関連のプルリクエストを取り上げた。  
その後、さらなるプルリクエストが公開され、*gfx950* では FP6/FP4 フォーマットを入力に取る命令がサポートされることが明らかとなった。  
先日の記事では *gfx950* が **Instinct MI325X** 関連の GPU ID ではないかと推測したが、これにより **Instinct MI350X** の GPU ID だと考えられる。  

 * [AMDGPU: Define v_mfma_f32_{16x16x128|32x32x64}_f8f6f4 instructions by arsenm · Pull Request #116723 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116723)
 * [AMDGPU: Add v_mfma_ld_scale_b32 for gfx950 by arsenm · Pull Request #116722 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116722)
 * [AMDGPU: Add V_CVT_PK_BF16_F32 for gfx950 by arsenm · Pull Request #116678 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116678)
 * [AMDGPU: Define v_mfma_f32_32x32x16_bf16 for gfx950 by arsenm · Pull Request #116679 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116679)
 * [AMDGPU: Handle gfx950 96/128-bit buffer_load_lds by arsenm · Pull Request #116681 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116681)
 * [AMDGPU: Handle gfx950 global_load_lds_* instructions by arsenm · Pull Request #116680 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116680)

## FP6/FP4 {fp6_fp4}
*gfx950* では `V_MFMA_SCALE_F32_{16X16X128,32X32X64}_F8F6F4` 命令と `V_MFMA_F32_{16X16X128,32X32X64}_F8F6F4` 命令をサポートする。  
それらの命令では入力に FP8/FP6/FP4 フォーマットの行列を取り、FP32 フォーマットに出力する。  

 * [AMDGPU: Define v_mfma_f32_{16x16x128|32x32x64}_f8f6f4 instructions by arsenm · Pull Request #116723 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116723)
 * [AMDGPU: Add v_mfma_ld_scale_b32 for gfx950 by arsenm · Pull Request #116722 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/116722)

`V_MFMA_SCALE_F32_{16X16X128,32X32X64}_F8F6F4` 命令ではスケール係数を指定することができる。  
FP8/FP6/FP4 フォーマットはダイナミックレンジが狭いため、オーバーフロー/アンダーフローを防ぎ、精度を維持するのにスケーリング処理が使われる。  

 * [大規模言語モデル (LLM)における低精度数値表現 - Speaker Deck](https://speakerdeck.com/pfn/20240508-hpckenkyukai-pfn-llm)
 * [OCP_Microscaling Formats (MX) v1.0 Spec_Final.pdf](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf)
 * [Transformer Engine ではじめる FP8 Training (導入編) - NVIDIA 技術ブログ](https://developer.nvidia.com/ja-jp/blog/introduction-to-fp8-training-using-transformer-engine/)
