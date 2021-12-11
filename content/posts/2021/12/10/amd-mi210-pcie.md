---
title: "GCD 1基で構成される Instinct MI210"
date: 2021-12-10T10:52:02+09:00
draft: false
tags: [ "Aldebaran", "gfx90a", "CDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

フィンランドの国営企業、CSC に主任HPCサイエンティストとして籍を置く [George Markomanolis](https://twitter.com/geomark) 氏によって、**Instinct MI210** のスペックが明かされた。  
**MI210** は GCD (Graphics Compute Die) 1基の構成を採り、CU 104基、HBM2e 64GB (4096-bit) というスペックとなる。  

 * <https://twitter.com/geomark/status/1466837043837915138>

[George Markomanolis](https://twitter.com/geomark) 氏 は、GPUパーテーションに *AMD Trento CPU + 4x MI250X* で構成されるノードを 2560ノード持つ **LUMI** スーパーコンピュータにおいて、CUDA で書かれたアプリケーションを AMD GPU でも動作する [HIP](https://github.com/ROCm-Developer-Tools/HIP) に移植する作業にも取り組んでいる。  

*CDNA 2 アーキテクチャ* を採用する **Instinct MI200 シリーズ** として 3つの SKU が発表されていたが、内スペックが明らかにされたのは **MI250/MI250X** の 2つ。もう 1つの **MI210** は PCIeカードで提供されることだけが明かされたのみで、その他スペックはこれまで不明だった。  
{{< link >}} [AMD、マルチダイ構成を採用した MI200シリーズ GPU を正式発表 | Coelacanth's Dream](/posts/2021/11/09/amd-mi200-series/) {{< /link >}}

**MI250/MI250X** は対称的な GCD 2基を *XGMI/Infinity Fabric Link* を用いて接続する構成であり、一部の用途に特化した複数種のチップを組み合わせる構成ではないため、GCD 1基のみで動作させることが可能となっている。  
Intel GPU では *{{< xe class="hp" >}}* が同様の構成であり、1/2/4 基のバリエーションが示されていた。  
ただ *{{< xe class="hp" >}}* は、oneAPI、Aurora スーパーコンピュータのソフトウェア開発用として Intel内の oneAPI devcloud のみに展開され、それとは別に製品化する予定は無いことが Raja Koduri 氏より語られている。[^xe-hp]  

[^xe-hp]: <https://twitter.com/Rajaontheedge/status/1453808598283210752>

[amd-cdna2-white-paper.pdf](https://www.amd.com/system/files/documents/amd-cdna2-white-paper.pdf) のブロックダイアグラムには GCD あたり 8-Links 持つが、4-Links は GCD-GCD間の接続に使われるため (400 GB/s)、GCD 2基をまとめたパッケージレベルでは最大 8-Links をサポートする形となる。  
**MI210** の有効 *XGMI/Infinity Fabric Link* 数は不明だが、フォームファクターが PCIeカードであるため、**MI100 (CDNA, Arcturus)** 同様に 3-Links となるのではないかと思う。  
PCIeカードの場合は *XGMI/Infinity Fabric Link* にブリッジを用いるため、ブリッジ部の性能による限界もある。  
GCD あたりが持つ XGMI用の SDMAエンジン (System DMA) 数は、*Arcturus* から減っているが、それでも 3基持っているため、**MI100** と同様に 4-GPU での相互接続が可能なはずである。  
{{< link >}} [Linux Kernel に 「Aldebaran」 GPU をサポートするパッチが投稿される　―― CDNA 2/MI200? | Coelacanth's Dream](/posts/2021/02/25/amd-aldebaran-gpu/#sdma) {{< /link >}}

{{< ref >}}
 * [Pre-Release Containers](https://www.intel.com/content/www/us/en/developer/articles/containers/pre-release-containers.html?wapkw=%22Xe-HP%22)
 * [Updated: Intel Cans Xe-HP Server GPU Products, Shifts Focus To Xe-HPC and Xe-HPG](https://www.anandtech.com/show/17041/intel-cans-xehp-gpu-products-shifts-focus-to-xehpc-and-xehpg)
 * [Preparing codes for LUMI: converting CUDA applications to HIP - LUMI](https://www.lumi-supercomputer.eu/preparing-codes-for-lumi-converting-cuda-applications-to-hip/)
 * [LUMI’s full system architecture revealed - LUMI](https://www.lumi-supercomputer.eu/lumis-full-system-architecture-revealed/)
{{< /ref >}}
