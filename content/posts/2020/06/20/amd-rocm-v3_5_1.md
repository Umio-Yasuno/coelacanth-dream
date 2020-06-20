---
title: "AMD、ROCm v3.5.1 をリリース & 問題への対処法"
date: 2020-06-20T18:27:17+09:00
draft: false
tags: [ "Radeon", "ROCm" ]
keywords: [ "", ]
categories: [ "Software", ]
noindex: false
---

AMD は一部問題の修正と機能追加を施した **ROCm v3.5.1** をリリースした。  
{{< link >}}[RadeonOpenCompute/ROCm at roc-3.5.1](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.5.1){{< /link >}}

自環境でも早速アップデートしてみたが、OpenCLの実行で新たに問題が発生した。  
その対処法は後述。  

## ROCm 3.5.1 の変更点

 * HIP-ROCclr(Radeon Open Compute Common Language Runtime) にストリームの優先度を問い合わせるAPI、`hipStreamGetPriority` の公開  
 * マルチGPU環境において長期稼働させた場合に発生するメモリアクセスエラーの修正  
 * NCCL(NVIDIA Collective Communication Library) 2.7 のサポート
 * RCCL(ROCm Collective Communication Library) のアップデート
   * Gater, Scatter, All-To-All の一括操作をサポート
   * ネットワークプロキシのプロファイリングをサポート

## 問題の対処法
ROCm v3.5.1 というマイナーアップデートだからなのか、ライブラリやバイナリを置くディレクトリを `/opt/rocm-3.5.1` にまとめるのではなく、更新があったのだろう一部を `/opt/rocm-3.5.1` に、  
そうでないものは `/opt/rocm-3.5.0` に残されている。  
それにより、AMD GPUが対応する中間命令を収めたビットコードが `/opt/rocm-3.5.1/lib` に移ったため、追加で環境変数を指定する必要があった。  
自環境では以下のようなコマンドを `.bashrc` に記述している。  

```
export ROCM_PATH="/opt/rocm-3.5.0"
export ROCM_PATH_3_5_1="/opt/rocm-3.5.1

export PATH=$HCC_HOME/bin:$HIP_PATH/bin:$PATH
export OPENCL_LIBS="${ROCM_PATH}/opencl/lib"
export LIBOPENCL="${ROCM_PATH}/opencl/lib"

export LD_LIBRARY_PATH="${ROCM_PATH}/opencl/lib":$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=${ROCM_PATH_3_5_1}/lib:$LD_LIBRARY_PATH

export OPENCL_HEADERS="${ROCM_PATH}/opencl/include"
export OPENCL_INCLUDES="${ROCM_PATH}/opencl/include"
export PATH=${ROCM_PATH}/bin:$PATH
export PATH=${ROCM_PATH}/opencl/bin:$PATH

```

ROCm v3.3.0 から最新版へのシンボリックリンクを作成するようになったのに、複数ディレクトリに分けるというのはどうなのだろう。  
自分の環境、やり方が悪かっただけ？  

{{< ref >}}

 * [コンピュータアーキテクチャの話(313) 1つの命令で複数の演算器を動かす「SIMD」 | マイナビニュース](https://news.mynavi.jp/article/architecture-313/)

{{< /ref >}}
