---
title: "OLCF Frontier と同様のノード構成となる Crusher ―― EPYC 7A53 + 4x MI250X"
date: 2022-01-14T22:50:55+09:00
draft: false
tags: [ "HPC", "Aldebaran", "gfx90a", "CDNA_2", "ROCm" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU", "GPU" ]
noindex: false
# summary: ""
---

Oak Ridge Leadership Computing Facility (OLCF) の HPCシステムの 1つ、**Crusher** の Quick-Start Guide が公開された。  

 * [Crusher Quick-Start Guide — OLCF User Documentation](https://docs.olcf.ornl.gov/systems/crusher_quick_start_guide.html)

## Crusher {#crusher}

**Crusher** のノードは、**1x AMD EPYC 7A53 (Zen 3) + 4x AMD MI250X (CDNA 2, Aldebaran)** 、メモリは CPU側に DDR4 512GB (3200 MT/s) が搭載されている。**AMD MI250X** GPU はそれぞれ GCD (Graphics Compute Die) 2基で構成され、GCD ごとに HBM2E 64GB が搭載されている。  
**AMD EPYC 7A53** は CPU-GPU とメモリコヒーレントを取る XGMI (Infinity Fabric) 接続をサポートしており、*Optimized 3rd Gen EPYC*、あるいはコードネーム *Trento* とも呼ばれる。  
*Optimized 3rd Gen EPYC* としてこれまでにもドキュメント中に出てきたが、具体的な SKU名が明かされたのはこれが初めてな気がする。  
**AMD EPYC 7A53** は 64-Core/128-Thread を持ち、この点は *EPYC Milan (Zen 3)* と同じ。CPUキャッシュ構成は明かされていないが、こちらも *EPYC Milan* と同様、というより同じ CCD を用いているのではないかと思われる。  

エクサスケールスパコン **Frontier** のノードも同様のハードウェアで構成され、**Crusher** は **Frontier** のテストベッド (実証基盤) として使用される。ソフトウェア環境も基本 **Frontier** と同じだとしている。  
**Crusher** システムは 128ノードと 64ノードのキャビネット 2基で構成され、全体では 196ノードとなる。  
同じく **Frontier** に向けたテストベッドには **Spock** システムも存在する。ただし、**Spock** はノードが **AMD EPYC 7662 (Zen 2) + 4x AMD MI100 (CDNA, Arcturus)** で構成されている。**Frontier** と近いハードウェア構成ではあるが、同じではない。[^spock]  

[^spock]: [Spock Quick-Start Guide — OLCF User Documentation](https://docs.olcf.ornl.gov/systems/spock_quick_start_guide.html#system-overview)

**AMD EPYC 7A53** は内部的に 4個の NUMAトポロジに分割され、NUMAトポロジあたり CCD 2基、L3キャッシュリージョンを 2個持つ構成を採っている。  
**AMD MI250X** はパッケージあたり GCD 2基で構成され、それらは 200+200 GB/s という広帯域で接続されている。ただ、あくまでも広帯域で接続された GPU 2基 として扱われる。  
そのためプログラムからはノードあたり GPU 8基 が見える形となっている。  

**Crusher** ノードでは、L3キャッシュリージョン 1個に GPU (GCD) 1基を関連付ける形式を採っている。  
これは **Crusher** の特性だとしており、プロセス間のメッセージ機能、Open MPI (Message Passing Interface) ライブラリを使う上で性能に影響する。  
MPIランク (プロセスIDに相当する) は各 GPU にマッピングされるが、この時その GPU に関連付けられた L3キャッシュリージョンとは異なる CPUスレッドが割り当てられた場合、性能が低下する恐れがある。  
そのため、マッピングによる性能への影響を考慮することが重要だとしている。  

## AMDGPUドライバ側の対応 {#amdgpu-driver}

Linux Kernel における AMDGPUドライバでは、CPU-GPU のメモリコヒーレントを取るシステムへの対応を進めている。  

 * [[PATCH v3 00/10] Add MEMORY_DEVICE_COHERENT for coherent device memory mapping](https://lists.freedesktop.org/archives/amd-gfx/2022-January/073357.html)

CPU と GPU すべてのメモリを単一のアドレススペースにマッピングには、BIOS/UEFI のレベルからそれを意識する必要がある。以下のパッチを投稿した AMD のソフトウェアエンジニアである Alex Sierra 氏は、GPUメモリを BIOS/UEFI では SPM (Special Purpose Memory) としてマッピングし、Linux Kernel はその情報を元にメモリのリマッピングを行う。  
以上とハードウェアのページ移行機能を採用する最初のユーザとして、**Frontier** が想定されているが、Alex Sierra 氏は AMD CPU + GPU 固有の機能ではないとし、将来的に他の GPUベンダにも活用されることを期待していると語っている。  
{{< link >}} [Linux Kernel に CPU + GPU の統合メモリ空間をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/07/add-svm-to-amdgpu-kfd/) {{< /link >}}


{{< ref >}}
 * [Hello World: MPIプログラム入門](https://www.gsic.titech.ac.jp/supercon/supercon2004/jp/mpi/hello.htm)
 * [AMD CDNA™ 2 Architecture | AMD](https://www.amd.com/en/technologies/cdna2)
{{< /ref >}}
