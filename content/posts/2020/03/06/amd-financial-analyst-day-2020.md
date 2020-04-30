---
title: "AMD Financial Analyst Day 2020 個人的まとめ"
date: 2020-03-06T07:03:36+09:00
draft: false
tags: [ "Radeon", "Arcturus", "CDNA", "RDNA", "RDNA_2", "Zen_3", "Zen_4" ]
keywords: [ "", ]
categories: [ "Hardware", "CPU", "GPU" ]
noindex: false
---

AMD Financial Analyst Day 2020の気になった部分を、AMDのスライド、プレスリリースからまとめた。  
[Financial Analyst Day – 2020 | Advanced Micro Devices](https://ir.amd.com/events/event-details/financial-analyst-day-2020)  
[AMD Details Strategy to Deliver Best-in-Class Growth and Strong Shareholder Returns at 2020 Financial Analyst Day | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-details-strategy-deliver-best-class-growth-and-strong)  

使用したスライドを全て公開してくれているのは嬉しいが、それぞれのファイルで被っている内容も多く、頭の中の整理に少し手間取ってしまった。  

## Table Of Content

 * [AMDの新パッケージング技術X3D](#x3d)
 * [CPU](#cpu)
 	* [2020年後期にZen 3 EPYC、Milan導入](#epyc)
	* [第4世代Ryzenも2020年中に登場](#ryzen)
 * [GPU](#gpu)
 	* [コンピュート向け GPUアーキテクチャ、CDNA](#cdna)
	* [8 GPU構成、CPUとGPUのメモリ空間統合を可能へ](#3rd-gen-infinity-architecturue)
	* [2020年後期にRDNA 2ベースGPUを発売予定](#rdna-2)

## AMDの新パッケージング技術X3D {#x3d}

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_4.webp" title="AMD Leadership Packaging" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Mark Papermaster: Future of High Performance](https://ir.amd.com/static-files/ccef22f0-f641-4fc5-861f-cb3d7d125a68)" >}}

AMDは今回、チップレット、ハイブリッド2.5D、ダイスタッキングを組み合わせ、帯域幅密度を10倍以上向上させたパッケージング技術、**X3D** を明らかにした。  
IntelのEMIB、Foveros技術への対抗と思われ、  
AMDは以前3D積層によってコンピュータを1つのパッケージに統合するビジョンを発表していたが、*X3D* はそれを実現に近づける技術とされる。[^3]  

[^3]: [【後藤弘茂のWeekly海外ニュース】AMDがCPUをフル3D積層へと進化させるビジョンを発表 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1098363.html)

## CPU
### 2020年後期にZen 3 EPYC、Milan導入 {#epyc}
*Zen 3* アーキテクチャを採用する第3世代EPYC *Milan* は2020年後期に導入予定であり、続く *Zen 4* アーキテクチャでは5nmプロセスを採用することを発表した。  

 > AMD plans to introduce the first processors based on its next-generation “Zen 3” core in late 2020. The “Zen 4” core is currently in design and is targeted to use advanced 5nm process technology.

 > 引用元: <cite>[AMD Details Strategy to Deliver Best-in-Class Growth and Strong Shareholder Returns at 2020 Financial Analyst Day | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-details-strategy-deliver-best-class-growth-and-strong)  


{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_9.webp" title="CPU RoadMap" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Forrest Norrod: Data Center Leadership](https://ir.amd.com/static-files/15702f66-d8d1-4816-8906-9612580f9aa1)<cite>" >}}

*Zen 3* のプロセスが7nmとされているが、これは *Zen 2* で採用されたTSMC N7プロセスではなく、EUV露光技術を使用した *TSMC N7+プロセス* だと思われる。  
*N7+* ではN7と比べて、15%から20%上のトランジスタ密度を可能とし、消費電力も削減される。  

 > N7+ is also providing improved overall performance. When compared to the N7 process, N7+ provides 15% to 20% more density and improved power consumption, making it an increasingly popular choice for the industry’s next-wave products.

 > 引用元: <cite>[TSMC’s N7+ Technology is First EUV Process Delivering Customer Products to Market in High Volume](https://www.tsmc.com/tsmcdotcom/PRListingNewsArchivesAction.do?action=detail&newsid=THHIHIPGTH&language=E)</cite>

*Zen 4* の5nmはTSMC N5プロセスと考えられ、そちらはN7と比べて、80%上のトランジスタ密度、16%の高速化、30%の低消費電力化を実現する見込みとなっている。[^4]  

[^4]: [「当面は微細化を進められる」 TSMCが強調 (1/2) - EE Times Japan](https://eetimes.jp/ee/articles/1905/07/news046.html)

{{< ins datetime="2020-03-07T13:20:40" >}}

しかし従来AMDが用いていた '7nm+' という表記を使わなかったことは、必ずしもTSMC N7+を採用するとは限らないことを意味する、と[AnandTech](anandtech.com)は指摘している。  
[AMD Clarifies Comments on 7nm / 7nm+ for Future Products: EUV Not Specified](https://www.anandtech.com/show/15589/amd-clarifies-comments-on-7nm-7nm-for-future-products-euv-not-specified)  
そのため、N7プロセスを改良したN7Pプロセスを採用する可能性もある。  
*N7P* はN7とデザインに互換性があるものの、EUV露光技術は使われない。  

{{< /ins >}}


### 第4世代Ryzenも2020年中に登場 {#ryzen}
*Zen 3* アーキテクチャ採用の第4世代Ryzenも2020年中に提供するとしている。  

 >  AMD is on track to bring increased performance to the gaming, content creation and productivity markets when it delivers the first “Zen 3”-based AMD Ryzen™ product in 2020.

 > 引用元: <cite>[AMD Details Strategy to Deliver Best-in-Class Growth and Strong Shareholder Returns at 2020 Financial Analyst Day | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-details-strategy-deliver-best-class-growth-and-strong)  

<br>
しかし、アーキテクチャの具体的な改良点やプラットフォームに関しては今回明らかにされなかった。  

## GPU
### コンピュート向け GPUアーキテクチャ、CDNA {#cdna}
今回新たに **CDNA** の名が出された。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_5.webp" title="Data Center GPU Road Map" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Forrest Norrod: Data Center Leadership](https://ir.amd.com/static-files/15702f66-d8d1-4816-8906-9612580f9aa1)<cite>" >}}

まだ出てきていない、7nm (恐らく *N7+*)ということから *CDNA* は [Arcturus](/tags/arcturus)を指し示していると思われる。  
*CDNA 2* がロードマップ上にあるが、採用プロセスは明らかにされていない。  
時期を考えれば *Zen4* 同様に5nmプロセスを採用しそうだが、そう記されていないのは設計がまだ完了していないためか。  

{{< details smry="ひどく個人的な話だが、CDNAという名前には少し思う所がある。" >}}
**RDNA** が *Radeon DNA* の略なのに、**CDNA** は *Compute DNA* の略だ。  

 > AMD unveiled its new AMD Compute DNA (AMD CDNA) architecture, designed to accelerate data center compute workloads. 

 > 引用元: <cite>[AMD Details Strategy to Deliver Best-in-Class Growth and Strong Shareholder Returns at 2020 Financial Analyst Day | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-details-strategy-deliver-best-class-growth-and-strong)</cite>

どうも格好がつかないというか、Radeon に対して Compute では寂しいというか、*GCN* の格好よさを受け継いで欲しかったというか。  
3つ目に関しては、**GCN** が *Graphic Core Next* の略で、*Arcturus /CDNA* はグラフィック処理をするためのユニットを持たず、GPGPU偏重のアーキテクチャとされるため、仕方がないと言えば仕方ないが。  

Arcturusでは長く、MI100では製品的過ぎたが、*CDNA* は呼びやすく、その点では好き。  
{{< /details >}}
#### 8 GPU構成、CPUとGPUのメモリ空間統合を可能へ {#3rd-gen-infinity-architecturue}

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_1.webp" title="3rd Gen AMD Infinity Architecture" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Mark Papermaster: Future of High Performance](https://ir.amd.com/static-files/ccef22f0-f641-4fc5-861f-cb3d7d125a68)<cite>" >}}

3rd Genでは最大8GPUのコヒーレント接続に対応するとされ、いつかの妄想話[^1]がかすっていたようで嬉しい限りだ。  
ただ *CDNA* は8GPUに対応するが、CPUとGPUのメモリ空間の統合に対応せず、**2nd Gen Infinity Architecture**とされ、  
8GPUが可能な上でCPUとGPUのメモリ空間統合が為された **3rd Gen Inifinity Architecture** は *CDNA 2* からとなる。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_6.webp" title="Unlocking Accelerated Computing" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Forrest Norrod: Data Center Leadership](https://ir.amd.com/static-files/15702f66-d8d1-4816-8906-9612580f9aa1)<cite>" >}}

図を鵜呑みするならば、1st Genと2nd GenのようにPCI ExpressでCPUにそれぞれのGPUを接続するのではなく、Infinity FabricでCPUと8GPUのクラスタに接続するのだろうか。  
図がひどく複雑になるため簡略化している可能性もある。  

そしてGPUネットワークをよく見るとわかるが、あるGPUの対角に位置するGPUには線がない。（これも簡略されてる可能性があるが）  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_3.webp" title="3rd Gen AMD Infinity Architecture - GPU Network" caption="対角のGPUには繋がれていない。<br><br>画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Forrest Norrod: Data Center Leadership](https://ir.amd.com/static-files/15702f66-d8d1-4816-8906-9612580f9aa1)<cite>" >}}

<del>しかしArcturus関連のコードから、ArcturusにはXGMIに最適化されたSDMAエンジンは8基ある。[^1]  
そしてGPUネットワークの繋がれた線は、隣のGPUとそれより離れたGPUとで太さが違う。  
そこで、隣の2GPUとはそれぞれ2リンク、それより離れた4GPUとはそれぞれ1リンクとすれば、合計リンク数がSDMAエンジンの数 8基と一致する。  
恐らくそういったネットワーク設計になっているのではないかと予想する。</del>  
<ins datetime="2020-03-07T16:37:28">大間違い。XGMIに最適化されたSDMAエンジンは6基だった。[^5]  
使用するSDMAエンジン数はどのGPUに対しても変わらない可能性のが高い。  
というかSDMAエンジン数=リンク数のように書いてたが、リンク数はまた別の話だ。<span class="hide">間抜けなシーラカンス……</span></ins>  

[^5]:[drm/amdkfd: Set number of xgmi optimized SDMA engines for arcturus](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdkfd?h=amd-staging-drm-next&id=b6689cf7b9cd2600ebd6981e19fb5f697819a60b)  

*Vega20* は複数GPUのコヒーレント接続でリングバスを構築していたが、8GPUの場合でもベースとしてそれを残すのだろうか。  

{{< ins datetime="2020-03-06T19:06:04" >}}
こういったネットワークを *コーダルリング (Chordal Ring)* と呼ぶらしい。  

<https://twitter.com/kato_kats/status/1235694954971717632>  
<https://twitter.com/chiakokhua/status/1235694201150439424>  
{{< /ins >}}


気になるのは時期と8GPUシステムが採用されるかどうかで、AMDが納入する予定の次世代Exascaleスパコン、*Frontier* と *El Capitan* はどちらもノードあたり GPU 4 : CPU 1 の設計とされている。  

 > New approach using accelerator-centric compute blades (in a 4:1 GPU to CPU ratio, connected by the 3rd Gen AMD Infinity Architecture for high-bandwidth, low latency connections) to increase performance for data-intensive AI, machine learning and analytics needs by offloading processing from the CPU to the GPU.

 > 引用元: <cite>[HPE and AMD power complex scientific discovery in world’s fastest supercomputer for U.S. Department of Energy’s (DOE) National Nuclear Security Administration (NNSA) | HPE](https://www.hpe.com/us/en/newsroom/press-release/2020/03/hpe-and-amd-power-complex-scientific-discovery-in-worlds-fastest-supercomputer-for-us-department-of-energys-doe-national-nuclear-security-administration-nnsa.html)</cite>

*Frontier* は2021年納入予定であり、調整の時間を考えると *CDNA* を、  
*El Captitan* は2022年か2023年早期に納入予定、そして *3rd Gen Infinity Architecture* に対応することから、*CDNA 2* をGPUに採用するとされる。  
別段スパコンだけがGPGPUを使う訳ではなく、データセンターや別のスパコンでも採用されるはずだが、それらスパコンが GPU 4 : CPU 1となると、今の段階では GPU 8 : CPU 1 の設計の採用予定は無いのかもしれない。  
ボード開発や、8GPUに対応したブリッジに課題が残っている可能性もある。8GPUもの規模になるとIntelが{{< xe class="hpc" >}}を16基搭載する *Ponte Vecchio* の構想で示したように、GPUをPCIeカードに収めるのではなく、ボード上に8GPUを搭載する形のが良いだろう。[^2]  
AMDが今回、パッケージング技術 **X3D** を発表したのは、その布石かもしれない。もしかしなくてもIntel、AMDで近い形になるか。  

そして、**3rd Gen Infinity Architecture** により、CPU-GPU間の帯域はPCIe Gen4の倍近くまで向上する。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_10.webp" title="AMD 3rd Gen Infinity Architecture Enables Accelerated Computing" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Mark Papermaster: Future of High Performance](https://ir.amd.com/static-files/ccef22f0-f641-4fc5-861f-cb3d7d125a68)" >}}

倍近くということから、PCIe Gen5を想定している可能性がある。  
また、CPUとGPUのメモリ統合による、従来あったアドレス変換のオーバヘッドの削減も効果しているはずだ。  

### 2020年後期にRDNA 2ベースGPUを発売予定 {#rdna-2}
{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_8.webp" title="Gaming GPU RoadMap" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Rick Bergman: Driving Growth Across PCs and Gaming](https://ir.amd.com/static-files/dd12bed4-a96e-42e7-b2d9-3940183e2473)<cite>" >}}

*RDNA 2* の製品情報までは出なかったが、RDNAからの進化点は明らかにされた。  

*RDNA* は *GCN* との比較で電力効率が50%向上したが、*RDNA 2* ではそのさらに50%の向上が達成されるとする。  
*RDNA 2* も *N7+* で生産されると思われるが、電力効率の向上はプロセス技術の進化だけではなく、マイクロアーキテクチャの改良、物理設計の最適化で実現するとしている。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_11.webp" title="AMD RDNA 2 Perf/Watt Improve" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - David Wang: Driving GPU Leadership](https://ir.amd.com/static-files/321c4810-ffe2-4d6c-863f-690464c033a9)<cite>" >}}

