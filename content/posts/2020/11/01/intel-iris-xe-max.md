---
title: "Intel、モバイル向け dGPU 「Iris Xe Max」 を正式発表"
date: 2020-11-01T04:05:29+09:00
draft: false
tags: [ "DG1", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Intel は現地時間 2020/10/31 付で、*{{< xe class="lp" >}} /Gen12LPアーキテクチャ* を採用するモバイル向け dGPU **Iris {{< xe >}} MAX** を正式発表した。  

 * [Innovation Extends with Intel Iris Xe MAX Graphics and Deep Link | Intel Newsroom](https://newsroom.intel.com/news/iris-xe-max-discrete-graphics-deep-link/)

{{< pindex >}}

 * [DG1 について](#dg1)
 * [Iris Xe Max 詳細](#detail)
 * [今後の Intel GPU](#next-gpu)

{{< /pindex >}}

## DG1 について {#dg1}

*{{< xe class="lp" >}} /Gen12LPアーキテクチャ* の dGPU には、ソフトウェア開発者向けの **DG1 SDV** 、サーバー向けの **SG1** 、今回発表されたモバイル向けの **Iris {{< xe >}} MAX** がいるが、これらは同一のダイをベースにしていると認識している。  
Intel は **SG1** [^sg1] と **Iris {{< xe >}} MAX** のパッケージとそれに搭載されるダイの画像を公開しており、{{< comple >}} **SG1** の方は画質が荒いが {{< /comple >}} 両方共同じであるように見える。  
このサイトでは、**Iris {{< xe>}} MAX** らのベースとなるダイを *DG1* としている。  

[^sg1]: [The Intel® Server GPU - ideal for Android cloud gaming and OTT live streaming - IT Peer Network](https://itpeernetwork.intel.com/intel-server-gpu/)

## Iris {{< xe >}} MAX 詳細 {#detail}

**Iris {{< xe>}} MAX** は *Tiger Lake* と同じ Intel 10nm SuperFinプロセスで製造される。  
メモリインターフェイスは LPDDR4x-4266 128-bit に対応し、ピークメモリ帯域は 68 GB/s。  
EU数は **Tiger Lake GT2** と同じ 96基。動作クロックは **Core i7-1185G7** の iGPU より 300MHz 高い 1.65GHz となる。  

TDP、消費電力は不明だが、*Tiger Lake (CPU+GPU)* と **Iris {{< xe >}} MAX (dGPU)** を合わせた消費電力枠を、その時のワークロードによって適した側に多く割り当てることで、より効率良く処理を行なう Dynamic Power Share に対応する。  
Dynamic Power Share についてすっかり言ってしまうなら、Intel版 AMD SmartShift である。  
*Tiger Lake* と組み合わせた機能には他に、AI処理、メディアのエンコード処理の高速化に対応する。  

また、**Iris {{< xe >}} MAX** は PCIe Gen4 に対応し、*Tiger Lake-U/UP3* とは PCIe Gen4 x4 で接続される。  
ただ、ベースとする GPUチップ/ダイが共通であろう **DG1 SDV** のカードには、PCIe x8 分が配線されていたため[^dg1-sdv]、PCIe Gen4 x4 で接続というのは *Tiger Lake-U/UP3* 側の仕様に引っ張られていると思われる。  
11th Generation Intel® Core™ Processor (UP3 and UP4)[^tgl-up3-doc] には、PCIe Gen4 x8 (x4 \*2) が、NVMe SSD と dGPU にそれぞれ x4 ずつ割り振られ、そのどちらかを利用できるとある。  

dGPU ながら、メモリに GDDR系ではなく LPDDR4x を採用したのは製造プロセスと関係していると思われる。  
*Tiger Lake* と同じ LPDDR4xメモリ対応とすれば、メモリコントローラー、PHY (物理層) 等の設計をそのまま持ってくることが可能であり、メモリとディスプレイ出力部との調節を行なう手間も減らせる。  
というよりは *DG1* の設計に掛けるリソースを節約する中で、*Tiger Lake* をベースとする、ことがまずあったのではないかと思う。  
*Tiger Lake* と *DG1* で違うのは、CPU、LLC (Last Level Cache)、USB4 等の有無、GPU L3キャッシュの容量くらいとされる。  

*Tiger Lake* は LPDDR5メモリにも対応しているため、その点も共通とすれば、将来的に LPDDR5メモリを搭載した **Iris {{< xe >}} MAX** の上位製品が考えられるが、  
そういった製品を出すかどうかはびみょうで、出さない気がする。  

[^dg1-sdv]: [Intelが開発者向けのPCIeカード型DG1を公開 | Coelacanth's Dream](/posts/2020/01/10/intel-dg1-unveil/)<br>
[^tgl-up3-doc]: [Intel® Core™ i7-1185G7 Processor (12M Cache, up to 4.80 GHz, with IPU) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/208664/intel-core-i7-1185g7-processor-12m-cache-up-to-4-80-ghz-with-ipu.html)

## 今後の Intel GPU {#next-gpu}

サーバー向けの **SG1** は今年中に出荷予定にあり、2021年前半には OEMパートナーを通してデスクトップ向け *DG1* が提供される。  
また、2021年にはデータセンター向けの {{< xe class="hp" >}}、ゲーミング向けの {{< xe class="hpg" >}} が予定されている。  

{{< xe class="hp" >}} はタイルと呼ぶダイを、EMIB によって最大 4タイルを 1つのパッケージに搭載する構成となる。タイルあたりの EU数は最大 512基。 {{< xe class="hp" >}} は Intel の次世代プロセス、10nm Enhanced SuperFin で製造される予定。  
{{< xe class="hpg" >}} は規模、構成は不明だが、HWレイトレーシングに対応し、メモリにはコスト比に優れる GDDR6 を採用する。  
製造は Intel ではなく、外部ファウンダリに委託される。[^intel-arch-day-2020]  

[^intel-arch-day-2020]: [Intel Architecture Day 2020 個人的まとめ　―― XeHP は 1-Tile 512EU、XeLPアーキテクチャ詳細 | Coelacanth's Dream](/posts/2020/08/14/intel-architecture-day-2020/)
