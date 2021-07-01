---
title: "ダイあたり 4基のメモリコントローラーを持つ Aldebaran/MI200 GPU"
date: 2021-07-01T15:22:43+09:00
draft: false
tags: [ "Aldebaran", "MI200", "CDNA_2", "gfx90a" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

Linux Kernel における、ハードウェアのエラーを検出、収集して報告する EDAC (Error Detection And Correction) ドライバーに *Aldebaran/MI200* GPU のサポートを追加するパッチが投稿された。  
*CDNA 2 アーキテクチャ* を採用する *Aldebaran* GPU は *3rd Gen Infinity Architecture* 、CPU との *XGMI/Infinity Fabric* 接続に対応し、*3rd Gen Infinity Architecture* では CPU と GPU のメモリ空間が統合される。  

 * [[PATCH 0/7] x86/edac/amd64: Add support for noncpu nodes](https://lore.kernel.org/linux-edac/20210630152828.162659-1-nchatrad@amd.com/T/#t)

パッチの主目的としては EDACドライバーに GPUノード (noncpu node) のサポートを追加することであり、CPU+GPU でノードを構成するのではなく、CPUノードと GPUノードを *XGMI/Infinity Fabric* で接続するシステム構成に *Aldebaran* GPU で新たに対応したのではないかと思われる。  

 > 		+/*
 > 		+ * On Newer heterogeneous systems from AMD with CPU and GPU nodes connected
 > 		+ * via xGMI links, the NON CPU Nodes are enumerated from index 8
 > 		+ */
 > 		+#define NONCPU_NODE_INDEX	8
 > 		+
 >
 > {{< quote >}} [[PATCH 0/7] x86/edac/amd64: Add support for noncpu nodes](https://lore.kernel.org/linux-edac/20210630152828.162659-1-nchatrad@amd.com/T/) {{< /quote >}}

先日 Intel は ISC2021 にて [Sapphire Rapids CPU](/tags/sapphire_rapids) と [{{< xe class="hpc" >}} GPU](/tags/xe-hpc) に関する新たな発表を行い、*Intel Ponte Vecchio* GPU が OCP Accelerator Module (OAM) フォームファクと、それを 4基組み合わせて構成するサブシステムに対応することを明らかにしたが、 *AMD Aldebaran* の GPUノード (noncpu node) も同様の構成を採るのかもしれない。[^intel-isc2021]
{{< link >}} [47個のタイルで構成される Intel Ponte Vecchio | Coelacanth's Dream](/posts/2021/03/26/intel-pvc-47-tiles/) {{< /link >}}

[^intel-isc2021]: [New Intel XPU Innovations Target HPC and AI](https://www.intel.com/content/www/us/en/newsroom/news/new-intel-xpu-innovations-target-hpc-ai.html)

## ダイあたり 4基の UMC {#umc}

パッチのコメント部にて *Aldebaran* メモリコントローラーについて触れられている。  
*Aldebaran* は 2個のダイによる MCM (Multi-chip Module)構成を採るが、ダイはそれぞれ 4基の Unified Memory Controller (UMC) を持ち、UMC は 8チャネルを処理可能、そしてメモリチャネルにはそれぞれ 2GB の HBM2(e)メモリが接続されるという。HBM系メモリでは 1024-bit のインターフェイスを持ち、128-bit のメモリチャネル 8本を束ねる形で構成される。  
UMC に接続される HBM2メモリスタックあたりの容量は 16GB となり、ダイあたりでは 64GB、*Aldebaran* GPU 全体では 128GB の GPUメモリが搭載されると考えられる。  
HBM2メモリではスタックあたり最大 8GB までだったが、HBM2e では 16GB の構成が可能になっている。  


 > 		Aldebaran has 2 Dies (enumerated as a MCx, x= 8 ~ 15) 
 > 		  Each Die has 4 UMCs (enumerated as csrowx, x=0~3)
 > 		  Each die has 2 root ports, with 4 misc port for each root.
 > 		  Each UMC manages 8 UMC channels each connected to 2GB of HBM memory.
 >
 > {{< quote >}} [[PATCH 0/7] x86/edac/amd64: Add support for noncpu nodes](https://lore.kernel.org/linux-edac/20210630152828.162659-1-nchatrad@amd.com/T/#m556d6870ba567a24c2677b6e5d81739fcd12f312) {{< /quote >}}

TSMC 7nmプロセス (N7) で製造される *NVIDIA GA100* はチップ的には 6144-bit (6x HBM2/eスタック)のメモリインターフェイスが実装されていたが、製品である **NVIDIA A100 GPU** では 1スタック分を無効化し、5120-bit (5x HBM2/eスタック) の構成となっていた。 *NVIDIA GA100* は 826 mm2 という巨大なダイサイズであり、メモリインターフェイスを一部無効化したのは歩留まり向上のためと考えられる。  
対し、*AMD Aldebaran* GPU ではダイあたりのメモリインターフェイスは 4096-bit (4x HBM2eスタック) になると見られ、ダイレベルでは [CDNA アーキテクチャ](/tags/cdna) を採用する *Arcturus/MI100* と同じ規模のメモリインターフェイスを保つことができ、歩留まり悪化の緩和が期待できる。こうした点は MCM構成の明らかな利点だろう。  
また、 *Aldebaran* GPU は 2個のダイで構成され、それぞれプライマリーダイとセカンダリーダイという役割の異なるダイに分かれるが、そこは設定可能な部分で、ダイ自体は同一になると思われる。  
{{< link >}} [プライマリーダイとセカンダリーダイで構成される Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/06/09/aldebaran-primary-secondary/) {{< /link >}}

{{< ref >}}
 * [8GB/16GB HBM2E with ECC - 8gb_and_16gb_hbm2e_dram.pdf](https://media-www.micron.com/-/media/client/global/documents/products/data-sheet/dram/hbm2e/8gb_and_16gb_hbm2e_dram.pdf?rev=dbfcf653271041a497e5f1bef1a169ca)
 * [nvidia-ampere-architecture-whitepaper.pdf](https://images.nvidia.com/aem-dam/en-zz/Solutions/data-center/nvidia-ampere-architecture-whitepaper.pdf)
 * [西川善司の3DGE：NVIDIAが投入する20 TFLOPS級の新GPU「A100」とはいったいどのようなGPUなのか？](https://www.4gamer.net/games/121/G012181/20200527061/)
{{< /ref >}}
