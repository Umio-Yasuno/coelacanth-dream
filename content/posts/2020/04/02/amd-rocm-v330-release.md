---
title: "AMD、ROCm v3.3.0 リリース"
date: 2020-04-02T14:52:49+09:00
draft: false
tags: [ "Radeon", "ROCm" ]
keywords: [ "", ]
categories: [ "Software", ]
noindex: false
---

レポジトリには既に反映されており、インストール可能となっている。  
今回も新しいGPUのサポート追加はなく、

 * 複数バージョンのインストールが可能に
 * GPUのプロセス情報表示機能の改良
 * 3Dプーリングレイヤーのサポート
 * ONNX(Open Neural Network eXchange)のサポート強化
 
が主な変更点となっている。  

中でも、*GPUのプロセス情報表示機能の改良* は複数のGPUに対応したものと見られ、[^1]  
*3Dプーリングレイヤーのサポート* では、これによりResNext3D等の3D畳込みネットワークの実行が可能になったとしている。

[^1]: [SMI: Add --showpidgpus flag to display GPUs used by a PID · RadeonOpenCompute/ROC-smi@feb4412](https://github.com/RadeonOpenCompute/ROC-smi/commit/feb441259fd933d8cc5224810157d0e9a34af2bf)

[Arcturus](/tags/arcturus)対応がメインになるであろう ROCm v3.2 を飛ばしての ROCm v3.3 のリリースとなったが[^2]、リリース方式に何か変更があったのだろうか。  
複数バージョンのインストールが可能となったため、新GPU対応版と既存GPUの機能強化版で分ける、といった感じに。  
ROCm v3.2 は *Arcturus* のテスト版としてリリース、ROCm v3.4 で正式対応といったシナリオも考えられる。  

[^2]: [merge develop into master for ROCm 3.2 Feature complete by amcamd · Pull Request #983 · ROCmSoftwarePlatform/rocBLAS](https://github.com/ROCmSoftwarePlatform/rocBLAS/pull/983)

{{< ref >}}

 * [RadeonOpenCompute/ROCm at roc-3.3.0](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.3.0#Whats-New-in-This-Release)  
 	* <https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.3.0#Whats-New-in-This-Release>

{{< /ref >}}
