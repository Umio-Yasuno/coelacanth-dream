---
title: "AMD 4700S の正体は GPU部を無効化したゲーム機向け APU だったか"
date: 2021-04-26T21:58:29+09:00
draft: false
tags: [ "Zen_2", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

先日 Geekbench 5 に出現した謎の CPU **AMD 4700S** だが、搭載したシステムの製品ページからメインメモリに GDDR6 16GB を採用していることが判明し、コンソールゲーム機向け {{< comple >}} Xbox Series X|S, PS5 {{< /comple >}} の SoC/APU を転用したものである可能性が出てきた。  
{{< link >}} [AMD 4700S CPU 出現 | Coelacanth's Dream](/posts/2021/04/22/amd-4700s/) {{< /link >}}

## 無効化されている GPU部 {#disabled-gpu}

下記がオンラインショップ *天猫Tmall* にある **AMD 4700S** を搭載したシステムの製品ページであり、自分は [188号 (@momomo_us) / Twitter](https://twitter.com/momomo_us) 氏のアカウント経由で知った。[^tw-4700s-system]  

 * [【新品预售】AMD 4700S独显ITX高配吃鸡组装机游戏办公直播编程设计CNC迷你高端台式电脑主机-tmall.com天猫](https://detail.tmall.com/item.htm?id=643602411425&sku_properties=5919063:6536025)

製品ページには、**AMD 4700S** が 7nmプロセスで製造され CPU は 8-Core/16-Thread、最大動作クロックは 4.0 GHz、キャッシュ容量は 12 MB {{< comple >}} 恐らく 8x 512 KB + 2x 4 MB と思われる {{< /comple >}}構成であること、  
メインメモリに GDDR6 16GB を採用すること、Discrete GPU として **Radeon RX 550 (GDDR5 2GB 64-bit)** を搭載していることが特徴に挙げられている。  

[^tw-4700s-system]: <https://twitter.com/momomo_us/status/1386631178304638979>

製造プロセス、CPU、そして GDDR6メモリという点から、コンソールゲーム機向けの SoC/APU を転用したものが **AMD 4700S** だと考えられる。  
単純に、既に *Zen 2 アーキテクチャ* 8-Core を搭載する *Renoir/Lucienne APU* 、*Zen 3 アーキテクチャ* 8-Core の *Cezanne APU* がある中、わざわざ GDDR6メモリに対応させた SoC/APU を新たに起こすとは考えにくい。  
チップレットアーキテクチャを採り、既存の CCD と新設計した IOD、それと新パッケージで実現している可能性も考えられるには考えられるが、それは深く考えすぎである気がするし、やはり上と同様の理由が適用できる。  

ゲーム機向け SoC/APU を転用した前例には、*Xbox One SoC* を転用した **A9-9820 (Cato) APU** が存在し、**AMD 4700S** も同様のパターンではないかと思われる。突然中国市場に登場する点も共通している。  
{{< link >}} [A9-9820 は Xbox One SoC と同一ダイ | Coelacanth's Dream](/posts/2020/10/14/a9-9820-silicon/) {{< /link >}}

**A9-9820** と異なるのは、**A9-9820** が DDR3メモリに対応しており、メモリスロットがボード上に実装され、ユーザーがメモリを拡張することが可能だったが、**AMD 4700S** は GDDR6メモリで、メモリがオンボード上に実装されているため、ユーザーによる拡張は不可能なところだ。それでも 16GB 搭載であるから、普段の用途においては十分だろう。  
そして **A9-9820** はフルスペックではないが GPU が有効化されており、APU として確かな要素を持っていたのに対し、**RX 550** というエントリー向け dGPU をわざわざ搭載しているあたり、**AMD 4700S** の GPU部は無効化されていると考えられる。  
そもそも現在半導体の製造キャパシティが逼迫し供給不足が叫ばれている中で、特に需要がありそうなゲーム機向け SoC/APU の転用が可能なのは、CPU部に問題はないが GPU部に不良があり、ゲーム機に使うことができなかったから、というシナリオが想像できる。  

## メモリ構成はどうなっているか {#memory-config}

ゲーム機向け SoC/APU には、*Xbox Series X|S* 、*PS5* の 3種があるが、どれもメモリ構成は異なり、**AMD 4700S** がどれを選んでいるかははっきりしない。  

3種の SoC/APU は、7nmプロセスで製造され、CPU に *Zen 2 アーキテクチャ* 8-Core/16-Thread、メモリ GDDR6 という点は共通しており、GPU が大きな違いの 1つと言えるのだが、**AMD 4700S** は GPU部が無効化されており、3種のどれも **AMD 4700S** と搭載システムの仕様を満たすことができる。  

*Xbox Series X|S* は特異なメモリアーキテクチャを採っており、容量の違うメモリチップを搭載していたり、一部メモリチップを 32-bit ではなく 16-bit で接続している。  
**Xbox Series X SoC** (以下 XSX SoC) はメモリバス幅 320-bit を持ち、製品的には 6x 16Gb(2GB) + 4x 8Gb(1GB) という容量違いのメモリを混載して GDDR6 16GB のメモリ仕様を採っている。  
**Xbox Series S Soc** (以下 XSS SoC) はメモリバス幅 128-bit を持ち、はっきりとメモリ仕様を記載した資料を見付けられなかったため推測だが、5x 16Gb(2GB) (2x 16-bit + 3x 32-bit = 128-bit) という構成でメモリ仕様 GDDR6 10GB を満たしているものと思われる。  
先 2種に対して **PS5 SoC** のメモリアーキテクチャはシンプルで、メモリバス幅 256-bit、8x 16Gb(2GB)、メモリ仕様は GDDR6 16GB となる。

*XSX SoC*, *PS5 SoC* は製品でも GDDR6 16GB という仕様であるから、そのまま **AMD 4700S** に適用できる。だが *XSX* は特異なメモリアーキテクチャであり、メモリチップ 10枚で GDDR6 16GB としているが、部品点数やコストを減らす目的で **AMD 4700S** では *PS5* 同様に 8x 16Gb(2GB) (256-bit) というメモリ構成を採ることも考えられる。  
*XSS SoC* については、8x 16Gb(2GB) をそれぞれ 16-bit で接続すればメモリ仕様を達成できる。**AMD 4700S** のピークメモリ帯域は記載されていないため、可能性としては一応ある。  

製品ページにある画像の 1つには *XSX* の画像が使われており、それが **AMD 4700S** の素性に沿ったものであれば *XSX SoC* を転用していると思われるが、やはりメモリ構成が *XSX* とは異なる可能性が考えられる。  

また、それら 3種の SoC/APU は PCIe Gen4 に対応しているが、**AMD 4700S** 搭載システムの dGPU用 PCIeスロットが Gen4 に対応しているか、レーン数がどうなっているかは不明。  

*Zen 2 アーキテクチャ* を採用する CPU は既にあり、**AMD 4700S** の特徴であり魅力は GDDR6メモリをメインメモリに搭載していることだと言える。  
そういう訳で、**AMD 4700S** の素性とメモリ構成は一層関心が持たれる。  
それと実際に手に入れられるかはともかく、別用途向けの SoC/APU を転用したシステムがどれだけ出回るかは個人的に気になるところだ。  

{{< ref >}}
 * [ゲーム機は先端半導体の宝庫！ 「PS5」「Xbox」のチップを比較する：この10年で起こったこと、次の10年で起こること（51）（1/3 ページ） - EE Times Japan](https://eetimes.jp/ee/articles/2104/05/news012.html)
 * [Xbox Series S | Xbox](https://www.xbox.com/en-US/consoles/xbox-series-s#target-specs)
 * [HotChips2020_GPU_Microsoft_Jeff_Andrews_v2.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Microsoft_Jeff_Andrews_v2.pdf)
{{< /ref >}}

