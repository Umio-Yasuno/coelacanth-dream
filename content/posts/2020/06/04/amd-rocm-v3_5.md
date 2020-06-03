---
title: "AMD、ROCm v3.5 をリリース"
date: 2020-06-04T03:20:01+09:00
draft: false
tags: [ "ROCm", "Radeon" ]
keywords: [ "", ]
categories: [ "Software" ]
noindex: false
---

ユーザーはレポジトリからダウンロード、インストールが可能となっている。  
Linuxディストリごとのインストール方法は以下から。  
{{< link >}}[AMD ROCm Installation Guide v3.5.0 — ROCm Documentation 1.0.0 documentation](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html){{< /link >}}

今リリースにおいてサポートが追加された GPU は無かった。  
*Arcturus* をサポートする v3.2 、最適化が施される v3.4 を飛ばして v3.5 のリリースだが、何だか *Arcturus /MI100* の正式発表のタイミングを計っているように感じる。  
以前、2020/06/02 〜 2020/06/06 に開催される [COMPUTEX TAIPEI](https://www.computextaipei.com.tw/en_US/index.html) で発表されるのではないかと予測したが、  
その COMPUTEX TAIPEI が 2020/09/28 〜 2020/09/30 に延期してしまった。  
近いイベントには [ISC 2020](https://www.isc-hpc.com/)(2020/06/22 〜 2020/06/25) があるが、[プログラム](https://2020.isc-program.com/contributors/)を見るに大々的な発表はないように思える。  
さていつになるのか。  

## ROCm 3.5 の主な変更点

 * HCC(Heterogeneous Compute Compiler) が非推奨となり、HIP-Clang compiler に置き換えられた
 * HIP-HCCランタイムが HIP-ROCclr(Radeon Open Compute Common Language Runtime) に変更
   * HIPランタイムAPI は ROCclr の上層に実装され、ROCclr は抽象化レイヤーとして機能する
 * OpenCLランタイムが OpenCL2.2 を拡張サポート
 * MIOpen でコンパイル済みのカーネルをオプションで提供、これにより起動時間が短縮される
 * プロファイラツール rocprof の実行に SQLite3モジュールを含む Python が必要に
 * 新たなツール ROCgdb(ROCm Debugger) が追加、CPUとGPUのコード両方に使用可能
   * ROCdbgapi(ROCm Debugger API) は [Vega10](/tags/vega10) と [Vega20(Vega 7nm)](/tags/vega20) アーキテクチャをサポート
 * 新たに RPP(Radeon Performance Primitives) ライブラリを追加、HIP と OpenCL をバックエンドとした AMD CPU + AMD GPU 向けの高性能コンピューティングライブラリとなる

その他、詳細は以下。  
{{< link >}}<https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.5.0#Whats-New-in-This-Release>{{< /link >}}

リリースされてから早速アップグレードしてみたが、微妙にうまくいかず、それが解決しても OpenCL ライブラリ、ヘッダーへのパスを再設定する必要があった。  
確か、以前はパスに `x86_64-linux-gnu` が付いてたはずだが、今リリースではなくなっていた。  
ただ最近になって Debian のリリースレポジトリを testing に変更したため、ROCm側の問題とは言い切れないところがある。  
一応、`.bashrc` または `.xsessionrc` に以下を記述することで再び OpenCLプログラムが実行可能となったため、参考にしていただけたら幸いだ。  

```
export ROCM_PATH="/opt/rocm"
export OPENCL_LIBS="${ROCM_PATH}/opencl/lib"
export LIBOPENCL="${ROCM_PATH}/opencl/lib"
export LD_LIBRARY_PATH=${ROCM_PATH}/opencl/lib:$LD_LIBRARY_PATH
export OPENCL_HEADERS="${ROCM_PATH}/opencl/include"
export OPENCL_INCLUDES="${ROCM_PATH}/opencl/include"

```
