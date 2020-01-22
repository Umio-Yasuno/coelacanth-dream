---
title: "RX 5600 XTのレビューが解禁"
date: 2020-01-22T05:24:55+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "Radeon RX 5600" ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

RX 5600 XTのレビューが世界的に解禁され、同時に国内発売日と価格も判明した。  

そして仕様を変更するのではないかと勘ぐったが、新たな仕様のvBIOSを許可するという形で、実装はベンダーに委ねられているようだ。  
冷却デザインや電源設計の都合上、そうすべてに適用はできないのだろう。  
新vBIOSだとLinuxでうまく動作しない不具合があるらしく、デフォルトの仕様とならなかったのはありがたいと言えるかもしれない。  

新たな仕様は各社の様子を見るに、Boost Clockが1750 MHz、メモリは14 Gbpsあたりと思われる。  
メモリは12 Gbps品をオーバークロックするとかではなく、元々14 Gbps品を搭載していたが（多分差別化とかの理由で）12Gbpsに制限していたらしい。  
[Sapphire Radeon RX 5600 XT Pulse Review #CircuitBoardPCBAnalysis | TechPowerUp](https://www.techpowerup.com/review/sapphire-radeon-rx-5600-xt-pulse/3.html#CircuitBoardPCBAnalysis)  
MicronのGDDR6チップのラインナップには12Gbps品があるはずだが、それを採用しないのは他のカード用と一緒に14Gbps品を仕入れた方が安く付く、とかなんだろうか。  
[MT61K256M32JE-12 - micron.com](https://www.micron.com/products/graphics-memory/gddr6/part-catalog/mt61k256m32je-12)  
GTX1660Tiでは12Gbps品を採用していたことから、流通しておらず14Gbpsしかない、という訳ではないはずだ。  

PowerColor は2つのvBIOSをそれぞれ切り替えられるモデルには新仕様をOC Modeとして、元の仕様をSilent Modeとし、vBIOSが1つだけのモデルは元の仕様のままとなっており、  
[Unleash Your RX 5600 XT Red Devil & Red Dragon with update vBIOS](https://www.powercolor.com/new?id=1579588862)  
SAPPHIREもそのような実装だ。  
[SAPPHIRE PULSE RX 5600 XT 6G GDDR6 - sapphiretech.com](https://www.sapphiretech.com/en/consumer/pulse-radeon-rx-5600-xt-6g-gddr6#Download)

しかしXFXはコアクロックは新仕様でも、メモリは12Gbpsのままになっている。  
[XFX AMD Radeon™ RX 5600 XT 6GB GDDR6 THICC III Pro - XFX-2019](https://www.xfxforce.com/gpus/xfx-amd-radeon-tm-rx-5600-xt-6gb-gddr6-thicc-iii-pro)  

細かい所が各社で違い、それによる性能や消費電力の差は決して小さいものではないため、RX 5600 XT購入の際は事前にスペックを調べた方がいいだろう。  

### 性能とか消費電力
<span class="reference">参考</span>

 * [Sapphire Radeon RX 5600 XT Pulse Review | TechPowerUp](https://www.techpowerup.com/review/sapphire-radeon-rx-5600-xt-pulse/)
 * [AMD Radeon RX 5600 XT Linux Gaming Performance Review - Phoronix](https://www.phoronix.com/scan.php?page=article&item=linux-rx5600xt-amd&num=1)

まず旧vBIOSではAMD公式の発表通り GTX1660Ti を性能で上回り、電力効率でもついに凌いでいる。  
新vBIOSでは急遽レビュワーに配布、適用させただけあってRTX 2060に何とか勝っているが、その分消費電力は大差ないものに。電力効率も相応に下がった。  

他のAMDGPUと比較すると、旧vBIOSではVega56、新vBIOSではVega64に近い性能で約8割も高い電力効率を示す。そしてそれはRX 5500、RX 5700よりも上であり、Navi系最強の電力効率となっている。  
WGP(CU)はほとんど、ROPは全く削られていないというのはやはり素晴らしい。  

懸念していたRX 5700との性能差も思っていた程深刻ではなかった。  
そもそもメモリバス幅と容量で、FullHDとWQHDでターゲット解像度の棲み分けが出来ているのだし、杞憂だったということだろう。  

### 国内発売日と価格
[SAPPHIRE「PULSE RADEON RX 5600 XT」の国内発売日と価格が判明 - エルミタージュ秋葉原](http://www.gdm.or.jp/pressrelease/2020/0121/335794)  
[ASRock、Radeon RX 5600 XT搭載「Phantom Gaming」と「Challenger」順次国内発売](http://www.gdm.or.jp/pressrelease/2020/0121/335795)  
[PowerColor、デュアルファンクーラー搭載RX 5600 XTの詳細スペックと発売日確定](http://www.gdm.or.jp/pressrelease/2020/0121/335787)  

現状判明しているものでは、早くて2020-01-25、そうでないのは2020-01-29。  
ASRockは旧vBIOSの仕様のままだが、PowerColorとSAPPHIREは新vBIOSなあたり、よく間に合ったな、という感想。  
まだ国内発表していないベンダーは間に合わなかったのかもしれないが。  

<br>
それはともかく、一番の問題は**価格**だ。  

 * SAPPHIRE PULSE RADEON RX 5600 XT : **<span class="yen">41,800（税込み）</span>**
 * ASRock Radeon RX 5600 XT Phantom Gaming D3 6G OC : **<span class="yen">42,980（税抜き）</span>**
 * ASRock Radeon RX 5600 XT Phantom Gaming D2 6G OC : **<span class="yen">38,980（税抜き）</span>**
 * ASRock Radeon RX 5600 XT Challenger D 6G OC : **<span class="yen">37,520（税抜き）</span>**
 * PowerColor RED DEVIL RX 5600 XT : **<span class="yen">39,980（税抜き）</span>**
 * PowerColor RED DRAGON RX 5600 XT : **<span class="yen">36,980（税抜き）</span>**

そして、Amazon.co.jpでRX 5700の価格を見ると、  

 * [SAPPHIRE サファイア PULSE RADEON RX 5700 8G GDDR6](https://www.amazon.co.jp/dp/B07WK7ZNSY/) :  **<span class="yen">38,212</span>**
 * [ASRock RX5700 Challenger D8G OC](https://www.amazon.co.jp/dp/B07X8TRWQY/) : **<span class="yen">40,384</span>**
 * [PowerColor AMD Radeon RX5700](https://www.amazon.co.jp/dp/B07WY3TJW3/) : **<span class="yen">41,791</span>**

……？？？？？  
お祝儀価格込みとはいえ下位製品が上位製品より高いってどういうことなの……  
さらにはRTX 2060が安いモデルだと約<span class="yen">35,000</span>とか<span class="yen">39,000</span>で買える。  
これじゃどう見たってRX 5600 XTを早速買おうとはならない。RX 5600 XTを買う予算があるならRX 5700かRTX 2060に行くに決まってる。  
いくら電力効率が優れていると言っても、新しいGPUを購入する人のほとんどは絶対性能とコスパで選ぶだろう。  
電力効率の優秀さで高めの価格設定が許されるのは、カード長も短めなMini-ITX向けモデルくらいだ。  

米Amazon.comでは$279.99の製品がいくつかあり、RX 5700も安い製品で$328.97と価格差もそもの値段もマシだ。  
<span class="hide">買うとしても――リスクを含めても――個人輸入する人がそれなりに出てくるはずだ。</span>  

性能も消費電力も優秀なのに、最後で *価格が落ち着けばね* 、と付いてしまうのは残念と言わざるをえない。  
ほんと、優秀なのに……
