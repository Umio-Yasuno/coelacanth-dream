---
title: "休止中に出てきた情報の個人的ハイライト"
date: 2020-03-21T15:12:16+09:00
draft: false
tags: [ "Renoir", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: true
---

## AMDが Renoir の詳細を明らかに
資料が個人へは公開されていないため、画像等は[マイナビニュース](https://news.mynavi.jp)による記事を見ていただければと思う。  
[AMD Ryzen Mobile 4000シリーズの詳細を明らかに (1) Ryzen Mobile 4000シリーズのアーキテクチャ詳細 | マイナビニュース](https://news.mynavi.jp/article/20200316-997459/)  
<span class="hide">公開されていたら単独記事にできたのに……</span>
### フェイクダイショット
さらっとスライド内で *Renoir* のダイショットが公開されたが、明らかにフェイク。実物とは大きく違うと思われる。特にCPU、CCX部。  
画像を見てもらえればわかるが、CPUのコアに [RDNA](/tags/rdna) のWGPが流用されている。その下にあるものは [Navi14](/tags/navi14) のHUB部と一致する。  
並べられている過去のAPUダイショットと見比べても、ディテールの度合いが大きく違い、*Renoir* はのっぺりしている。  
[拡大画像 015l | AMD Ryzen Mobile 4000シリーズの詳細を明らかに (1) Ryzen Mobile 4000シリーズのアーキテクチャ詳細 | マイナビニュース](https://news.mynavi.jp/photo/article/20200316-997459/images/015l.jpg)  

一応、フェイクダイショットを作成、使用するのは機密を守るといった意味があり、そう珍しいことでもない。加えて言えば、公式が示すダイショットは、完全に信用することはできないが、全く信用できないという訳でもない。個人それぞれで判断するしかない。  

 > なお、この図はアーティストがCGで描いたものであり、実際のチップ写真ではなく、学術的な意味はない単なる壁紙である。
 >
 > {{< quote >}} [COOL Chips 22 - NVIDIAのGPUは真のクールAIチップ (1) アクセラレータとして必須となったGPU | マイナビニュース](https://news.mynavi.jp/article/20190516-822581/) {{< /quote >}}

今回の件に関しては、*Zen 2* コアの詳細は既にISSCC 2020で発表しているため[^1]、CCXの構造を隠したかったのかもしれない。  

[^1]: [Zen 2: The AMD 7nm Energy-Efficient High-Performance x86-64 Microproc…](https://www.slideshare.net/AMD/zen-2-the-amd-7nm-energyefficient-highperformance-x8664-microprocessor-core)

### PCIe 4-Lane と USB 2-Port を追加
いつかに増やしてないと考えてたが、大ハズレ。  
[Ryzen 4000 U/H-Series (Renoir) 考察 | Coelacanth's Dream](/posts/2020/01/07/renoir-consider/#i-o)
ダイショットから判断することはできないが、バランスを取ったか、Intelのモバイル向け製品のように CPU(+メモコン) + PCH といった構成を取らずに1ダイに収める分、他デバイスに多く接続できるようにする必要があったか。  

### Renoir GPUのL2キャッシュは1MB
個人的にしばらくの謎だったが、ブロックダイアグラムより[Raven](/tags/raven)/[Picasso](/tags/picasso)と同じ1MBだと判明した。  
[拡大画像 009l | AMD Ryzen Mobile 4000シリーズの詳細を明らかに (1) Ryzen Mobile 4000シリーズのアーキテクチャ詳細 | マイナビニュース](https://news.mynavi.jp/photo/article/20200316-997459/images/009l.jpg)  
ROP数も同様に8基だろうか。  

## Xbox Series X のハードウェア構成

| Xbox Series X | |
| :-- | :---: |
| CPU | 8x Cores @ 3.8 GHz (3.6 GHz w/ SMT) Custom Zen 2 CPU |
| GPU | 12 TFLOPS, 52 CUs @ 1.825 GHz Custom RDNA 2 GPU |
| Die Size | 360.45 mm<sup>2</sup> |
| Process | 7nm Enhanced |
| Memory | 16 GB GDDR6 w/ 320b bus |
| Memory Bandwidth | 10GB @ 560 GB/s, 6GB @ 336 GB/s |

引用元: <cite>[Xbox Series X: A Closer Look at the Technology Powering the Next Generation - Xbox Wire](https://news.xbox.com/en-us/2020/03/16/xbox-series-x-tech/)</cite>  
[Inside Xbox Series X: the full specs • Eurogamer.net](https://www.eurogamer.net/articles/digitalfoundry-2020-inside-xbox-series-x-full-specs)  

GPU CU 52基というのは少し中途半端な数字に思えるが、ダイとしてはCU 56基あり、内4基は歩留まり向上のための分で、全体としては2-ShaderEngine、4-ShaderArrayのような構成なのだろうか？  
ShaderArray同士は非対称構成も可能であることから、CU 54基というのもあり得る。  

### 非対称的なメモリシステム
Xbox Series X はメモリバス幅 320-bitだが、GDDR6 16Gbitと8Gbitを混載させることで計16GBを構成している。内訳は16Gbitチップが6枚、8Gbitチップが4枚となる。速度は14Gbpsで統一。  
帯域が10GBと6GBのプールで別れており、前者がGPUに最適化された高速(=広帯域)なメモリプール、後者が標準的な低速(=狭帯域)メモリプールとされている。  
帯域の数字から推察するに、高速メモリプールは10枚すべてのチップから1GBずつ、低速メモリプールは16Gbit(=2GB)チップ 6枚から1GBずつ取る形になっている。  

ロット違いのメモリ混載というのはシステムの安定性に不安が抱かれるが、Microsoftは実際に機能し、出荷できるレベルのシステムを構築していると語る。  

 > we could while at the same time building the system that would actually work and we could actually ship.
 >
 > {{< quote >}} [Xbox Series X: A Closer Look at the Technology Powering the Next Generation - Xbox Wire](https://news.xbox.com/en-us/2020/03/16/xbox-series-x-tech/) {{< /quote >}}

ゲームからは高速メモリプールに加え、低速メモリプール中の3.5GBを利用可能としているが、  
プログラマ側がメモリプールの違いを意識する必要があるのか、それともPGO(Profile Guided Optimization)のように何かを元にして自動的に割り振ってくれるのか。  

### 厳しいコスト
メモリ混載によって、確かに広帯域と容量を両立しているが、GDDR6 16Gbit\*6, 8Gbit\*4のコストが伸し掛かる。  
ダイサイズ 360.45mm<sup>2</sup>というのも負担となるだろう。  
PS4までの時代のように、微細化によるチップの統合や小型化に期待できないため、高コストになることは前々から囁かれていたが、Microsoftは一体いくらで Xbox Series X を売り出すつもりなのか。  

## PlayStation 5
[Unveiling New Details of PlayStation 5: Hardware Technical Specs [UPDATED] – PlayStation.Blog](https://blog.us.playstation.com/2020/03/18/unveiling-new-details-of-playstation-5-hardware-technical-specs/)  

ぶっちゃけ、[4gamer.net](4gamer.net)での西川善司氏の記事が素晴らしいため、それを読むことが一番だと思う。下手に解説を試みるよりずっと良い。  
[西川善司の3DGE：Mark Cerny氏のPS5技術解説プレゼンテーションを読み解く(前編)。ここまで分かったPS5のSSDとGPUの詳細 - 4Gamer.net](https://www.4gamer.net/games/990/G999027/20200319173/)  

個人的な感想を言えば、NVMe SSDのカスタムコントローラ、圧縮/展開を行なう独自のコプロセッサは、まさに専用ハードウェアだから可能な機能といった感じで大好きだ。  
後は、メモリバス幅 256-bit、GDDR6 16Gbit\*8で計16GB、GPU CU 36基と Xbox Series X よりも控えめではあるが、その分コストもマシとなっているだろう。  

