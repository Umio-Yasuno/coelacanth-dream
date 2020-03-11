---
title: "働き者のPolaris"
date: 2020-03-11T01:31:11+09:00
draft: false
tags: [ "GFX8", ]
keywords: [ "", ]
categories: [ "Hardware", "GPU" ]
noindex: false
---

童話のタイトルっぽい。  

## 14nmのRX 590 GME

先日、AMDはこっそりと中国市場向けに *RX 590 GME* をラインナップに追加した。  
[AMD Radeon RX 590 GPU | AMD 显卡 | AMD](https://www.amd.com/zh-hans/products/graphics/radeon-rx-590#product%20footnotetids_9496-1866-2116-816)  

RX 590 GME はクロックが RX 580 とほぼ変わらず、そしてチップ自体も、 RX 590 の大きな特徴であった12nmプロセスでの生産ではなく、これも RX 580 と同じ14nmプロセスで生産されている。  
AMD公式サイトでのスペックでは無印、GME共に `第三代 14nm 工艺`[^1]となっているため、それがわかりにくいが、Sapphireの製品ページでは RX 590 GME が `14 nm FinFET 制程技术`[^2]、 RX 590 が `12 nm 制程技术`[^3]と記されている。  

それなら RX 580 として出せばいいのではないかとなるが、AMDとベンダーが真新しさを出したかったのだろう。  
それに RX 580 は既に複数バージョンがあり、これ以上は増やすのは避けたい、というのもあるかもしれない。なんたって RX 580 だけで5種類存在する。

 * RX 580
 * RX 580 (OEM)
 * RX 580G
 * RX 580X
 * RX 580 2048SP

内 *RX 580G* 、*RX 580 2048SP* は中国市場限定。  
RX 580X の最大クロックが1340MHz、RX 590 GME は1380MHzであるから、*RX 580 XT* としても良かったかもしれないが、それでは RX 580 の枠内が複雑になってしまうし、中国市場限定というのがわかりにくい。  

[^1]: <cite>[AMD Radeon RX 590 GPU | AMD 显卡 | AMD](https://www.amd.com/zh-hans/products/graphics/radeon-rx-590#product%20footnotetids_9496-1866-2116-816)</cite> <br> 英語版でもそれは同様。<br> [AMD Radeon RX 590 GPU | AMD Graphics Card | AMD](https://www.amd.com/en/products/graphics/radeon-rx-590#product-specs)
[^2]: <cite>[蓝宝石RX590 8G D5 超白金 极光特别版 FO](https://www.sapphiretech.com/zh-cn/consumer/nitro-rx-590-8g-g5-se_c#Specification)</cite>
[^3]: <cite>[蓝宝石RX 590 GME 8G D5 超白金 极光特别版](https://www.sapphiretech.com/zh-cn/consumer/nitro-rx-590-gme-8g-g5-se_c#Specification)</cite>

## 働き者のPolaris
さて、Polaris (GCN 4)アーキテクチャは長生きで、働き者と言える。  

初の Polaris10ベースの製品、RX 480が発売されたのは2016/06/29[^4]。それから2016/08/04に同様の Polaris10ベースの RX 470[^5]、2016/08/16に Polaris11ベースの RX 460[^6]が続いた。  
次の年、2017年にはVega (GCN 5)アーキテクチャが発表されたが、ハイエンド帯をターゲットとしたため Polarisの置き換えとはならず、2017/04/18に RX 480/470/460 のスペックを一部引き上げた RX 580/570/560、そして Polaris12ベースの RX 550が発売された。[^7]  
2018/04/13には特にスペックが変わらない、OEM向けとされる RX 580X/570X/560X/550Xが追加された。[^8]OEM向けにはデスクトップ向けの製品が既にあったが、そうでない自作PC市場向けよりもベースクロックが低かった。X付きはOEM向けを自作PC市場向けに合わせたもの。RX 550Xだけは例外で、ブーストクロックが引き上げられている。  
そして、2018/11/15には Polaris10をそのまま12nmプロセスで生産し、クロックをさらに向上させたRX 590を発表。[^9]  
それから2019年にはOEM向けに Polaris12ベースの RX 640/625/620や、組み込み向けに Polaris10ベースの Embedded Radeon E9560/E9390、ワークステーション向けに Polaris12ベースの Radeon Pro WX 3200 が発表された。自作PC市場向けにはハイエンド帯に Radeon VII が投入されたが、 Polarisがカバーするミドル帯に動きはなかった。  
そして2020/03/09、中国市場向けに RX 590 GME がこっそりと追加される。  

一部市場限定とはいえ、半導体製品において、登場から3年半近く経っても新たな製品が追加されるというのは中々に珍しく、長生きと言えるのではないだろうか。  
OEM向けを見てもわかるようにローエンドからメインストリームまでカバーしているため、AMDとしても使い勝手がいいのだろう。  
ソフトウェアサポートの面でも息が長く、Windows版ドライバ Radeon Software Adrenalin 2020 Editionでの新機能、整数スケーリングやAnti-Lagといったものが Polarisアーキテクチャでもサポートされた。[^10]  

[^4]: <cite>[AMD Launches the Radeon Rebellion With the Radeon(TM) RX 480 Graphics Card, Available Now | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-launches-radeon-rebellion-radeontm-rx-480-graphics-card)</cite>
[^5]: <cite>[The Radeon Rebellion Storms Ahead With the Gamer Optimized Radeon(TM) RX 470 GPU | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/radeon-rebellion-storms-ahead-gamer-optimized-radeontm-rx-470)</cite>
[^6]: <cite>[Introducing Radeon(TM) RX 460 Graphics: Disruptive Technology for All eSports Gamers Around the World | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/introducing-radeontm-rx-460-graphics-disruptive-technology-all)</cite>
[^7]: <cite>[Radeon(TM) RX 500 Series: The Most Compelling Graphics Card Upgrade Yet | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/radeontm-rx-500-series-most-compelling-graphics-card-upgrade-yet)</cite>
[^8]: <cite>[Radeon(TM) RX 500 Series: The Most Compelling Graphics Card Upgrade Yet | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/radeontm-rx-500-series-most-compelling-graphics-card-upgrade-yet)</cite>
[^9]: <cite>[New AMD Radeon™ RX 590 Graphics Cards Deliver Leading-Edge, Smooth HD Gaming Experience for Latest Games | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/new-amd-radeontm-rx-590-graphics-cards-deliver-leading-edge)</cite>
[^10]: <cite>[AMD、Radeon Software Adrenalin 2020 Editionの提供開始 - Boost機能を追加 | マイナビニュース](https://news.mynavi.jp/article/20191210-936136/)</cite>

### 長生きの理由
これには Polaris10/11/12ベースを置き換える製品が、[Navi14](/tags/navi14)ベースの RX 5500 XT まで出なかったこと、  
そしてそのNavi14ベースの製品が中々拡張されないことがあげられる。  

前者は、一時期 Vega (GCN 5)アーキテクチャの製品がミドル帯にも出るのではないかと噂されていたが、結局そう言えるのはApple製品に搭載された[Vega12](/tags/vega12)ベースのGPUだけであり、自作PC市場にPCIeカード型に搭載したものは出なかった。  
これはやはり、1スタック構成でもVRAMにHBM2を採用するのはコスト的に難しかったのだろうか。PCIeカード型では1パッケージに納め、専有面積を減らすことの意義もオンボード実装であるモバイル向けよりは薄くなる。  

そして後者にも関連するが、RX 5500 XTは自作PC市場において RX 590 しか置き換えたとは言えないし、AMDもそのつもりはないと思える。  

{{< figure src="/image/2020/03/11/product-perf-matrix.webp" title="Radeon Product Performance Matrix" caption="画像出典: <cite>[パートナーリソース&情報 | AMDパートナーハブ](https://www.amd.com/ja/partner)</cite>" >}}

RX 580/570等はメインストリームとして RX 5500 XTより下の性能帯に位置する。  
Navi14ベースでは、メモリバス幅を96-bitに減らし、VRAMも3GBとなる製品があるはずだが、現状では RX 5300Mが発表されているのみで、デスクトップ、ワークステーション向けで近い製品はまだ表に出てきていない。  
これは、RX 5500 XTではメモリバス幅が RX 580/570の半分である128-bit幅でも、GDDR6メモリがGDDR5に比べて倍近くの速度(14Gbps)を持ち、16Gbitチップも合わせ、帯域とVRAM容量でも綺麗に置き換えるのが可能だったのに対し、  
96-bit幅では、GDDR6 14Gbpsで 168GB/sと微妙な位置にあり、VRAM容量も3GBと単純に置き換えることができない。メモリベンダが計画しているGDDR6 16Gbpsでは 224GB/s、16GbitでVRAM容量と6GBとすることもできるが、それがその性能帯の製品に見合うとは思えない。  
そのため、少なくともPolaris10ベース製品の在庫が捌けるまで待つ必要があると考えられる。  

また、Navi14の生産コストが高く、RX 5500 XT 4GBの参考価格 $169よりも下げることが難しい可能性もある。  
7nmプロセスノードは16/14nmプロセスノードの2倍近いコストがかかるとされ、

 > 　歩留まりを加味したコストを比較すると16/14nmプロセスノードに対して、7nmノードは2倍近いコストに膨れ上がっていることがわかる。45nmプロセス当時と比較すると4倍のコストだ。言い換えれば、16/14nmプロセスと同じダイサイズのチップの製造でも、7nmでは2倍のコストがかかるということだ。

 > 引用元: <cite>[【後藤弘茂のWeekly海外ニュース】ZEN 2ベースの64コアCPU「Rome」はなぜCPUとI/Oを分離したのか - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1156455.html)</cite>

そして Navi14のダイサイズは約157mm<sup>2</sup>[^11]、Polaris10のダイサイズは約244mm<sup>2</sup>[^12]であるため、単純に計算すれば Navi14 は Polaris10 よりもコストが高い。  
それならば性能で見られることが多いコンシューマ向けではPolarisベースの製品を継続し、消費電力、電力効率が重要なモバイル向けやワークステーション向けに拡張していった方が良い、となる。  
EUV露光技術を導入するプロセスではコストが改善されるが[^13]、Radeon GPUにおけるタイミングは早くとも[RDNA 2](/tags/rdna-2)からとなるため、ミドルやローエンド帯GPU好みの人には、またしばらくPolarisに働き者でいてもらわなければならないかもしれない。  
そう言う自分もNavi14ベースのGPU欲しかったけど RX 560 のまま……  

[^11]: [AMD@7nm@RDNA_1th_gen@Navi14@Radeon_RX_5500_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/49437016132/)
[^12]: [comparison\_\_\_Fiji\_-\_Polaris10\_-\_Vega10\_-\_Polaris22 | Flickr](https://www.flickr.com/photos/130561288@N04/46202429935/)
[^13]: [【福田昭のセミコン業界最前線】EUV露光による先端ロジックと先端DRAMの量産がついにはじまる - PC Watch](https://pc.watch.impress.co.jp/docs/column/semicon/1159880.html)
