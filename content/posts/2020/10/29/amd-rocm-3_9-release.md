---
title: "AMD ROCm v3.9 リリース"
date: 2020-10-29T10:49:45+09:00
draft: false
tags: [ "ROCm", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

 * [RadeonOpenCompute/ROCm at roc-3.9.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.9.x)

今バージョンの新機能、変更点は以下。  

 * OpenMPの GPUオフロード機能の強化
   * LLVMをバックエンドとする Clang/++ コンパイラがサポートされている
 * ROCm-SMI (ROCm System Management Information) の改良
   * ハードウェアトポロジの詳細が確認できるようになっている
   * プロセスごとの VRAM使用量、SDMA (System DMA) の使用状況、CU (Compute Unit) の利用率も確認できる、CU の利用率については *Vega /GFX9* 世代の GPU のみのサポート
 * 計算ライブラリの最適化

今バージョンではサポートする GPU の追加等は行なわれなかった。  
ROCm-SMI の改良点等から、HPC 用途に向けたバージョンと見られる。  

また、今バージョンではクリーンインストールが推奨されているが、自環境でアップグレードしても特に問題は出ていない。  
複数バージョンインストール時の取扱いが改良され、`LD_LIBRARY_PATH` を環境変数に設定して切り替えるようになった。  
自分の環境では `.bashrc` に以下のように記述し、`ROCM_PATH` でバージョンを制御するようにしている。  

 >      export ROCM_PATH="/opt/rocm-3.9.0"
 >      export PATH=$HCC_HOME/bin:$HIP_PATH/bin:$PATH
 >      export OPENCL_LIBS="${ROCM_PATH}/opencl/lib"
 >      export LIBOPENCL="${ROCM_PATH}/opencl/lib"
 >      export LD_LIBRARY_PATH="${ROCM_PATH}/opencl/lib":$LD_LIBRARY_PATH
 >      export LD_LIBRARY_PATH="${ROCM_PATH}/lib":$LD_LIBRARY_PATH
 >      export LD_LIBRARY_PATH="${ROCM_PATH}/include":$LD_LIBRARY_PATH
 >      export OPENCL_HEADERS="${ROCM_PATH}/opencl/include"
 >      export OPENCL_INCLUDES="${ROCM_PATH}/opencl/include"
 >      
 >      export PATH=${ROCM_PATH}/bin:$PATH
 >      export PATH=${ROCM_PATH}/opencl/bin:$PATH

