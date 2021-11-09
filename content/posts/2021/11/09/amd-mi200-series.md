---
title: "AMD、マルチダイ構成を採用した MI200シリーズ GPU を正式発表"
date: 2021-11-09T08:47:49+09:00
draft: false
tags: [ "CDNA_2", "Aldebaran" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD は 2021/11/08 付で、*CDNA 2 アーキテクチャ* を採用、HBM2e 128GB を搭載し、初のマルチダイ構成を採用した **MI200 シリーズ** を正式発表した。  
**MI200 シリーズ** として CU 208基、*XGMI/Infinity Fabric Link* 6本の **Instinct MI250** と、CU 220基、*XGMI/Infinity Fabric Link* 8本の **Instinct MI250X** の 2種類が用意されている。  
フォームファクターはどちらも OAM (OCP Accelerator Module) で提供され、PCIeカードに収めた **Instinct MI210** も用意されているとするが、**MI210** の性能、規模は不明。  

 * [New AMD Instinct™ MI200 Series Accelerators Bring Leadership HPC and AI Performance to Power Exascale Systems and More :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1032/new-amd-instinct-mi200-series-accelerators-bring)
 * [AMD Instinct™ MI250 Accelerator | AMD](https://www.amd.com/en/products/server-accelerators/instinct-mi250#product-specs)
 * [AMD Instinct™ MI250X Accelerator | AMD](https://www.amd.com/en/products/server-accelerators/instinct-mi250x#product-specs)

*CDNA 2 アーキテクチャ* は FP64, FP32 のデータフォーマットに対応した第2世代 Matric Core を持ち、CU あたりの FP64演算性能は前世代の *CDNA アーキテクチャ* から倍になっている。  

 * [AMD CDNA™ 2 Architecture | AMD](https://www.amd.com/en/technologies/cdna2)
    * [amd-cdna2-white-paper.pdf](https://www.amd.com/system/files/documents/amd-cdna2-white-paper.pdf)
    * [amd-instinct-mi200-datasheet.pdf](https://www.amd.com/system/files/documents/amd-instinct-mi200-datasheet.pdf)

各ダイは *GCD (Graphics Compute Die)* と呼ばれ、TSMC 6nm FinFet プロセスで製造される。  

[amd-cdna2-white-paper.pdf](https://www.amd.com/system/files/documents/amd-cdna2-white-paper.pdf) が公開されているが、前世代 *CDNA アーキテクチャ* の [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) より情報量が少なく、 *CDNA 2* CU とそこに搭載された SIMDユニット 4基の詳細は載っていない。  

オープンソース・ドライバーや ROCmソフトウェア等で公開された情報によれば、*CDNA アーキテクチャ* ではSIMDユニットごとに持つベクタレジスタと Matric Core 専用のレジスタファイルは別々になっていたが、*CDNA 2 アーキテクチャ* では統合され、処理によってより多くのレジスタを割り当てることが可能になった。  
{{< link  >}} [CDNA 2 アーキテクチャでは Unified Register Files を実装 | Coelacanth's Dream](/posts/2021/07/01/aldebaran-unified-vgpr/) {{< /link >}}
FullRateFP64 への対応が行われ、CU ごとに持つ LDS (Local Data Share) も従来のアーキテクチャでは 64 KB (32 [banks] x 512 [entries] x 4 [bytes per entries]) だったのが、エントリあたり 8 bytes (64-bit) になったと思われるが、*CDNA 2 アーキテクチャ* の LDSサイズは明かされていない。  

L2キャッシュは GCD あたり 8MB と、*Arcturus/MI100* と同サイズだが、キャッシュラインサイズは 64 Bytes から 128 Bytes に変更され、L2キャッシュの帯域は増加した。  


## 情報整理 {#sort}

自分が書いたことだから自分が一番覚えている訳だが、自分はこれまで *Aldebaran/MI200 シリーズ* を CU 110基、GCD あたり CU 55基と考えていたが、実際は GCD あたり CU 110基という構成だった。  
CU 110基というのは ROCmソフトウェアの記述から明らかにされたもので、GCD あたり CU 55基と考えたのは、*Aldebaran* がマルチダイ・MCM 構成を採用し、2つのダイで 1つの GPU と認識されるような密接な GPU 間接続技術を使っていると考えていたからだった。  
{{< link >}} [やっぱり Aldebaran/MI200 はトータル 110CU ? | Coelacanth's Dream](/posts/2021/09/17/re-gfx90a-110cu/) {{< /link >}}

だが今回明らかにされた詳細から、*Aldebaran/MI200 シリーズ* は各 GCD が 1つの GPU と認識され、パッケージレベルでは 2つの GPU となると思われる。  

**MI200 シリーズ** では *XGMI/Infinity Fabric Link* により、GCD間は 400GB/s、パッケージ間では各 GCD が他パッケージの GCD に一対一で接続されるため 200GB/s となる。  
若干否定的な言い方になってしまうが、*Aldebaran/MI200 シリーズ* において同パッケージに搭載された 2基の GCD はあくまでも他パッケージ・カードよりも高速、広帯域に接続された別々の GPU だと考えられる。  
Intel が *{{< xe class="hp" >}}* で目指す、複数のタイル・ダイで構成されながらも、ユーザースペース・UMD (User Mode Driver) からは単一の GPU と認識されるような、密接な統合を行うマルチダイ・MCM 構成は **MI200 シリーズ** ではまだ実現されていないと見られる。  
{{< link >}} [Intel のマルチタイル GPU をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/10/10/intel-xe_hp-multi-tile/) {{< /link >}}
Intel *{{< xe class="hp" >}}* では 2タイルか 4タイルの複数タイル構成が可能となっている。また、Intel *{{< xe class="hpc">}}* 、**Ponte Vecchio** では 2タイルで 1つの *{{< xe class="hpc">}} Slice* を構成し、1つの Hardware Context を 2タイルで実行するという、マルチタイル構成をさらに推し進めたものとなっている。  

将来的に単一の GPU として認識されるよう、一部機能、技術が *Aldebaran* に実装されている *かもしれないが* 、製品構成を見る限り有効化はされていないだろう。  
しかし、言い換えれば、いきなり同パッケージに搭載した複数の GPU を 1つの GPU として見せ掛けるようにするのではなく、まずは 1つのパッケージに 2基の GPU を搭載して、それぞれを高速、広帯域に接続、パッケージあたりの演算性能を大きく高めた製品のリリースを優先した、慎重な姿勢とも取れる。  
**Instinct MI250X** では CU数の倍化と FullRateFP64 の対応により、**MI100** 比で 4倍の FP64演算性能となっている。  
*XGMI/Infinity Fabric Link* によるメモリコヒーレントを取ったマルチGPU構成は、*Vega20* (**Instinct MI50/MI60**) から対応が行われているため、ソフトウェア側の対応に必要な変更も少なめで済む。  
ただ、帯域の異なるトポロジが存在する、従来よりも GPU数の増えた (8 -\> 16,32) マルチGPU構成であるため、ネットワーク構成は複雑なものになると思われる。  

[amd-cdna2-white-paper.pdf](https://www.amd.com/system/files/documents/amd-cdna2-white-paper.pdf) では 3種類のノード構成が紹介されており、CPU+GPU の統合メモリを有効化した、EPYC CPU 1基、**MI250X** 4基、そして各 **MI250X** を 200Gbps NIC に接続した構成を Flagship HPC Topology、  
CPU-GPU間を PCIe で接続し、GPU間の *XGMI/Infinity Fabric Link* を有効化した CPU 2基、**MI250** 4基の構成 Mainstream HPC/ML Topology、  
CPU-GPU間、GPU間は Mainstream HPC/ML Topology と同じだが、**MI250** を 8基に増やし、PCIe Switch 経由で複数の NIC/Storage を接続した構成を Flagship ML Topology としている。  
