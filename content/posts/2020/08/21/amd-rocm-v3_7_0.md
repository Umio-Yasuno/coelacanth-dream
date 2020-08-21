---
title: "AMD、ROCm v3.7.0 をリリース & 今後のリリース予定"
date: 2020-08-21T13:22:56+09:00
draft: false
tags: [ "Radeon", "ROCm" ]
keywords: [ "", ]
categories: [ "Software", ]
noindex: false
# summary: ""
---

AMD は 大規模GPUコンピューティングのオープンソースプラットフォーム **ROCm** の ver3.7.0 をリリースした。  
{{< link >}} [RadeonOpenCompute/ROCm at roc-3.7.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-3.7.x) {{< /link >}}

Ubuntu 20.04 をサポートし、  
マルチスレッドによる並列プログラミングAPIである [OpenMP](https://www.openmp.org/) の AMDGPU にオフロードする機能をサポートするオープンソースコンパイラ [AOMP](https://github.com/ROCm-Developer-Tools/aomp) の機能強化や、  
GPU間通信に用いられる [RCCL(ROCm Communications Collective Library)](https://github.com/ROCmSoftwarePlatform/rccl) で *NCCL(NVIDIA Communications Collective Library) API* との互換性を確保、  
[rocSOLVER](https://github.com/ROCmSoftwarePlatform/rocSOLVER) での二対角行列の特異値分解や、[rocSPARSE](https://github.com/ROCmSoftwarePlatform/rocSPARSE)での疎な行列の実行のサポートが、今回のリリースにおける主な更新内容となる。  

## 今後のリリース予定

また、開発者の [mvermeulen](https://github.com/mvermeulen)氏は [ROCmSoftwarePlatform/AMDMIGraphX](https://github.com/ROCmSoftwarePlatform/AMDMIGraphX) の中で、今後のリリース目標を挙げており、**ROCm 3.9** は September 9th (2020/09/09) に、**ROCm 3.10** は October 7th (2020/10/07) と設定している。[^rocm-3_9-10]  

[^rocm-3_9-10]: [Release 0.8 definition · Issue #579 · ROCmSoftwarePlatform/AMDMIGraphX](https://github.com/ROCmSoftwarePlatform/AMDMIGraphX/issues/579)


AMD が [Financial Analyst Day 2020](https://ir.amd.com/events/event-details/financial-analyst-day-2020) で掲げた、  
**ROCm** がソフトウェアスタックとして、エクサスケールスパコンにおける機械学習、HPC (高性能コンピューティング) を満たすソリューションとなる **ROCm 4.0** はそれ以降、2020/11、2020/12 にリリースされるものと思われる。  

{{< figure src="/image/2020/08/21/rocm-4_0_0.webp" caption="画像出典: [Driving GPU Leadership](https://ir.amd.com/static-files/321c4810-ffe2-4d6c-863f-690464c033a9)" >}}


余談だが、`rocm-dkms-firmware_3.7-20_all.deb` パッケージには既に次世代 AMD GPU [Arcturus](/tags/arcturus)、[Sienna Cichlid](/tags/sienna_cichlid) のファームウェアが含まれている。  
ただ、GPU ASIC の情報が一部組み込まれている `*gpu_info.bin` は含まれておらず、個人的な憶測では、ファームウェアのリバースエンジニアリングや逆アセンブルを防ぐためではないかと思われる。  
どの口が言うんだと自分でも思いはするが、ファームウェアから GPU 情報を読み取った結果が話題になり過ぎたのかもしれない。  
一応同梱されているライセンスにはリバース・エンジニアリングや逆アセンブルは許されていないが、ライセンスにはあまり詳しくない。ただ、オープンに出来ない理由がどこかしらあるわけで、やはりオープンソースコードのみをネタにするのが安全なように思う。
