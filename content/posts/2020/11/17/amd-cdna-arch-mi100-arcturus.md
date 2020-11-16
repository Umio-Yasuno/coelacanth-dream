---
title: "AMD、初の CDNAアーキテクチャ GPU 「MI100」 を発表"
date: 2020-11-17T00:20:47+09:00
draft: false
tags: [ "CDNA", "Arcturus" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

SC20 の開催に合わせ、初の *CDNAアーキテクチャ* 採用製品となる **AMD Instinct™ MI100** 、開発コードネーム *Arcturus* を発表した。  

 * [AMD EPYC™ Processors and New AMD Instinct™ MI100 Accelerator Redefine Performance for HPC and Scientific Research :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/980/amd-epyc-processors-and-new-amd-instinct-mi100)
 * [AMD CDNA Architecture | AMD](https://www.amd.com/en/technologies/cdna)
   * [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf)
 * [AMD Instinct™ MI100 Accelerator | AMD](https://www.amd.com/en/products/server-accelerators/instinct-mi100)

Linux Kernel (amd-gfx) には丁度 *Arcturus* の実験的サポートのフラグを外すパッチが投稿されている。[^arcturus-stable]  

{{< pindex >}}

 * [MI100 スペック](#mi100-spec)
 * [CDNAアーキテクチャ](#cdna)
   * [各アーキテクチャの CU 比較](#cu)
      * [RDNAアーキテクチャとの SIMDユニット比較](#simd)

{{< /pindex >}}

[^arcturus-stable]: [[PATCH] drm/amdgpu: remove experimental flag from arcturus](https://lists.freedesktop.org/archives/amd-gfx/2020-November/056158.html)

## MI100 スペック {#mi100-spec}

*Arcturus* は ShaderEngine 8基、最大総 CU 数 128基という構成だが、**MI100** では各 ShaderEngine 内の CU を 1基無効化し、総 CU 数 120基という仕様になっている。  

動作クロックは最大 1502MHz。ピークFP64演算性能は 11.5TFLOPs に達し、10TFLOPs の壁を突破した初の GPU を謳う。  

*Vega20/MI50* では、3基以上を *Infinity Fabric/XGMI* を用いて接続する場合、リングバス構成となり、GPU 間でのデータ転送に幾許かの遅延が発生していた。[^if-xgmi]  
だが *Arcturus/MI100* では最大 4基での相互接続に対応し、より高性能な GPU トポロジを構成可能となる。  
*Arcturus* は {{< comple >}}ネットワーク構造こそ変わるが {{< /comple >}}最大 8基のトポロジ構成に対応しているはずだが、**MI100** では *Infinity Fabric Link* が 3本となっている。[^exa]  

[^exa]: [AMD Financial Analyst Day 2020 個人的まとめ | Coelacanth's Dream](/posts/2020/03/06/amd-financial-analyst-day-2020/#3rd-gen-infinity-architecturue)
[^if-xgmi]: [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/)

メモリバス幅は HBM2 4スタック (4096-bit)、容量は最大 32GB、ECCサポート、という点は *Vega20/MI50* と変わらないが、メモリクロックは 0.2GHz 向上した 1.2GHz となり、ピークメモリ帯域は 1228.8GB/s となる。  
また、HBM2メモリのメモリ階層 1つ上、L2キャッシュは *Vega20/MI50* から倍の数に増やされ 32スライスとなり、L2キャッシュ容量は計 8MB。  

製造プロセスは *Vega20/MI50* と同じ TSMC 7nm FinFet とされ、大きく性能を向上させながらも TBP 300W を維持している。  

## CDNAアーキテクチャ {#cdna}

*Arcturus/MI100* はグラフィクス処理を行なうための固定機能ユニット {{< comple >}} ジオメトリプロセッサ、ラスタライザ、RB (Render Backend)、ディスプレイエンジンさえも {{< /comple >}} を持たず、空いた分を利用して、CU の改良、強化、実装密度を引き上げを実現している。  
ただ、{{< comple >}} GPU とそのアーキテクチャを活用したアクセラレータの境目は曖昧で、うまい言い方が思いつかないが {{< /comple>}} 一般的な GPU への転用はできないものとなっている。  
AMD も *Arcturus/MI100* を GPU と呼びつつも、アクセラレータとしての見方が強いのか、ブランドを **Radeon Instinct** ではなく **AMD Instinct** としており、**Radeon** の名は使っていない。ニュースリリース内を *Radeon* で検索しても見つからなかった。  

グラフィクス用の固定機能ユニットは持たないが、マルチメディアエンジンは搭載しており、用途としては機械学習のオブジェクト検出の際などを想定している。  
また、Linux Kernel (amd-gfx) に記述されているコードによると、VCN (Video Core Next) のバージョンは 2.5 となり、それを 2基搭載している。[^arcturus-vcn2_5]  

[^arcturus-vcn2_5]: [drm/amdgpu: enable VCN0 and VCN1 sriov instances support for Arcturus](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=ab5999dea06945f1f7b00bc20c23ab045afa74af)


### 各アーキテクチャの CU 比較 {#cu}

<!--
{{< figure src="/image/2020/11/17/amd-gcn-rdna-cu.webp" title="GCN/RDNA CU" caption="画像出典: []" >}}
{{< figure src="/image/2020/11/17/amd-rdna-wgp-cu.webp" title="RDNA WGP/2CU" caption="画像出典: [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) (Page 5)" >}}
{{< figure src="/image/2020/11/17/amd-cdna-cu.webp" title="CDNA CU" caption="画像出典: [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) (Page 6)" >}}
-->

{{< figure src="/image/2020/11/17/amd-cdna-rdna-gcn-cu.webp" title="CDNA/RDNA/GCN CU" caption="画像出典: [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) (Page 6), <br>&emsp; [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) (Page 5), <br>&emsp; [M&E GBGH - HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf](https://www.hotchips.org/wp-content/uploads/hc_archives/hc29/HC29.21-Monday-Pub/HC29.21.10-GPU-Gaming-Pub/HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf) (Page 21)">}}

丁度良い画像が見つからなかったため、少し不格好となってしまったが、上が *CDNA/RDNA/GCN CU* の各構造をまとめたものである。  
*CDNAアーキテクチャ* の CU 全体は *GCNアーキテクチャ* のそれに似通っているが、SIMDユニットあたりのベクタレジスタ (VGPR) は、CU あたり 256KB (GCN) から 512KB (CDNA) と、倍の容量に増量されている。  
その反面、スカラレジスタ (SGPR) は CU あたり 32KB (GCN) から 12.8KB (CDNA) と、半分の容量に減らされている。  
*RDNAアーキテクチャ* はと言うと、CU あたりの総ベクタレジスタは *GCNアーキテクチャ* と変わらないが、スカラレジスタは 20KB となっている。  

また、*CDNAアーキテクチャ* ではテクスチャフィルタユニットが省かれていることが分かる。 グラフィクス用の固定機能ユニットを省いたというのは CU 内部にも渡っている。  

| Register Size <br> per CU | Vector | Scalar |
| :-- | :--: | :--: |
| GCN | 256KB | 32KB |
| RDNA | 256KB | 20KB |
| CDNA | 512KB | 12.8KB |

#### RDNAアーキテクチャとの SIMDユニット比較 {#simd}

SIMDユニットは 512-bit幅であり、ベクタパイプラインとしては 16-wide (SIMD16)。  
Wave64 (64スレッド) を SIMD16ユニットで 4サイクル掛けて処理というのも *GCNアーキテクチャ* から変わらないものと思われる。  


{{< figure src="/image/2020/11/17/amd-cdna-rdna-simd.webp" title="CDNA/RDNA SIMD" caption="画像出典: [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) (Page 6), <br>&emsp; [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) (Page 9)" >}}

SFU (Special Function Unit) は、*CDNAアーキテクチャ* では 4基と、ここも *GCNアーキテクチャ* と変わらない。*RDNAアーキテクチャ* では 8基となっているが、SIMD32ユニットあたりであるため、SIMD幅に対するバランスは同じである。  
また、Wave Controller が保持する Wave数も同様。  

*CDNAアーキテクチャ* では SPユニット (single-precision, FP32, 単精度) 16基に対し、DPユニット (double-precision, FP64, 倍精度) 8基を持つため、FP32演算の半分のレートでFP64演算を行なえる。*RDNAアーキテクチャ* では DPユニットが 2基であるため、1/16のレートとなる。  
また、HPユニット (half-precision, FP16, 半精度) を別に 32基持ち、恐らくは SPユニットでの Packed実行と合わせて、184.6TFLOPS というピークFP16演算性能が可能になっていると思われるが、どのような処理フローになっているかは不明。  

新たに行列演算を高速に処理するための MFMA (Matrix Fused Multiply-Add) 命令に対応し、それを実行する Matrixユニットが増設された。データ精度は FP3、FP16、BF16 に対応する。  
ベクタパイプラインを用いて処理するよりも高速に、優れた電力効率で行列演算の処理が可能となっている。  

<br>
*Arcturus* の情報はこのサイトを開設した以前から Linux Kernel 等へのサポートが追加されており、個人としても少しずつ情報を追ってきた。  
*Arcturus/MI100* が正式発表されたことをオタクとして本当に嬉しく思う。  

