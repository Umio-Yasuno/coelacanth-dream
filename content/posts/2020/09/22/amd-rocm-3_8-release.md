---
title: "AMD ROCm v3.8.0 をリリース & 非公式なAMD GPUファームウェアレポジトリを作ってみた"
date: 2020-09-22T12:37:37+09:00
draft: false
tags: [ "ROCm", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

 * [RadeonOpenCompute/ROCm at roc-3.8.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.8.x)

ハードウェアでは、**Radeon Pro VII** (Vega 7nm Workstation /Vega20 GL-XE) のサポートが追加されている。  

今バージョンではインストール方法に、前バージョンからのアップグレードではなく、クリーンインストールが推奨されている。というより、今バージョンへのアップグレードはサポートされないとしている。  
自分の場合はとりあえず適当に `# apt purge comgr rocm*` 実行した後、たまに使う OpenCL のため、`# apt install rocm-opencl` でインストールし直したが今の所問題は無い。  

今バージョンでは、FORTRANプログラムから GPUカーネルにアクセスするためのインターフェイスライブラリ [hipfort](https://github.com/ROCmSoftwarePlatform/hipfort) と、  
AMD GPU で構築されたクラスタの管理を簡潔に行なうためのツール、*RDC (ROCm Data Center) Tool* が追加されている。  
*RDC* は各GPUのモニタリング機能を備えており、GPU/メモリクロック、GPU/メモリ使用率、GPU温度、PCIeバス帯域、ECCエラーのカウントを確認することができる。  

[^rocm-release-plan]: [Release 0.8 definition · Issue #579 · ROCmSoftwarePlatform/AMDMIGraphX](https://github.com/ROCmSoftwarePlatform/AMDMIGraphX/issues/579)

また、**ROCm v3.8.0** のパッケージ内には [Navy Flounder](/tags/navy_flounder) のファームウェアが追加されていた。  
最新 AMD GPUのサポートで心配されることの 1つは、クローズドソースであるファームウェアがいつリリースされるか、[kernel/git/firmware/linux-firmware.git](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/) レポジトリにいつアップロードされるかなのだが、**ROCm** や Linux向け Pro ドライバー等では先行して追加されていたりする。  
そこで、AMDGPU のファームウェアはリバースエンジニアリング、逆コンパイル、逆アセンブルの禁止を守り、ライセンスを含めれば再配布が許可されているはずなので、以下のレポジトリを作ってみた。  
{{< comple >}}活用する機会が無ければいいが。{{< /comple >}}  
{{< link >}} [Umio-Yasuno/unofficial-amdgpu-firmware-repo](https://github.com/Umio-Yasuno/unofficial-amdgpu-firmware-repo) {{< /link >}}

