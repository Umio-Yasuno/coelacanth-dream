---
title: "Navi10のダイ観察 & 推測"
date: 2020-01-22T15:36:49+09:00
draft: false
tags: [ "Navi10", "GFX10" ]
keywords: [ "", ]
categories: [ "Hardware", "GPU" ]
noindex: false
---

先日、[Fritzchens Fritz - Flickr](https://www.flickr.com/photos/130561288@N04/)氏が、氏自ら撮影したRX 5700 XT――Navi10――のダイショットを公開した。  
[AMD@7nm@RDNA_1th_gen@Navi10@Radeon_RX_5700_XT@215-0917210@… | Flickr](https://www.flickr.com/photos/130561288@N04/49411586768/in/photostream/)  
その画像に境界がわかりやすくなるよう補正を施し、そこからダイアグラムを推測し、画像に書き出してみた。  
――用途があるとは思えないが――作成した画像もパブリック・ドメインに準ずる。  

作成には、Sashlecat氏のダイアグラムを大いに参考とした。  
[Diagram - Navi 10 | Sashleycat Stuff](https://www.sashleycat.com/diagram-navi-10)  
氏が詳細な解説をしているため、自分がここに書くことはあんまりない。  
ダイアグラムが氏のものと若干違っているが、別段自分の方が正しいと主張する気はなく、示せるだけの根拠も持ち合わせていない。  

{{% figure src="/image/2020/01/22/navi10-dieshot-guess.webp" title="Navi10推測" %}}

サイズとの兼ね合いで非常に見辛くなってしまったが、RBEとHUBの間にある黄緑（lime）色のブロックは2xL1$(128KB)と推測している。  
（リポジトリサイズ節約のために1MB未満に抑えたかった。）  

あるはずだけどイマイチわからないのは、

 * USB-Cコントローラー
 * 6x Display PHY(DCN 2.0)
 * VCN 2.0

Display PHYは左中付近にある大きめのブロックが怪しい。DSCエンコーダーも含まれるため、この大きさであってもおかしくない。  
が、確信はない。  
Navi14は5x Display PHYだから、Navi14のダイショットが来ればもう少しはっきりするかもしれない。  
そしてそれはFritzchens Fritz氏次第なため、騒がずにおとなしく待つこととする。  

VCN 2.0はバッファらしきSRAMが固まっている、Display Controller（仮）下部付近が怪しい。  
後気になるのはRasterize/PrimUnitの隅にある、謎のブロックだろうか。  

それと、やっぱりという感じではあるがInfinity Fabric用らしきPHYはなかった。  
グラフィック向けならAPI側で分割してmGPUにするだけで十分だろうから、あってもロマン重視となりそうだが。  
<span class="hide">でもちょっと残念。</span>

{{% figure src="/image/2020/01/22/navi10-wgp-guess.webp" title="WGP内部推測" %}}

#### おまけ
AMDがスライド内で使用してきたダイショット。  
![](/image/2020/01/22/navi10-dieshot-official.webp)  

一部ブロックが違うが、恐らくどの層をピックアップするかで見えてくるものが変わってくるのだろう。  
<span class="hide">公式のダイショットは信用できないという訳ではない。念の為。</span>
