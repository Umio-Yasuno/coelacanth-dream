---
title: "謎のIntel SoMaチップ推測"
date: 2020-03-03T01:03:47+09:00
draft: false
tags: [ "Unknown", "SoMa" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel" ]
noindex: false
---

少し前にeBayに現れ、話題になった謎のMCM (Multi Chip Module)構成のプロセッサ、SoMaだが、[Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/)氏によって配線層、ダイショットの画像が公開された。  
それを基にSoMaの正体を推測してみる。  
先に結論を言うと、確かな答えは全く得ることができず、個人の推測に留まる。  

## 推測
### 前提情報

 > * IHS (Intergrated Heat Spreader) は Haswell/Skylake と同一
 > * LGA115xらしきパッケージ
 > * PCBの厚さは Haswell と Skylake の間
 > * パッケージ裏面にチップコンデンサは無し
 > * IHSの向きが間違っている (?)
 > * IHSにあるバッチナンバーは2014年後半に生産されたことを示す
 > * エンジニアサンプリングではなく、箱にはS-SpeC SR26Uの記載があった (このナンバーに関して情報は無し)
 > * かなり奇妙なIPN (Intel Part Number) が割り当てられている
 > * パッケージには4ダイが実装されている

 > 情報元: <cite>[Information request on Intel “SoMa” : intel](https://www.reddit.com/r/intel/comments/e082ng/information_request_on_intel_soma/)</cite>

### ダイレイアウト

{{< figure src="/image/2020/03/03/intel-soma-1die_1.webp" title="Intel SoMa 1-Die" caption="56.60mm<sup>2</sup><br>画像元: <cite>[Intel@unknown_soma_cpu@SoMa_L450C234_Malay___DSCx8_poly@5x… | Flickr](https://www.flickr.com/photos/130561288@N04/49603570216/in/photostream/)" >}}

ダイ内は4つのクラスタに別れ、そして2つのクラスタを縦にまとめ、片方を180度回転させて配置している。  
クラスタ間の境界がはっきりしており、ダイショットに配線が見られないが、クラスタ同士はシリコンインターポーザ内で接続しているのだろうか？  
ダイ間はパッケージ内のサブストレートで接続されているのかもしれない。  

#### 全体
まずメモリコントローラー、PHYがない。  
またそれぞれのユニットが離れている（=大して最適化が為されていない）ことは、テストチップであることを裏付けている。  
何らかの外部コントローラー、FPGA等を接続してデータをロードする仕組みだろうか。  

そしてMCMではあるが、ダイ間接続用の広いI/Oも特別見当たらない。  
そのため、MCMとしたのは性能よりも研究を目的としたと思われる。  

#### 各ユニット
ここでは1つのクラスタから推測することとする。  
{{< figure src="/image/2020/03/03/intel-soma-1die.webp" title="Intel SoMa 1-Cluster" caption="画像元: <cite>[Intel@unknown_soma_cpu@SoMa_L450C234_Malay___DSCx8_poly@5x… | Flickr](https://www.flickr.com/photos/130561288@N04/49603570216/in/photostream/)" >}}

まず目に付くのは、クラスタの大部分を占める計64基のユニットだが、境界が曖昧で入り組んでいることから、設計ソフトの自動配置配線を用いていると思われる。  
また特別ユニットが大きくなく、大量に並べられていることから、x86 IAコアよりも、もっと構成がある程度単純な演算ユニットだろう。  
ひとまず、仮に演算ユニットとする。  

左上にあるユニットも自動配置配線と思われるが、演算ユニットよりももう少し複雑な構造をしている。  
演算ユニットのスケジューリング等を担当しているのだろうか？  

左中央にある計26基のユニットは、I/OのPHY (物理層、PHYsical layer)、そして間にある6基のユニットはそれらのPLL (Phase-Locked Loop)であるように見える。  
ダイの外側となるように配置されていることもI/Oだと考える要因の1つだ。  
しかし、クラスタとダイの接続、どちらかに使われているかは不明。分割して使って居る可能性もある。  
ダイの左右にあるため、横隣のダイとは近い距離で接続できるかもしれないが、縦隣とは遠くなり、  
4ダイであることを前提としていたならば、各クラスタのI/Oがそれぞれ外側へ別方向になるように配置するように思う。  
その方が4ダイよりも多い構成に対応可能のスケーラブルな構成になるはずだ。  
このクラスタ配置は不自然、というか意図が読みにくい。  

左下のユニットは内部のブロックの境界がはっきりとしており、かなり小規模なx86 IAコアか、上記I/Oのエージェントの可能性がある。恐らく後者だろう。  
もしx86 IAコアだとしたらL2キャッシュも削られた、極めて小規模なものとなる。それにメモリコントローラーも無いとなるとx86 IAコアを無理に搭載する理由も無いように思える。  

### 個人的な結論
#### 研究用説
シリコンインターポーザでのクラスタ間接続と、パッケージ内のダイ間接続を比較、研究するためのチップ、という *説* 。  
またはメッシュバスのネットワーク検証用か。  

#### Co-processor 説
x86 IAコアらしきものも見当たらないことから、ホストとなるCPUを必要とする、 *Co-processor* のようにも思える。  
それならば計64基もの演算ユニットがあるのも自然だろう。  

そしてメモリを直接接続できないことを考えると、ホストCPU側のメモリにアクセスできるようなI/Oである必要があるはずだ。  
Intelソケットに対応したパッケージに収めていることと合わせると、使うならばQPI[^1]の可能性がある。  
左中央の計26基のユニットはQPI PHY、その間にある6基のユニットはQPI PLL、左下のユニットはQPI Agentとなる。  

XeonにQPIでCo-processor（多くはFPGA）を接続する構想はあり、Altera[^2]よりIntel CPUソケットに対応したディスクリートFPGAが提供されていた。  
<cite>[Altera Demonstrates Industry’s First QPI 1.1 FPGA Home Agent for Enhanced Server Capabilities | Intel Newsroom](https://newsroom.intel.com/news-releases/altera-demonstrates-industrys-first-qpi-1-1-fpga-home-agent-for-enhanced-server-capabilities/)</cite>  
<cite>[【後藤弘茂のWeekly海外ニュース】Intelのサーバー戦略の要となるXeon PhiとFPGA - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1008797.html)</cite>  

SoMaはそういったもののテストチップとして、もしくはUPI[^3]の開発のために作られた、という *説* 。  
それかFPGA以外にQPI接続のCo-Processorを出す計画があったか。  

ただこの説には問題があり、  
まずソケット間のQPI接続に対応したLGA115xソケットCPUはない（はず）。  
少なくとも自分には見つけられなかった。  
SoMaのパッケージが、形状こそLGA1156近くともピン配置は大きく違うという可能性はあるが、当然確証はない。  
ソケットが非対称でも可能なのか、という疑問も浮かぶ。  

<br>
2つの説の要素を含めた、Co-processorにおける接続方法の違いの検証用という説も考えとしては浮かぶが、イマイチ確信は得られない。  

そのため、やはりIntel SoMaチップは謎のままだ。  

<hr>
<span class="reference">参考:</span>

 * <cite>[Xeon SoMa: Geheimnisvolles MCM-Design von Intel aufgetaucht (Update: Die-Shots) - Hardwareluxx](https://www.hardwareluxx.de/index.php/news/hardware/prozessoren/51658-xeon-soma-geheimnisvolles-mcm-design-von-intel-aufgetaucht.html)
 * <cite>[Making MultiCore: A Slice of Sandy | The CPU Shack Museum](http://www.cpushack.com/2018/03/24/making-multicore-a-slice-of-sandy/)</cite>


[^1]: QuickPath Interconnect [Intel® QuickPath Interconnect](https://www.intel.com/content/www/us/en/io/quickpath-technology/quickpath-technology-general.html)
[^2]: Intelより2015年に買収された。
[^3]: Ultra Path Interconnect [Intel® Xeon® Processor Scalable Family Technical Overview | Intel® Software](https://software.intel.com/en-us/articles/intel-xeon-processor-scalable-family-technical-overview)
