---
title: "AMD、RX 6500 XT 8GB の仕様、4GBモデルとの性能比較を公開"
date: 2022-07-31T16:59:32+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "Beige_Goby", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日、2022/07/26 に Sapphire Technology より発表され、AMD 非公式かとも言われていた従来の 4GBモデルからメモリ容量を増やした **RX 6500 XT** の 8GBモデルだが、少し遅れて AMD 公式サイトの製品ページに 8GBモデルの仕様が掲載された。  
最近では **RX 6700 (non-XT) 10GB** も Sapphire Technology が先に製品を発表し、後に AMD が公式ページに掲載するという流れだった気がする。  

 * [AMD Radeon™ RX 6500 XT Graphics Card | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6500-xt)

公式モデルの仕様では、4GBモデルと 8GBモデルの間にはメモリ容量と TBP (Typical Board Power) 以外に違いは無く、*Navi24/Beige Goby* ベース、有効 CU 数、ROP 数、動作クロック、メモリバス幅、実効メモリ速度等は同じ。  
TBP は 4GBモデルが 107W なのに対し、8GBモデルは 113W となっている。  
GDDR6メモリを両面実装 (Clamshell Mode) することでメモリ容量を 8GB に増やしたと見られ、TBP +6W が追加したメモリチップの分なのだろう。  

英語版のページでは、ゲームを 1080p High Setteings で実行した時の性能比較が掲載されている。  
ゲームは 8種類あり、すべてで 8GBモデルの性能が勝っている。  
差が数fps のゲームもあれば、Forza Horizon 5 +24fps、DOOM Eternal +36fps、F1 2021 +54fps と大きな差が出たゲームもある。  

AMD は以前、2020/05/06 付に今日のゲームは解像度 1080p 設定であっても VRAM 4GB は十分なメモリ容量ではない、と主張するブログ記事を AMD Community 上で公開している。  
当時は競合の同価格帯モデルが最大 4GB であったため、8GBモデルも用意されている **RX 5500 XT** の優位点をアピールする意味もあったのだろうが、それを抜きにしてもやはりゲーミング用途において VRAM 4GB というのは不足気味だったということなのだろう。  

 * [Game Beyond 4GB - AMD Community](https://community.amd.com/t5/gaming/game-beyond-4gb/ba-p/414776)

{{< ref >}}
 * [AMD非公式？メモリ8GB搭載のRadeon RX 6500 XTがSapphireから - PC Watch](https://pc.watch.impress.co.jp/docs/news/1427540.html)
 * [PULSE AMD Radeon RX 6500 XT](https://www.sapphiretech.com/en/consumer/pulse-radeon-rx-6500-xt-8g-gddr6)
 * [Game Beyond 4GB - AMD Community](https://community.amd.com/t5/gaming/game-beyond-4gb/ba-p/414776)
 * [Where to Buy Go Beyond 4GB | AMD](https://www.amd.com/en/where-to-buy/go-beyond-4gb)
{{< /ref >}}
