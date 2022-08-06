---
title: "レイトレーシング対応が進む Intel Vulkan ドライバー、Windows ドライバーと共有のライブラリを導入"
date: 2022-08-06T02:43:28+09:00
draft: false
categories: [ "Software", "Intel", "GPU" ]
tags: [ "DG2", "Xe-HPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

オープンソースで開発されている Intel GPU 向け Vulkan ドライバー *Anvil (ANV)* では、現在 Vulkan API におけるレイトレーシングのサポートが進められている。  
コンパイラバックエンドにおける Intel GPU、主に *DG2/Alchemist* に向けたレイトレーシング機能のサポートは 2020/10/29 に公開され、2020/11/25 にマージされたマージリクエストで実装されていたが、その時はまだ Vulkan API のレイトレーシング拡張が策定途中であったため、それ以上の実装はされていなかった。[^rt-backend]  

先日、*Intel ANV* ドライバーにおけるレイトレーシングパイプラインのスタックサイズを設定する部分で、今までは BO (Buffer Object) を CPU側のシステムメモリに割り当てていたのを GPU VRAM (ローカルメモリ) に割り当てるよう変更するパッチが Intel の Lionel Landwerlin 氏によって投稿された。[^rt-scratch]  
パッチに付けられていたコメントに *「Like a 100x (not joking) improvement.」* とあったこともあり、話題となっていたが、それまでシステムメモリへの割り当てにより性能への影響が気付かれなかった理由には、Vulkan レイトレーシングの実装がまだ途中であったことがあるだろう。  
なお、同じく Lionel Landwerlin 氏によって、可能な場合は常にローカルメモリを使用するパッチ、マージリクエストが公開されており、同様の問題への対策が行われている。[^local-mem]  
また DXR に対応する Windows ドライバーでは以前より実装されていたことから、同様の問題は無かったと考えられる。  

[^rt-backend]: [intel/compiler: Add support for ray-tracing (!7356) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7356)
[^local-mem]: [anv: deprecate use of LOCAL_MEM flag internally (!17873) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/17873/)
[^rt-scratch]: [anv: allocate RT scratch in local memory (!17674) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/17674)

## GRL, Graphics Library for Ray-tracing {#grl}

現在 Intel のソフトウェアエンジニア、Jason Ekstrand 氏、Jordan Justen 氏、Lionel Landwerlin 氏、Ivan Briano 氏、Caio Oliveira 氏らによって進められている *Intel ANV* ドライバーへのレイトレーシング実装では *DG2/Alchemist* を対象としている。  

 * [anv: Initial ray-tracing support (!16970) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/16970/)

そしてレイトレーシング処理に必要なトラバーサルシェーダや交差判定、ソートを GPU 上で実行するための手段として、OpenCLカーネルと独自フォーマットのメタカーネルが採用されている。  
メタカーネルとそのライブラリは *GRL, Graphics Library for Ray-tracing* と呼ばれ、Windows ドライバーと共有のライブラリであることが語られている。  
AMD GPU 向け Vulkan ドライバー *RADV* でも近い手段を採用しており、ソートをコンピュートシェーダで実装し、ドライバーのビルド時にコンパイルして組み込んでいる。  

 > 		GRL, or Graphics Library for Ray-tracing is a library we share with the
 > 		Windows drivers for doing BVH builds on the GPU.  It consists of a few
 > 		headers shared between CL and C code, a bunch of CL kernels, and some
 > 		GRL meta-kernels in their own format.
 >
 > {{< quote >}} [anv: Import GRL (fe208cb9) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/fe208cb9c385094d489c74cc380fe24d3e378b27) {{< /quote >}}

Intel は Architecture Day 2020 にて、LLVM ベースの *Intel Graphics Compiler (IGC)* を Windows ドライバーのバックエンドに採用し、内部では Mesa3D におけるオープンソースドライバーにも *IGC* を採用したプロトタイプをj実験的に開発していることを発表していた。[^arch-day-2020]  
プロトタイプに関する発表はその後行われていないが、今回の *GRL* 導入により、オープンソースドライバーと Windowsドライバーとの一部ドライバースタックの共通化は引き続き取り組まれていることがうかがえる。  

[^arch-day-2020]: [Intel Is Using IGC In Their Windows Drivers, Internal Prototype For Mesa - Phoronix](https://www.phoronix.com/news/Intel-IGC-Windows-Mesa-Info)

{{< ref >}}
 * [intel: add an OpenCL C compiler binary (!13171) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13171)
 * [intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler)
 * [iris: Enable clover support with an environment variable (!7047) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7047)
 * [Intel's Open-Source Vulkan Driver For Ray-Tracing Gets "Like A 100x Improvement" - Phoronix](https://www.phoronix.com/news/Intel-Vulkan-RT-100x-Improve)
 * [Intel Is Using IGC In Their Windows Drivers, Internal Prototype For Mesa - Phoronix](https://www.phoronix.com/news/Intel-IGC-Windows-Mesa-Info)
{{< /ref >}}
