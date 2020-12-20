---
title: "Intel Xe GPU 近況　――  Xe-HPのメモリリージョン, PXP, Iris Xe MAX のサポート状況"
date: 2020-12-21T04:09:59+09:00
draft: false
tags: [ "XeHP", "DG1", "SG1", "Gen12" ]
# keywords: [ "", ]
categories: [ "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

## {{< xe class="hp" >}} のメモリリージョン

2020/11/27、Linux Kernel (intel-gfx) に *DG1* の持つ VRAM、LMEM (Local Memory) を有効にするパッチが投稿された。[^dg1-lmem]  
パッチには *{{< xe class="hp" >}}* に向けた準備も含まれており、*{{< xe class="hp" >}}* のメモリリージョンについて言及された部分があった。  

*DG1* は 1つの GPUチップに LPDDR4x 128-bit という、GPU としては一般的な構成を取る。  
データセンター向け PCIeカード *H3C XG310 GPU* は、サーバー向け dGPU **SG1** を 4チップ搭載するが、GPU間は PLXチップで接続されていると思われ、GPU はそれぞれ別のデバイスとして認識される。メモリ空間も別々となる。[^sg1-xg310]  

*{{< xe class="hp" >}}* はチップをタイルとして 1つのパッケージに複数搭載され、タイル間は EMIB技術を用いて接続される。  
**H3C XG310 GPU** と異なるのは、*{{< xe class="hp" >}}* はマルチコアGPUであるように動作し、単一のメモリ空間となる (だろう) 点。  
メモリ空間を共有し、EMIB技術によって GPU間が広帯域で接続されても、やはりメモリアクセスに掛かる時間としてはその GPU に直接接続されているメモリが速い。  
そのため、マルチコアGPUとしてもメモリに異なる層、リージョンが存在する。  
パッチではそれを区別するため、LMEM とは別に SHMEM のバッファーオブジェクト (BO, GPU メモリ管理の単位) のサポート準備が含まれている。  

他社の GPU間接続には、NVIDIA は NVLink、AMD は Infinity Fabric (xGMI) といった名前が付いているが、Intel のそれにはまだマーケティング的な名前は見受けられない。  

[^dg1-lmem]: [[Intel-gfx] [RFC PATCH 000/162] DG1 + LMEM enabling](https://lists.freedesktop.org/archives/intel-gfx/2020-November/254003.html)
[^sg1-xg310]: [Intel、サーバー向け GPU 「SG1」 と搭載カードを正式発表 | Coelacanth's Dream](/posts/2020/11/12/intel-sg1/)

## Gen12 GPU は PXP をサポート

2020/11/14 に、PXP (Protected Xe Path) をサポートするパッチが投稿された。[^pxp]  
PXP はユーザースペースで実行可能な ring3 レベルに、ハードウェアで保護されたセッションを確立させ、各セッションの状態とライフサイクルを管理するのに役立つ機能とされている。  

[^pxp]: [[Intel-gfx] [PXP CLEAN PATCH v06 01/27] drm/i915/pxp: Introduce Intel PXP component](https://lists.freedesktop.org/archives/intel-gfx/2020-November/252790.html)

ユーザースペース側では、**Iris (OpenGL)** 、**ANV (Vulkan)** ドライバーに PXP をサポートするパッチが投稿されている。現在はマージリクエストの段階。[^pxp_umd]  

[^pxp_umd]: [WIP: iris: protected content support (!8092) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/8064/commits) [anv: protected memory support (!8064) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/8064/commits)

似た機能として AMD TMZ (Trust Memory Zone) があり、TMZ ではメモリを暗号化し、GPU内の信頼できるハードウェアブロック部のみで復号化を可能にすることでコンテンツを保護する。 TMZ は *Vega/GFX9* 世代からサポートされており、EPYC の SME (Secure Memory Encryption(、SEV (Secure Encrypted Virtualization) といい、メモリの暗号化機能に関しては AMD のが早くから意識していたような印象を受ける。  
{{< link >}} [【XDC2020】AMD GPU のメモリ暗号化機能 TMZ　―― DRMへの活用も | Coelacanth's Dream](/posts/2020/09/21/xdc2020-amdgpu-tmz/) {{< /link >}}

PXP も TMZ 同様に DRM への活用を想定した機能ではないかと思われる。  

## Linux環境における Iris Xe MAX のサポート状況

ディスクリートGPUと言うと、マザーボードの PCIeスロット差し、ケーブルでモニタと繋げば映像が出力され、適切なドライバーをインストールすればそのグラフィクス処理を発揮できる、とシンプルに思っているが、**DG1 / Iris Xe MAX** はまだそういった段階にはないらしい。  

Intel が公開しているドキュメントによると、*{{< xe class="lp" >}}/Gen12アーキテクチャ* のサポートは既に Linux に為されているが、それは *Tiger Lake* 等に統合されている GPU までで、dGPU である **Iris Xe MAX** のサポート作業はまだ進行中だとされている。  

 * [GPGPU: Intel Iris Xe MAX graphics](https://dgpu-docs.intel.com/devices/iris-xe-max-graphics/index.html)
 
現在では早期アクセスとして、仮想環境を構築、異なる Linux Kernel を 2つインストールし、**Iris Xe MAX** はゲストVM で動作させる手段を提供している。  
ディスプレイ出力は *Tiger Lake* の iGPU側から行ない、**Iris Xe MAX** でグラフィクス処理、GPGPU、メディアエンコードを実行できるが、やはりユーザーが一般的に見て、求めるシステムとは離れている気がする。  

**Iris Xe MAX** はゲストVMに PCIパススルーを用いてアクセスする。  
データセンター向け製品である **H3C XG310 GPU** がリリースされ、2021年には *{{< xe class="hp" >}}* GPU が登場するため、PCIパススルー機能が既に Intel dGPU でサポートされているのは歓迎されることではある。また、近頃 Intel にとってサーバー市場はますます重要なものとなっており、そのために仮想化機能のサポートに力を入れている可能性もある。  

ただ 2021年前半には OEM向けにデスクトップ向け *DG1* がリリースされ、同年中にはHWレイトレーシングもサポートしたゲーミング向け dGPU *{{< xe class="hpg" >}}* がリリース予定にある中で、[^intel-dgpu-2021]  
それらの先立ちである **Iris Xe MAX** の、Linux環境におけるサポートがまだ十分な状態にないことに不安を覚えるのは確かだ。  

[^iris-xe-max]: [Intel、モバイル向け dGPU "Iris Xe Max" を正式発表 | Coelacanth's Dream](/posts/2020/11/01/intel-iris-xe-max/)
[^intel-dgpu-2021]: [Intel、モバイル向け dGPU "Iris Xe Max" を正式発表 | Coelacanth's Dream](/posts/2020/11/01/intel-iris-xe-max/#next-gpu)

{{< ref >}}

 * [Intel Xe MAX Needs Two Linux Kernels For Now - Meaning You Need To Use A GPU-Accelerated VM - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Intel-Xe-MAX-dGPU-VM)

{{< /ref >}}


