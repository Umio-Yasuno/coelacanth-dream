---
title: "Hot Chips 32 のスライド資料がパブリックに"
date: 2020-12-23T13:53:33+09:00
draft: false
tags: [ "Xe-HP", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

2020/08 に開催された Hot Chips 32 にて発表された各資料、各スライドが一般にも公開された。  

 * <https://www.hotchips.org/>
  * <https://www.hotchips.org/program/conference/>

動画も Hot Chips の Youtubeチャンネル [hotchipsvideos - YouTube](https://www.youtube.com/user/hotchipsvideos/featured) に逐次アップロードされている。  

改めて発表を眺めると、*Ice Lake-SP* 、*Tiger Lake* 、*Xe GPU* 、*FPGA* と Intel の発表が多く、Keynote も Intel が務めている。  
AMD は *Renoir APU* について発表しており、それと Microsoft より、CPU部に *Zen 2 8-Core* 、GPU部に *RDNA 2* を採用した **Xbox Series X SoC** の詳細が発表されている。  
NVIDIA からは 7nmプロセスで製造される **A100 GPU** のアーキテクチャと新機能について発表されている。  

## Intel Xe GPU 復習

Intel の Xe GPU、特に *{{< xe class="hp" >}}* の知識が抜け落ちていることに気が付いたため、発表内容を整理しながら復習したい。  

 * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)

*Xe/Gen12アーキテクチャ* はハイレベルで見ると、シェーダー等を処理する 3D/Compute Slice、メディアエンコード/デコードを処理する固定機能部 Media Slice、Intel GPU で言う L3キャッシュやその他 I/O がある Memory Fabric/Cache に分かれている。  
3D/Compute Slice にはグラフィクス処理のための固定機能部 (Geometry, Raster, Pixel Backend/ROP) と Sub-Slice が収められている。  
Sub-Slice の数は可変で、*{{< xe class="lp">}}アーキテクチャ* は Sub-Slices数は最大 16基の構成となっている。Slice数は 1基。  

Sub-Slice には、実行に直接関わる EU (Execution Unit)、スレッドディスパッチャー、各キャッシュメモリ (命令キャッシュ、L1データキャッシュ、テクスチャキャッシュ)、SLM (Shared Local Memory)、ロード/ストアユニット、テクスチャサンプラー、メディアサンプラーが収められている。  
*Xe/Gen12アーキテクチャ* では、Sub-Slice あたりの EU数は 16基。  
また、*{{< xe class="hpg" >}}* が対応する HWレイトレーシングを処理するユニットはここに配置され、Sub-Slice ごとの数となる。  
何故 EU 16基あたり 1 RTユニットというバランスを取ったのか、という質問に対して、発表者の David Blythe 氏は、それがスケーラビリティにおいて適切と思ったから、と回答している。また、RTユニットごとのスループットも変更可能としている。  
RTユニットが BVHデータを読み込む上で、キャッシュ、ローカルメモリをどのように扱うかも重要になると思われる。  
SLM は *Gen9アーキテクチャ* では GPU L3キャッシュの一部を割り当てて利用する形だったが、*Gen11アーキテクチャ* から分離され、Sub-Slice部に配置するようになった。これにより、EU から直接 SLM にアクセス可能となり、レイテンシ削減とスループット向上という効果を得ている。  

EU にはスレッドスケジューラー、レジスタファイル、Branch/Sendユニット、演算器とそのベクタパイプで構成される。  
ベクタパイプには 倍精度演算 (FP64) や行列演算用の XMX が追加可能で、これらは *{{< xe class="hp">}}* 、*{{< xe class="hpc" >}}* で実装されると見られる。  

Memory Fabric には各用途に割り当てられる L3キャッシュが配置されており、GTI (Graphics Technology Interface) を介してメモリにアクセスする。  
*RAMBOキャッシュ* という *{{< xe class="hpc" >}}* に実装されるキャッシュもこのレベルに配置される。  
*{{< xe class="hp" >}}* はマルチ GPU構成を取り、1パッケージ内に EMIB技術を用いてタイルと呼ぶ各GPUチップを接続し、複数搭載する。タイル数は 1/2/4タイル構成の 3種類が予定されている。  
タイルは Memory Fabric から接続される。  
*{{< xe class="hpc" >}}* に実装される CXL ベースの GPU間接続技術には **Xe Link** というそれらしい名前が付いているが、*{{< xe class="hp" >}}* のタイル間接続は **Tile-to-Tile** とどこか素っ気ないものである。  

*{{< xe class="hp" >}}* は機械学習とメディア処理に最適化されたものとなり、メモリに HBM2e を採用する。メディア処理に最適化とのことから、Media Slice部を改良、あるいは同時に複数搭載すると思われる。  

