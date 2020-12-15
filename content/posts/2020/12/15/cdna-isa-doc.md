---
title: "AMD、CDNA/MI100 ISA Reference Guide を公開"
date: 2020-12-15T12:01:27+09:00
draft: false
tags: [ "CDNA", "Arcturus" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

2020/12/04 付で、**AMD Instinct MI100** (コードネーム [Arcturus](/tags/arcturus)) のISA (命令セット) ドキュメント、*"AMD Instinct MI100" Instruction Set Architecture Reference Guide* が公開された。  

 * [Developer Guides, Manuals & ISA Documents - AMD](https://developer.amd.com/resources/developer-guides-manuals/)
   * ["AMD Instinct MI100" Instruction Set Architecture: Reference Guide - CDNA1_Shader_ISA_14December2020.pdf](https://developer.amd.com/wp-content/resources/CDNA1_Shader_ISA_14December2020.pdf)


*MI100/Arcturus* はグラフィクス処理用の固定機能ユニットを持たず、コンピュート処理専用のハードウェアとなる。  
{{< link >}} [AMD、初の CDNAアーキテクチャ GPU 「MI100」 を発表 | Coelacanth's Dream](/posts/2020/11/17/amd-cdna-arch-mi100-arcturus/) {{< /link >}}
命令セットにおいても、ピクセルシェーダーを出力するための命令等は実行することができない。  

## MFMA, miSIMD, AccVGPR

*CDNAアーキテクチャ* では、通常の SIMDユニット、VGPR (Vector General-Purpose Registera, ベクタレジスタ) に加え、miSIMDユニット (Machine Intelligence SIMD) と AccVGPR (Accumulation VGPR) が増設された。これらはそれぞれ分けて配置され、データポートも非対称となる。  

miSIMD は主に 4-way のドット積を処理し、この時の入力値は全て AccVGPR から読み込まれる。  

*MI100/Arcturus* 発表時、同時に公開されたホワイトペーパーから、*CDNAアーキテクチャ* の SIMDユニットまたは CU あたりの VGPR容量は *GCNアーキテクチャ* から倍になったと書いたが、  
(GCN: SIMDユニットあたり 64KB、CUあたり 256KB, CDNA: SIMDユニットあたり 128KB、CUあたり 512KB)  
今回公開された SIMDユニットと VGPR との帯域、データポートを示したダイアグラムによると、通常の VGPR は *GCNアーキテクチャ* から変わらず 64KB となっており、AccVGPR 64KB と合わせて SIMDユニットあたり 128KB、CU あたり 512KB となる。  

{{< figure src="/image/2020/12/15/cdna-vgpr.webp" >}}

ダイアグラムを見る限り、AccVGPR は通常の SIMDユニットである shSIMD (shader SIMD) からは利用することができないものと思われる。そういった点で shSIMD と VGPR は *GCNアーキテクチャ* からほとんど変わりない。  
miSIMD からは VGPR の一部ポートを利用してデータを読み込めるが、AccVGPR への移動のためとされる。  
miSIMD と AccVGPR 間のアクセス幅は shSIMD と VGPR のそれよりも広いが、これは行列演算では入力値の多くを再利用するため、読み出し回数を減らして高速化と電力効率の向上を目的としたものと考えられる。  

miSIMD で処理した結果は直接 LDS (Local Data Share) やキャッシュ、メモリに出力することができず、通常の VGPR を介して出力する必要がある。  

shSIMD と VGPR、miSIMD と AccVGPR は非対称で役割は異なるが、それぞれ独立している訳ではなく、shSIMD と VGPR をメインに組み合わされた形となる。  
当初 [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) の *MI100/Arcturus* 全体ダイアグラムには、特別説明も無しに CU ではなく XCU と記載されていたが、  
miSIMD と AccVGPR を組み合わせた意味での XCU だったのかもしれない。  
現在では CU に修正されているため、それ以上の意味を推し量ることはできない。  

