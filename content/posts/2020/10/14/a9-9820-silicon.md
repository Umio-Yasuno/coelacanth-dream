---
title: "A9-9820 は Xbox One SoC と同一ダイ"
date: 2020-10-14T22:53:14+09:00
draft: false
tags: [ "Cato", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

謎の APU **A9-9820** を搭載したボードが AliExpress にて出品されているのを [188号 (@momomo_us) / Twitter](https://twitter.com/momomo_us) 氏が発見、Twitter に投稿した。[^tw]  
{{< link >}} <https://ja.aliexpress.com/item/1005001340237782.html> {{< /link >}}

[^tw]: <https://twitter.com/momomo_us/status/1316352583338917890>

## A9-9820 の正体
製品画像には **A9-9820** のパッケージ、ダイの外観が映っており、まず **A9-9820** について以下のことが分かる。  

 * **A9-9820 (Cato)** は **Xbox One SoC (Durango)** と同一のシリコンを用いる
 * **A9-9820** は APU であり、GPU部は無効化されていない

### Xbox One SoC と同一ダイ

１つ目は、**Xbox One SoC** パッケージ、ダイの外観は HotChips 25 (2013) で発表された際のスライドに掲載されているが、それと **A9-9820** とを見比べるとパッケージ上のチップコンデンサ数と配置が全く同じである。パッケージに対してのダイサイズも同じとなる。[^hc25-xbox-one]  
以前、**A9-9820** の推測にて巨大な ESRAM 32MB がゲーム機以外の通常用途においては無駄となるため、転用が可能かどうか疑問に思うと書いた。  
実際に [Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/) 氏が撮影、パブリック・ドメインで公開している **Xbox One SoC** のダイショットを見ると、ダイの 2/5 程が ESRAM 32MB を占めている。[^xbox-one-dieshot]  
だが結果は (恐らく) ESRAM 32MB を無効化しての転用であった。  
再設計も特に行なっていないということは *Xbox One* 向けに生産して余ったダイなのだろうか？  

ESRAM の無効化が可能かどうかについては、HotChips 25 (2013) のスライドには SoC 内部ブロックとバスを描いたダイアグラムがあり、それによると DDR3 4ch のコントローラーとは別のバスで接続されている (メモリキャッシュではない) ため可能だろう。  
また、**A9-9820** 搭載ボードには DDR3 メモリスロット 4基が実装されているが、DDR3 2ch か DDR3 4ch かは不明。配置からして 4ch のように思えはするが。  

[^xbox-one-dieshot]: [AMD@28nm@Jaguar@Durango@XBox_One@X877045_001_DG3000FEG84HR… | Flickr](https://www.flickr.com/photos/130561288@N04/31376514813/in/album-72157715578309233/)
[^hc25-xbox-one]: [PowerPoint Presentation - HC25.26.121-fixed- XB1 20130826gnn.pdf](https://www.hotchips.org/wp-content/uploads/hc_archives/hc25/HC25.10-SoC1-epub/HC25.26.121-fixed-%20XB1%2020130826gnn.pdf) (Page3)

### R7 350 の謎

**A9-9820** を搭載するボードは GPU が **R7 350** とされており、この製品名は過去 dGPU に使われていた。[^gigabyte-r7-350]  
そのため、以前は APU の GPU部を無効化して dGPU に **R7 350** を搭載していると考えたのだが、ボードにそれらしきものはない。  
製品画像のボード右側にはヒートシンクで隠されたチップがあるが、USB等のフロントI/O の配線がそこから出されているため、外部チップセットと思われる。専用 VRAM も実装されていない。  
よって、**A9-9820** の GPU部は無効化されておらず、**R7 350** は GPU部に付けられた名前と考えられる。  
[先日](/posts/2020/10/11/llvm-add-gfx6_8-gpu/) LLVM に追加された GPUID *gfx705* はやはり *Cato APU* に向けたものだったのかもしれない。  

次に何故 APU の GPU部に dGPU に付けた製品名を使ったかについてを考えると、GIGABYTE を出している **R7 350** は 28nmプロセス、970MHz(標準: 925MHz)、DDR3-1600 128-bit というスペックであり、  
対し **Xbox One SoC** は 28nmプロセス(TSMC 28nm HP)、853MHz、DDR3-2133 256-bit となる。  
**R7 350** の CU数、ROP数が記載されていないが、TechPowerUp GPU Database にある GDDR5版には 8CU(512SP)、16ROP とある。[^tpu-r7-350]  
そして **Xbox One SoC** は、ダイ自体は 14CU(896SP)、16ROP を持つ。  

**A9-9820** の最大GPUクロックだけは判明しており、CHUWI の AeroBox 製品ページによると最大 985MHz で動作する。[^chuwi-aerobox]  

[^gigabyte-r7-350]: [GV-R735OC-2GI (rev. 1.0) スペック | グラフィックスカード - GIGABYTE Japan](https://www.gigabyte.com/jp/Graphics-Card/GV-R735OC-2GI-rev-10/sp#sp)
[^tpu-r7-350]: [AMD Radeon R7 350 Specs | TechPowerUp GPU Database](https://www.techpowerup.com/gpu-specs/radeon-r7-350.c3135)

**A9-9820** の仕様詳細が分かるまではっきりとはしないが、GPUクロックが **R7 350** の 970MHz に近いため、後は有効 CU数を 8CU とする等の調整を行なえば **R7 350** 同等の性能に設定することは可能だろう。  
これが **R7 350** の名を使った理由と考える。自分には他のうまい理由が思いつかない。  

[^chuwi-aerobox]: [Chuwi AeroBox](https://www.chuwi.com/jp/product/items/Chuwi-AeroBox.html)

外部チップセットについては、**Xbox One** もシステムとしては HDMI入力、SATA、USB 等を出すサウスブリッジを搭載していたが、それも一緒に転用しているかどうかはやっぱり不明。  
既に **Xbox One** で検証が行なわれているのだから、使い回すのが妥当なように思う。  

まあ一番の謎は何で **Xbox One SoC** を転用した *Cato APU* が今になって登場したかなのだが、そうした予測不可能な部分こそが中国市場独特の魅力であると言えるのかもしれない。  
