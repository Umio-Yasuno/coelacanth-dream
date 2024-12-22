---
title: "レジスタファイルサイズが倍になり、Int64 のサポートが復活する Intel Xe2"
date: 2023-10-07T14:54:26+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Lunar_Lake", "Xe2", "Xe-HPC" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel GPU オープンソースドライバーでは現在、Kernel Mode Driver (i915, intel-xe)、User Mode Driver (Iris [OpenGL], Anv [Vulkan]) ともに、*Lunar Lake* のサポート作業が進められている。  
*Lunar Lake* は GPU コアに *Xe2 アーキテクチャ*、ディスプレイエンジンに *Xe2-LPD* を採用するとされている。  

 * [Draft: Intel LNL platform (!24377) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/24377)

## Intel Xe2 {#xe2}
先日 Francisco Jerez 氏により公開された、コンパイラバックエンドに *Xe2 アーキテクチャ* のサポートを追加するマージリクエストでは従来のアーキテクチャからの変更点に触れている。  

 * [intel: Second round of compiler patches enabling Xe2 platforms: SWSB synchronization support. (!25514) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/25514)

まず、*Xe2 アーキテクチャ* では FP64 と (Extended) Math 命令は in-order タイプのパイプラインが処理するようになる。ここでのパイプラインは実行ポートと同義と思われる。  
Francisco Jerez 氏はコメントで、*Xe-LP, Xe-HPG, Xe-LPG* における Math 実行ポートのように out-of-order タイプの命令のトラッキングを行うよりも in-order の方が効率的だとしている。  

FP64 命令は Math 実行ポートではなく Long 実行ポートに実装される。  
Intel GPU は *Gen11 アーキテクチャ* から FP64 の実装が外され、*Xe-LPG アーキテクチャ* で再度実装されるようになった。  
ただし *Gen9 アーキテクチャ* が FP32 演算性能に対して 1/4 の FP64 演算性能だったのに対し、*Xe-LPG アーキテクチャ* では 1/16 となっている。[^xe-lpg-fp64]  

*Xe2 アーキテクチャ* では Int64 命令のサポートも復活するが、これは Long 実行ポートではなく Int 実行ポートに実装される。  

こうした変更から見るに *Xe2 アーキテクチャ* は *Xe-HPC アーキテクチャ* に近いものとなっている。  
Intel は *Xe-HPC アーキテクチャ* の EU 内部構成等に関する資料を公開していないが、[intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler) のソースコードを読むに、Math 実行ポートの in-order 化、Long 実行ポート (FP64) の追加、Int 実行ポートの Int64 サポートは *Xe-HPC アーキテクチャ* で実装されていた。[^igt-regdeps]  
推測だが、*Xe (Xe1)* 世代では *Xe-LP, Xe-HPG, Xe-HP, Xe-HPC, Xe-LPG* ではそれぞれで EU 内部構成が異なっていたが、*Xe2* 世代では出来る限り統一する方針となったのかもしれない。  
AMD GPU における RDNA 系と CDNA 系のように、FP64 演算性能や行列演算専用命令の有無で差別化するのだろうか。  

[^igt-regdeps]: <https://github.com/intel/intel-graphics-compiler/blob/master/visa/iga/IGALibrary/IR/RegDeps.cpp>
[^xe-lpg-fp64]: [Graphics and Media Deep Dive](https://www.intel.com/content/www/us/en/content-details/788848/graphics-and-media-deep-dive.html)

## レジスタファイルサイズが倍に {#grf}
*Xe-LP アーキテクチャ* は EU あたり 7スレッド、スレッドあたりのレジスタファイルは 128エントリ (4KiB)、EU 全体では 28KiB のレジスタファイルを持つ構成となっていた。  
それが *Xe-HPG アーキテクチャ* では EU あたり 8スレッドをサポートするようになり、それにあわせてレジスタファイルも増えたため、EU 全体では 32KiB のレジスタファイルを持つようになった。  
*Xe-HPC アーキテクチャ* ではエントリあたりのサイズが 64Byte となり、EU 全体では 64KiB を持つ。  
レジスタファイルサイズが倍ということから、ここにおいても *Xe-HPC アーキテクチャ* と同様の変更を *Xe2 アーキテクチャ* で施した可能性がある。  

 * [intel/compiler: initial compiler changes for xe2/lnl register size change; adds reg_unit() (!25020) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/25020)

{{< ref >}}
 * <https://github.com/intel/intel-graphics-compiler/blob/master/visa/iga/IGALibrary/IR/RegDeps.cpp>
 * [2つの非同期パイプラインが追加される Xe-HP EU | Coelacanth's Dream](/posts/2021/04/02/xehp-add-two-async-pipeline/)
 * [Introduction to the Xe-HPG Architecture](https://www.intel.com/content/www/us/en/developer/articles/technical/introduction-to-the-xe-hpg-architecture.html)
{{< /ref >}}
