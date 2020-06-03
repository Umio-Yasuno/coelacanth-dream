---
title: "RX 5600情報アップデート & 推測 Part2"
date: 2019-12-22T12:50:50+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "Radeon RX 5600" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

### EECにGIGABYTE、PowerColorのRX 5600 XTが追加
[GIGABYTE preparing Radeon RX 5600 XT EAGLE series - VideoCardz](https://videocardz.com/newz/gigabyte-preparing-radeon-rx-5600-xt-eagle-series)  

[Карточка документа - portal.eaeunion.org](https://portal.eaeunion.org/sites/odata/_layouts/15/Portal.EEC.Registry.UI/DisplayForm.aspx?ItemId=66050&ListId=d84d16d7-2cc9-4cff-a13b-530f96889dbc)  
<code>（情報元: <https://twitter.com/KOMACHI_ENSAKA/status/1208076472885202944>）</code>  

RX 5600 XT 6GBのみであり、8GBモデルの名はない。  
ネット上では、ASUSに限って8GBだけがあり他のベンダーから確認された6GBはないことから、ASUSが間違って登録したのではないか、という見方をする人は少なくない。  
段々私もそう思えてきた。  

### 改めて推測
8GB(256-bit)の分がなくなったため、随分とすっきりした。  

| Navi Product (guess) | <span style="color:#f4a460">RX 5600 6GB?</span> | <span style="color:#f4a460">RX 5600 XT 6GB?</span> |
| :--- | :---: | :---: |
| WGPs | 14 | 16 |
| CUs | 28 | 32 |
| SPs | 1792 | 2048 |
| TMUs | 112 |128 |
| ROPs | 48/56? | 56/64? |
||
| Memory Interface (bit) | 192 | 192 |
| Memory Speed (Gbps) | 14 | 14 |
| Memory Bandwidth (GB/s) | 336 | 336 |

が、まだはっきりしないのはRX 5600シリーズの立ち位置だ。  

#### RX 5600シリーズは誰が為に

これまで出てきたNavi RX 5000シリーズがターゲットとする解像度は、  
RX 5700が1440p、RX 5500が1080pとなっている。  
謳い文句では、 *Ultimate 1440p Gaming Platform* 、だとか *Incredible 1080p Performance* 、 *next level of high-performance 1080p gameplay* が並ぶ。  

[AMD Unleashes Ultimate PC Gaming Platform with Worldwide Availability of AMD Radeon™ RX 5700 Series Graphics Cards and AMD Ryzen™ 3000 Series Desktop Processors - AMD](http://ir.amd.com/news-releases/news-release-details/amd-unleashes-ultimate-pc-gaming-platform-worldwide-availability)  
[The Ultimate 1440p Gaming Platform with AMD Ryzen™ and Radeon™ - Youtube](https://www.youtube-nocookie.com/embed/OBnSTlPja0c)  

[AMD Unveils the AMD Radeon™ RX 5500 XT Graphics Card: Incredible 1080p Performance, Breath-Taking Visuals and Powerful Software Features - AMD](http://ir.amd.com/news-releases/news-release-details/amd-unveils-amd-radeontm-rx-5500-xt-graphics-card-incredible)  
[Introducing the AMD Radeon™ RX 5500 XT Graphics Card - Youtube](https://www.youtube-nocookie.com/embed/J7r6yVA27LY)  

解像度において、RX 5600シリーズの席はない。  

Polaris、Vegaの置き換えだとしても、RX 5700シリーズがVega56/64を、RX 5500シリーズがRX 590を置き換えているため、ここでも居場所がない。  

それでもありえそうなターゲットがあり、1080p 144fpsだ。  

最近流行っている(?)eスポーツではとにかく入力遅延を減らすことが求められる。  
上のRX 5500 XTの動画を観ると、一部のゲームでは144fpsを超えていたりするが、基本1080p 60〜90fpsとなっており、ある程度安定して144fpsというのは難しいだろう。少なくとも144fpsを推せるレベルではない。  
RX 5700は、144fpsが達成可能なはずだが、若干オーバースペックな気は否めないし、AMDだってそう安売りしたくはないだろう。  

そういうことで、1080p 144fpsというポジションならばRX 5600シリーズが丁度よく納まるのはないかと思う。  
価格的にもスペック的にもGTX16xxシリーズと戦うことができ、カジュアルゲーマーからの受けもいいはずだ。  
<span style="color:darkslategray">正直、1080p 144fpsよりも、ただGTX16xxシリーズ対抗を出すという意味の方が強いのではないかと思う。</span>  

上記表の推測スペックで1080p 144fpsが可能なのかという点は、ROP数を信じる、あとはクロック次第、としか言えない。  
ただeスポーツ向けとか言うと、AMDは中国市場向けや中国のネットカフェ向けのモデルを出すことがあるため、そういった方向の製品になってしまわないか不安ではある。  
RX 5500シリーズのように、無印はネットカフェ向けやOEM、XTは自作市場に、といった感じになればまだ嬉しいが。  

登場時期は、CES 2020が近いことからそこで発表するのでは、と言われている。  
ただ同様の理由で、Renoir、Navi12、Navi21も発表するんじゃ、とも言われており、詰め込み過ぎな気がしなくもない。  
RX 5500 XTでは発表から販売まで、だいぶ間が空いてしまったために、NVIDIAからGTX16xx SUPERを先に出されてしまった。  
（推測が当たったとして）Navi10ベースならばそれほど時間を掛けずに出せるはず。Naviの勢いを出すためにも早くに出して欲しいところだ。  

| Navi Product | <span style="color:coral">RX 5500 XT</span> | <span style="color:#f4a460">RX 5600 series (guess)</span> | <span style="color:coral">RX 5700</span> |
| :--- | :---: | :---: | :---: |
| WGPs | 11 | 14/16 ? | 18 |
| CUs | 22 | 28/32 ? | 36 |
| SPs | 1408 | 1792 /2048 ? |2304 |
| TMUs | 88 | 92/128 ? | 144 |
| ROPs | 32 | 48/56 ? | 64 |
||
| Memory Interface (bit) | 128 | 192 | 256 |
| Memory Speed (Gbps) | 14 | 14 | 14 |
| Memory Bandwidth (GB/s) | 224 | 336 |448 |
||
| TGP (W) | 130 | ~150 ? | 180 |
| Launch Price | $169 | ~$269 ? | $379 |

<hr>
<code>RX 5600関連</code>

 * [Radeon RX 5600推測 ](/posts/2019/12/18/radeon-rx-5600-guess/)

