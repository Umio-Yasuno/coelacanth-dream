---
title: "AMD、ROCm v3.1.0リリース"
date: 2020-02-28T11:28:02+09:00
draft: false
tags: [ "Radeon", "ROCm" ]
keywords: [ "", ]
categories: [ "Software", ]
noindex: false
---

新しいGPUのサポート追加等はなく、  
インストール方法の変更[^4]、Vega7nm( Vega20 ) のRAS[^1]機能のサポート、SLURM[^2]のサポート追加が主。  
大きな更新は[Arcturus](/tags/arcturus)のサポートが予想されるROCm v3.2までお預けだろう。  
しかしSLURMはクラスタのリソース管理、スケジューリングを行なうためのソフトウェアであり、HPCにROCmを採用するための改良は確実に進んでいる。  

[^4]: 従来のインストール先だった */opt/rocm* をシンボリックリンクとし、実ファイルは */opt/rocm-\<version\>* にインストールするようになった。これで複数バージョンのインストール、切り替えが容易となるはずだ。  

[Release Tag for the roc 3.1 release · RadeonOpenCompute/ROCm](https://github.com/RadeonOpenCompute/ROCm/releases/tag/rocm-3.1)  
<https://github.com/RadeonOpenCompute/ROCm#Whats-New-in-This-Release>  

[^1]: Reliability, Accessibility, and Serviceability <br> <https://github.com/RadeonOpenCompute/ROCm#reliability-accessibility-and-serviceability-support-for-vega7nm>
[^2]: Simple Linux Utility for Resource Management <br> [SchedMD/slurm: Slurm: A Highly Scalable Workload Manager](https://github.com/SchedMD/slurm)

OpenCLドライバも更新されているため、手持ちのRX560 (Polaris11, gfx803)[^5] で darktable-cltest (ver 3.0.0)[^3] を実行してみたが、問題はなかった。  
それにしても、OpenCLだけでもGFX10世代のサポートを追加してもいいように思う。  
自分が検証していないだけで、実行可能だったりするのだろうか？  

[^3]: [darktable-org/darktable: darktable is an open source photography workflow application and raw developer](https://github.com/darktable-org/darktable)
[^5]: CPU: Ryzen 5 2600, Memory: 16GB, M/B: GA-A320M-HD2 F50a,OS: Debian GNU/Linux 10 (buster), Linux Kernel: 5.4.21 (custom)