また、*Ray Tracing* と *Variable Rate Shading* の対応も明言されたが、それらはXbox Series X、PS5からの情報で既に明らかになっていたため、そこまでの驚きはないかもしれない。  

性能面でも妥協のない4Kゲーミングをもたらす、とAMDは述べており、  
相変わらず出てこない [Navi12](/tags/navi12) では [Navi10](/tags/navi10) 以上のゲーミング性能を目標とせず、  
*Navi10* のターゲットであった1440Pより上の4Kは *RDNA2* で果たす、ということと思われる。  
しかし、今回の発表の中でRDNAアーキテクチャの特徴に、モバイルからクラウドゲーミングまでカバーするスケーラブルな構成であることがあげられた。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_12.webp" title="AMD RDNA Architecture" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - David Wang: Driving GPU Leadership](https://ir.amd.com/static-files/321c4810-ffe2-4d6c-863f-690464c033a9)<cite>" >}}

Navi12はRDNA世代では唯一SR-IOV (MxGPU)用のDevice IDが用意されており、その機能が強化されている。  
そのことと合わせると、昨今広まりつつあるクラウドゲーミング向けのGPUとして *Navi12* が出てくる可能性も十分にある。  

*RDNA 2* の動作デモはなかったが、実シリコンにて DXR 1.1 を実行した際のスクリーンショットは公開された。  

