---
title: "Intel Architecture Day 2020 個人的まとめ　―― XeHP は 1-Tile 512EU、XeLPアーキテクチャ詳細"
date: 2020-08-14T04:24:22+09:00
draft: false
tags: [ "DG1", "Alder_Lake", "Tiger_Lake", "Gen12" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU", "GPU" ]
noindex: false
# summary: ""
---

Intel は 2020/08/13 にバーチャルイベント「Architecture Day 2020」開催、次世代 CPU/GPU のアーキテクチャ詳細、次々世代 CPU/GPU の概要を発表した。  

記事タイトルにある通り個人的なまとめであり、あえて取り上げていない内容もある。これまでの情報と結びつけるメモ書きに近い。  
内容の網羅、特に *Tiger Lake* の詳細情報を望む方は以下を確認していただきたく思う。  

 * [Intel Delivers Advances Across 6 Pillars of Technology, Powering Our Leadership Product Roadmap | Intel Newsroom](https://newsroom.intel.com/editorials/advances-across-6-pillars-technology/)
 * [Fact Sheet: - intel-2020-architecture-day-fact-sheet.pdf](https://newsroom.intel.com/wp-content/uploads/sites/11/2020/08/intel-2020-architecture-day-fact-sheet.pdf)
 * [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)

{{< pindex >}}

 * [CPU](#cpu)
   * [Alder Lake は Golden Cove と Gracemont のハイブリッド構成](#adl)
 * [GPU](#gpu)
   * [{{< xe class="HP" >}} は 1-Tile 512EU](#xe-hp-1t-512eu)
   * [ゲーミング向け Intel GPU {{< xe class="HPG" >}}](#xe-hpg)
   * [{{< xe class="lp" >}} アーキテクチャ詳細](#xe-lp-detail)
   * [{{< xe class="lp" >}}ベースのサーバー向けGPU SG1](#sg1)

{{< /pindex >}}

## CPU {#cpu}
### Alder Lake は Golden Cove と Gracemont のハイブリッド構成 {#adl}
ハイブリッドコア構成を採用する [Alder Lake](/tags/alder_lake) が、次々世代`Core`アーキテクチャ *Golden Cove* 、次世代`Atom`アーキテクチャ *Gracemont* を組み合わせたものであることが明かされた。  
性能ではクライアント向けをカバーしつつ、優れた電力比性能を発揮するとしている。  

## GPU {#gpu}
### {{< xe class="HP" >}} は 1-Tile 512EU {#xe-hp-1t-512eu}

{{< xe class="hp" >}} GPU はコアとなる Tile から構成され、Tile数は 1-Tile から 2-Tile、または 4-Tile となる。複数のタイルはマルチコアGPU であるように動作し、高いスケーリング性能を持つ。  
複数の Tile は 1つのパッケージに収められ、Tile 間は EMIB 技術で接続される。  
また、製造プロセスは *Intel 10nm Enhanced SuperFin* とされており、*Tiger Lake* や *DG1* の製造プロセスから次世代のものとされる。  

そして、下記画像は、動画内で {{< xe class="hp" >}} のスケーリング性能を示すデモを行なった時のスクリーンショットだが、ログの部分に `Compute units: 512` と表示されている。  

{{< figure src="/image/2020/08/14/xe-hp-scaling.webp" caption="画像出典: [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)" >}}

ログの内容から、実行しているソフトウェアは [clpeak](https://github.com/krrishnarraj/clpeak) は思われる。[先日、自分が **RX 560** で CU数のスケーリング性能を調査した時](/posts/2020/08/06/polaris11-cu-scaling-test/)にも使ったもので、ログのフォーマットが一致する。  

 >       Platform: AMD Accelerated Parallel Processing
 >         Device: gfx803
 >           Driver version  : 3137.0 (HSA1.1,LC) (Linux x64)
 >           Compute units   : 16
 >           Clock frequency : 1196 MHz
 >       
 >           Single-precision compute (GFLOPS)
 >             float   : 2356.53
 >             float2  : 2314.70
 >             float4  : 2265.87
 >             float8  : 2248.82
 >             float16 : 2207.62

この時 `Compute units` の行に表示されるのは、シェーダープロセッサ数ではなくそれを多数内包する *CU (AMDGPU)、EU (Intel GPU)* の数だ。  
よって、{{< xe class="hp" >}} は 1-Tile 512EU と考えられる。  

ただ、**RX 560** を使った調査の時にも書いたが、`clpeak` はメモリ性能が影響しないベンチマークソフトウェアであり、それもあって性能が単純にスケーリングしやすくなっている。  
マルチGPU構成であっても性能がスケーリングすることを示す目的があるのかもしれないが、もっと大規模な、メモリが重要となるソフトウェアの場合、どこまでスケーリングするかはデモから測ることができない。  

### ゲーミング向け Intel GPU {{< xe class="HPG" >}} {#xe-hpg}
ゲーミング向けに最適化された {{< xe >}}系アーキテクチャ、{{< xe class="hpg" >}} の存在が明らかにされた。  
{{< xe class="lp" >}}の優れた電力比グラフィック性能、{{< xe class="hp" >}}のスケーリング性能、{{< xe class="hpc" >}}の最適化された周波数を組み合わせたアーキテクチャとしている。  
メモリには帯域と帯域あたりの消費電力に優れる HBM系ではなく、コスト比に優れる GDDR6 を採用するとしている。  
ハードウェアレイトレーシングもサポートし、その他グラフィック向けの機能も多くサポートする。  

外部ファウンダリによって製造され、2021年に出荷予定。  


### {{< xe class="lp" >}} アーキテクチャ詳細 {#xe-lp-detail}
これまでにもちょくちょく OSS から判明した [{{< xe class="lp" >}} /Gen12アーキテクチャ](/tags/gen12) の構成を記事にしてきたが、今回公式により詳細が語られた。  

EU部は前世代の *Gen11アーキテクチャ* が以下のように、Thread Control を EUごとに持ち、演算部が *4-wide FP/INT ALU* と *4-wide FP/Extend Math ALU* となり、浮動小数点演算では SIMD8 を構成していたのに対し、

{{< figure src="/image/2020/08/14/gen11-eu.webp" caption="画像出典: [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)" >}}

*Gen12アーキテクチャ* では、Thread Control が 2つ EU をペアとし、またがるものとなった。  
演算部は、*8-wide FP/INT ALU* で SIMD8 を構成するものとなり、データ精度 INT16 を INT32時の 2倍、INT8 は 4倍のスループットで処理可能となった。複雑な計算を処理する *Extend Math ALU* は EU ごとに 2-wide となり、そこは *Gen11アーキテクチャ* より減っている。  

{{< figure src="/image/2020/08/14/gen12-eu.webp" caption="画像出典: [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)" >}}

キャッシュ構成としては、Sub-Slice 内に L1 Data Cache(L1/Tex$) が新設され、L3 Cache は最大 16MBを取ることが可能となっている。[DG1](/tags/dg1)は以前より L3 Cache 16MB であることが判明していたが、それが {{< xe class="lp" >}}アーキテクチャとして最大の容量であるようだ。[^dg1-cd]  

[^dg1-cd]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)<br>[Intel、media-driver に DG1 へ向けたパッチを投稿　―― GPGPUに傾いた DG1 のキャッシュ設定 | Coelacanth's Dream](/posts/2020/06/25/intel-media-driver-dg1/)

{{< figure src="/image/2020/08/14/xe-lp-cache.webp" caption="画像出典: [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)" >}}

この L1 Data Cache だが、ブロック内には L1/Tex$ とあり、これまで Sampler部に含まれていた Tex$(Texture Cache) を汎用的に使えるよう移動させたと考えていいのだろうか。  

*EU、Texture Sampler、Pixel Backend* 等のコア/エンジン部は前世代から 1.5倍の規模となった。  
ただ *Pixel Backend* に関しては、*DG1* と *TGL GT2* は [intel/compute-runtime](https://github.com/intel/compute-runtime) から前世代と変わらない `16 pixels/clock` であるように読めるが[^dg1-tgl-fill-rate]、果たして単なる自分の読み違いか、それとも `24 pixels/clock` の {{< xe class="lp" >}}アーキテクチャベースの GPU が別に存在するのか。  

[^dg1-tgl-fill-rate]: <https://github.com/intel/compute-runtime/blob/68294a0d68adfdf11236c18050c926e488d1a057/opencl/source/gen12lp/hw_info_dg1.inl#L138> <br><https://github.com/intel/compute-runtime/blob/68294a0d68adfdf11236c18050c926e488d1a057/opencl/source/gen12lp/hw_info_tgllp.inl#L136>

{{< figure src="/image/2020/08/14/xe-lp-engine.webp" caption="画像出典: [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)" >}}

また、OSS 等では *{{< xe class="lp" >}} /Gen12アーキテクチャ* より、Sub-Slice ではなく Dual Sub-Slice と記述されるようになったが、その真意は明らかにはならなかった。  
Texture Sampler は 12 Sub-Slices (6 Dual Sub-Slices) 相当であるため、グラフィック部を減らし、Sub-Slice 内の EU を増やすことでコンピュート性能重視とする策とも違うだろう。  
新設された L1 Data Cache を共有する単位としての Dual Sub-Slice なのだろうか？  


### {{< xe class="lp" >}}ベースのサーバー向けGPU SG1 {#sg1}
*Tiger Lake GPU* 、*DG1* 同様に *{{< xe class="lp" >}}アーキテクチャ* をベースとする、サーバー向けのディスクリートGPU **SG1 (Server GPU)** の存在が明らかにされた。  
小型フォームファクタで構築するデータセンターや低レイテンシで高密度な Android 向けクラウドゲーミング、映像ストリーミングサービスをターゲットとしている。  


今年後半に出荷される予定であり、まもなく生産を開始するとしている。  

