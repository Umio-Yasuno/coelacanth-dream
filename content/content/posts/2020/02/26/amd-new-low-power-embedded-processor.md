---
title: "AMD、新たな低消費電力向けRyzen Embeddedプロセッサー発表 ――R1305G, R1102G"
date: 2020-02-26T09:20:57+09:00
draft: false
tags: [ "Raven2", "Dali" ]
keywords: [ "", ]
categories: [ "Hardware", "APU" ]
noindex: false
---

前々から名前だけは出てきていた、より低消費電力なRyzen Embedded R1305G /R1102G だが、202/02/25付けでAMDより正式にプレスリリースが出された。  
[New AMD Processors Drive High-Performance Computing for Embedded Industry | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/new-amd-processors-drive-high-performance-computing-embedded)  

開催中である embedded world 2020 に合わせての発表と思われる。  
[Trade fair for embedded technologies | embedded world](https://www.embedded-world.de/en)  

今回追加されたR1305GはTDP 8-10W、R1102GはTDP (SDP[^1]) 6Wと、それまでのRyzen Embedded R1000シリーズの下限12Wより低く、  
これにより、省電力が求められ、メモリは比較的少容量なシステムにおけるコストを削減できるとし、ファンレスシステムの設計も容易となる。  
これら2つのプロセッサーは3月末より注文可能になる予定とのこと。  

[^1]: Scenario Dissipation Power の略。特定のアプリケーションを実行するシナリオにおいての消費電力を表す。  

AMDはZenアーキテクチャで、前世代の小規模x86 CPUアーキテクチャ Jaguarと同程度の低消費電力までカバーできるとしていたが、  
今回の R1305G、R1102G の正式発表により、実に3年半かけてそれが果たされた。  

 > まずOverallであるが、ZenはJaguarと同程度の低い消費電力から、Excavatorにかなり近いところまでカバーするScalableなcoreであり、Excavatorと比較して、同じ消費電力であればIPCがおおむね40%高いとしている(Photo01)。

 > 引用元: <cite>[Zenアーキテクチャの概要が明らかに - 分岐予測やCache構造の強化で40%のIPC改善を実現 (1) モバイルからサーバまでカバーするスケーラブルな構成 | マイナビニュース](https://news.mynavi.jp/article/20160830-zen/)</cite>

もっと早くにZenでTDP 10W以下、Intel Atom対抗を出してよかったようにも思う。  
そういったローエンド帯、組み込み向けよりも、まずはデスクトップ向けやサーバー向けに力を入れるべき、といった判断や、  
モバイル向けにも7nm製品 *Renoir* を出したため、GF 14nm製品でローエンド帯、組み込み向けを攻める余裕が出てきた、とかがあったのかもしれないが。  

とにもかくも、これで28nm世代のx86 CPUアーキテクチャ、Jaguar/+、Excavator、Steamrollerも安心してZenにバトンを渡せるようになったのではないかと思う。  
搭載製品が新たに出てくることはあるかもしれないが[^3]、SoCのラインナップに28nm世代が追加されることももうないはずだ。  
Chromebook向けは現状A4-9120C、A6-9220Cしかないが、Chronium OSへのパッチからRyzen、Athlonの具体的なSKU名が明らかになっている。  
そう遠くない内に実際に搭載した製品が出てくれることだろう。  

[^3]: [SAPPHIRE Announces New Family of Compact AMD Ryzen™ Embedded Processor Based Motherboards at Embedded World 2020.](https://www.sapphiretech.com/en/news/embeddedworldmb)  

## 製品仕様

| R1000 | R1305G | R1102G |
| :--- | :---: | :---: |
| TDP | 8-10W | 6W (SDP) |
| CPU Core/Thread | 2/4 | 2/2 |
| CPU Base Freq | 1.5 GHz | 1.2 GHz |
| 1T CPU Boost Freq | 2.8 GHz | 2.6 GHz |
| GPU CU | 3 | 3 |
| Max GPU Freq | 1.0 GHz | 1.0 GHz |

### 特筆点
#### Raven2
2つともソケット /パッケージは他のモバイル向け、組み込み向け同様のFP5。  
また、"Zen" CPU とあることから、ダイも他のR1000シリーズ製品同様 *Raven2* とされる。  

 > Built on “Zen” CPU and Radeon™ “Vega” graphics cores, 

 > 引用元: <cite>[New AMD Processors Drive High-Performance Computing for Embedded Industry | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/new-amd-processors-drive-high-performance-computing-embedded)</cite>

*Raven2* と言っても、その中で *Raven2* / *Dali* / *Pollock* のどれにあたるかは不明だが、組み込み向けに割り当てられることが多い RevisionID: 0x9\* を、*Pollock* が2つ持っているため、  
*Pollock* である可能性が高いように**思う**。  
しかし、Chromebookにおいての *Dali* 、*Pollock* の違いは前者がFP5ソケット、後者がFT5ソケットとはっきり別れていたが、  
今回追加された2つはどちらもFP5ソケットであり、そう簡単には行かない。  
さらに *Dali* でも6WのSKUは予定されており[^4]、*Pollock* が *Raven2* を特別省電力に振り切ったバージョンという訳でもない。  
もっと他に違いがあるはずだが、まだはっきりとしない。  
これに関しては引き続き情報を追いたい。  

[^4]: [[Dali] Raven 2 detection Patch ](https://lists.freedesktop.org/archives/amd-gfx/2020-February/045579.html)

#### 仕様に関して
AMD公式サイトの仕様を見るとR1305GのみECC対応の記載がないが、  
[Embedded Processor Specifications | AMD](https://www.amd.com/en/products/specifications/embedded)  
それは不自然であるし、R1305Gを採用している Kontron D3713-V/R mITX のデータシートを見ると ECC supported とあるため、単なるAMDの記載漏れだろう。<span class="hide">いつものこと。</span>  
[D3713-V/R mITX | Kontron](https://www.kontron.com/products/boards-and-standard-form-factors/motherboards/mini-itx/d3713-v-r-mitx.html)  
(<https://www.kontron.com/downloads/datasheets/d/d3713-mitx.pdf?product=158681>)  

R1102Gは他のR1000シリーズ製品とは違い、SMT無効、最大画面出力は2、メモリはシングルチャネル、PCIe Gen3 4-Laneまでとなっている。（Ethernet、USB、SATAの仕様は他のR1000シリーズ製品同様。）  
Raven2でDDR4シングルチャネルというと、最近出てきたFT5ソケット[^2]を思わせるが、Zen APUでTDP (SDP) 6Wまで抑えるにはコア/スレッド数、クロックの調整だけでは足りない、ということなのかもしれない。  

[^2]: [Chromebookに搭載されるPollockはFT5ソケット ――DDR4シングルチャネル、SATA無し | Coelacanth's Dream](/posts/2020/02/12/amd-pollock-ft5/)
