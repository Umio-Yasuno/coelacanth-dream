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

Oak Ridge Leadership Computing Facility (OLCF) の HPCシステムの 1つ、*Crusher* の Quick-Start Guide が公開された。  

 * [Crusher Quick-Start Guide — OLCF User Documentation](https://docs.olcf.ornl.gov/systems/crusher_quick_start_guide.html)
 * [olcf-user-docs/crusher_quick_start_guide.rst at 0dddf16f0229da8afe314860fd4fb979995582a3 · olcf/olcf-user-docs](https://github.com/olcf/olcf-user-docs/blob/0dddf16f0229da8afe314860fd4fb979995582a3/systems/crusher_quick_start_guide.rst)

## Crusher {#crusher}

*Crusher* のノードは、*1x AMD EPYC 7A53 (Zen 3) + 4x AMD MI250X (CDNA 2, Aldebaran)* 、メモリは CPU側に DDR4 512GB (3200 MT/s, total 205 GB/s) が搭載されている。  
*AMD MI250X* GPU はそれぞれ GCD (Graphics Compute Die) 2基で構成され、GCD ごとに HBM2E 64GB (1.6 TB/s) を持つ。  

CPU-GPU はメモリコヒーレントを取る XGMI (Global Memory Interconnect, Infinity Fabric) で接続され、*AMD EPYC 7A53* は通常の *EPYC* プロセッサと異なり、XGMI接続をサポートしている。  
*AMD EPYC 7A53* は *Optimized 3rd Gen EPYC*、あるいはコードネーム *Trento* とも呼ばれ、これまでにもそうした呼び名でドキュメント中に出てきたが、具体的な SKU名が明かされたのはこれが初めてな気がする。64-Core/128-Thread を持ち、この点は *EPYC Milan (Zen 3)* と同じ。  
CPUキャッシュ構成は明かされていないが、こちらも *EPYC Milan* と同様、というより同じ CCD を用いているのではないかと思われる。  
*AMD EPYC 7A53* は内部的に 4個の NUMAトポロジに分割され、NUMAトポロジあたり CCD 2基、L3キャッシュリージョンを 2個持つ構成を採っている。  

*AMD MI250X* はパッケージあたり GCD 2基で構成され、それらは 200+200 GB/s という広帯域で接続されている。ただ、あくまでも広帯域で接続された GPU 2基 として扱われる。  
そのためプログラムからはノードあたり GPU 8基 が見える形となっている。  
異なる *AMD MI250X* パッケージ間も XGMI で接続されているが、リンク数が少ないため GCD間より帯域は狭く、50+50 GB/s となっている。  
CPU-GPU間は 36+36 GB/s、帯域は GCDあたりとなり、また PCIe Gen4 x16 (32+32 GB/s) より帯域が広いが、このあたりが *Optimized 3rd Gen EPYC* とされる部分なのだろうか。  
*MI250X* はそれぞれ CPU 以外に、NIC とも PCIe Gen4 ESM (Extend Speed Mode) で接続されている。これは GCD に PCIe Root Complex が実装されたことで可能となった。  
このことを AMD CDNA 2 Whitepaper では、CPU と GPU のメモリ空間を統合したエクサスケールコンピューティングにおいて、非常に重要なものだとしている。  
また、*Crusher, Frontier* のノード構成は AMD CDNA 2 Whitepaper にて、Flagship HPC Topology として掲載されている。  

 * [amd-cdna2-white-paper.pdf](https://www.amd.com/system/files/documents/amd-cdna2-white-paper.pdf)

*Crusher* ノードでは、L3キャッシュリージョン 1個に GPU (GCD) 1基を関連付ける形式を採っている。  
これは *Crusher* の特性だとしており、プロセス間のメッセージ機能、Open MPI (Message Passing Interface) ライブラリを使う上で性能に影響する。  
MPIランク (プロセスIDに相当する) は各 GPU にマッピングされるが、この時その GPU に関連付けられた L3キャッシュリージョンとは異なる CPUスレッドが割り当てられた場合、性能が低下する恐れがある。  
そのため、マッピングによる性能への影響を考慮することが重要だとしている。  

エクサスケールスパコン *Frontier* のノードも *Crusher* と同様のハードウェアで構成され、*Crusher* は *Frontier* のテストベッド (実証基盤) として使用される。ソフトウェア環境も基本 *Frontier* と同じだとしている。  
*Crusher* システムは 128ノードと 64ノードのキャビネット 2基で構成され、全体では 196ノードとなる。  
*Frontier* のノード数は公開されていないが、キャビネット数は 100 を超えるとされている。[^frontier]  
同じく *Frontier* に向けたテストベッドには *Spock* システムも存在する。ただし、*Spock* はノードが *AMD EPYC 7662 (Zen 2) + 4x AMD MI100 (CDNA, Arcturus)* で構成されている。*Frontier* と近いハードウェア構成ではあるが、CPU、GPU ともに一世代前であり、同じではない。[^spock]  

[^frontier]: [Frontier](https://www.olcf.ornl.gov/frontier/)
[^spock]: [Spock Quick-Start Guide — OLCF User Documentation](https://docs.olcf.ornl.gov/systems/spock_quick_start_guide.html#system-overview)

## AMDGPUドライバ側の対応 {#amdgpu-driver}

Linux Kernel における AMDGPUドライバでは、CPU-GPU のメモリコヒーレントを取るシステムへの対応を進めている。  

 * [[PATCH v3 00/10] Add MEMORY_DEVICE_COHERENT for coherent device memory mapping](https://lists.freedesktop.org/archives/amd-gfx/2022-January/073357.html)

CPU と GPU すべてのメモリを単一のアドレススペースにマッピングには、BIOS/UEFI のレベルからそれを意識する必要がある。以下のパッチを投稿した AMD のソフトウェアエンジニアである Alex Sierra 氏は、GPUメモリを BIOS/UEFI では SPM (Special Purpose Memory) としてマッピングし、Linux Kernel はその情報を元にメモリのリマッピングを行う。  
以上とハードウェアのページ移行機能を採用する最初のユーザとして、*Frontier* が想定されているが、Alex Sierra 氏は AMD CPU + GPU 固有の機能ではないとし、将来的に他の GPUベンダにも活用されることを期待していると語っている。  
{{< link >}} [Linux Kernel に CPU + GPU の統合メモリ空間をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/07/add-svm-to-amdgpu-kfd/) {{< /link >}}


{{< ref >}}
 * [Hello World: MPIプログラム入門](https://www.gsic.titech.ac.jp/supercon/supercon2004/jp/mpi/hello.htm)
 * [AMD CDNA™ 2 Architecture | AMD](https://www.amd.com/en/technologies/cdna2)
 * [amd-cdna2-white-paper.pdf](https://www.amd.com/system/files/documents/amd-cdna2-white-paper.pdf)
 * [Frontier](https://www.olcf.ornl.gov/frontier/)
{{< /ref >}}