そしてAMDは2020年後期に最初の*RDNA 2* ベースの製品を発売する予定にある。  

 > The first AMD RDNA 2-based products are expected to launch in late 2020.

 > 引用元: <cite>[AMD Details Strategy to Deliver Best-in-Class Growth and Strong Shareholder Returns at 2020 Financial Analyst Day | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-details-strategy-deliver-best-class-growth-and-strong)  

#### RDNA 3
*RDNA 3* もロードマップ上に姿を現したが、*CDNA 2* 同様に Advanced Node とされ、具体的なプロセスは明らかにされなかった。  

<hr>
<div class="reference">参考:</div>

 * <cite>[Powering the Exascale Era | AMD](https://www.amd.com/en/products/exascale-era)</cite>

[^1]: [AMDは8 GPUsのシステムを計画している？ | Coelacanth's Dream](http://localhost:1313/posts/2019/11/26/amd-planning-8gpus-system/)
[^2]: [Ponte Vecchio: The Old Bridge in the land of Gelato - Analyzing Intel’s Discrete Xe-HPC Graphics Disclosure: Ponte Vecchio, Rambo Cache, and Gelato](https://www.anandtech.com/show/15188/analyzing-intels-discrete-xe-hpc-graphics-disclosure-ponte-vecchio/3)
