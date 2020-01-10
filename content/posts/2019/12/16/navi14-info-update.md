---
title: "Navi14の情報アップデート"
date: 2019-12-16T12:17:11+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

まずは訂正から。  
以前からRadeon Pro 5300MをWKS Pro-XLM、Radeon Pro 5500MをWKS Pro-XTMとし、  
残るはXL、WKS Pro-XLMだけだ、とか言っていたが、これは間違いだった。  

NotebookCheck.netのレビュー記事より、Radeon Pro 5500Mが DeviceID:7340、RevisionID:40 と判明した。  
[Apple MacBook Pro 16 2019 Laptop Review: A convincing Core i9-9880H and Radeon Pro 5500M powered multimedia laptop](https://www.notebookcheck.net/Apple-MacBook-Pro-16-2019-Laptop-Review-A-convincing-Core-i9-9880H-and-Radeon-Pro-5500M-powered-multimedia-laptop.445902.0.html)  
<https://www.notebookcheck.net/fileadmin/Notebooks/Apple/MacBook_Pro_16_2019_i7_5500M/gpuz_mbp16_5500m.gif>  

ファームウェアを読み込むコードにて、（DID:7340かつRID:00以外）でない場合にWKSのファームウェアを読み込むようになっている。  
もう少し簡単に書くと、DID:7340以外かRID:00の場合に読み込む。  
そして、DID:7340のRadeon Pro 5500MはWKSではない、となる。  

ちなみに、通常のとWKSで何が違うのかは、CP（Command Processor?）の挙動が若干変更されている、かもしれない程度しかわからない。  

<br>
そういうことで、AMDはApple向け（専用?）とは別にNavi14を使った製品を3つ *くらい* 用意しているとなる。  
判明してる中では、XL、XLM、XTMとデスクトップ向け1つ、モバイル向けが2つあり、  
Navi14のスペックからして、  
Polaris10を使ったRadeon Pro （[WX 5100](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-5100#product-specs)/[WX 7100](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-7100#product-specs)） やPolaris11の[WX 4100](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-4100#product-specs)あたりの後継として登場するかもしれない。  
モバイル向けのPolaris10を使ったRadeon Proには[WX 7100 (Mobile)](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-7100-mobile)があり、そこにNavi14 XTMが当てはまり**そう**な感じ。  
Polaris11では [WX 4170 (Mobile)](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-4170-mobile)だろうか。  

SKU名こそ不明だが、DID:7343、RID:00も確認されているため、他にもRadeon Proとして登場するかもしれない。バランスを考えるとデスクトップ向けのXTかXTXだろうか。  
それより下の性能にはわざわざNavi14で出すまでもなく、Polaris12続投で十分だろう。  

以下はNavi14を使った製品それぞれの仕様等をまとめたもの。  
RX 5300MのTGPはまだはっきりと出ていないが、他の<span style="color:coral">RX 5500シリーズ</span>でのページで見られた

	Additional power connector:1 x 8-pin

の記述がなかったため、ひとまず、補助電源が必要とされない75Wとした。  
実際にはもう少し低い、55〜65Wあたりではないかと思う。  

| <span style="color:darkorange">Navi14 Product</span> | <span style="color:coral">RX 5300M</span> | <span style="color:skyblue">Pro 5300M</span> | <span style="color:coral">RX 5500M</span> | <span style="color:skyblue">Pro 5500M</span> | <span style="color:coral">RX 5500</span> | <span style="color:coral">RX 5500 XT</span>
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| WGPs | 11 | 10 | 11 | 12 | 11 | 11 |
| CUs | 22 | 20 | 22 | 24 | 22 | 22 |
| SPs | 1408 | 1280 | 1408 | 1536 | 1408 | 1408 |
| TMUs | 88 | 80 | 88 | 96 | 88 | 88 |
| ROPs | 32 | 32 | 32 | 32 | 32 | 32 |
||
| Game Clock (MHz) | 1181 | ? | 1448 | ? | 1670 | 1717 |
| Boost Clock (MHz) | 1445 | 1250 | 1645 | 1300 | 1845 | 1845 |
||
| Max Memory Size (GB) | 3 | 4 | 4 | 8 | 4 | 8 |
| Memory Interface (bit) | 96 | 128 | 128 | 128 | 128 | 128 |
| Memory Speed (Gbps) | 14 | 12 | 14 | 12 | 14 | 14 |
| Memory Bandwidth (GB/s) | 168 | 192 | 224 | 192 | 224 | 224 |
| <span style="color:darkorange">Navi14 Product</span> | <span style="color:coral">RX 5300M</span> | <span style="color:skyblue">Pro 5300M</span> | <span style="color:coral">RX 5500M</span> | <span style="color:skyblue">Pro 5500M</span> | <span style="color:coral">RX 5500</span> | <span style="color:coral">RX 5500 XT</span>
| Peak Texture Fill-Rate (GT/s) | 127.2 | 100.0 | 144.76 | 114.4 | 162.36 | 162.36 |
| Peak Pixel Fill-Rate (GP/s) | 46.2 | 40 | 52.60 | 41.6 | 59.00 | 59.00 |
| FP16 (TFLOPS) | 8.14 | 6.4 | 9.26 | 8.0 | 10.4 | 10.4 |
| FP32 (TFLOPS) | 4.07 | 3.2 | 4.63 | 4.0 | 5.2 | 5.2 |
||
| TGP (W) | ~75? | 50 | 85 | 50 | 110 | 130 |

| <span style="color:darkorange">Navi14 SKU</span> | DID | RID | Product |
| :--- | :---: | :---: | :---: |
| Apple? | 7340 | 40 | <span style="color:skyblue">Radeon Pro 5500M</span> |
| | | 41 | |
| | | 43 | <span style="color:skyblue">Radeon Pro 5300M</span>[^1] |
| | | 47 | |
| Gaming XTM | | C1 | <span style="color:coral">Radeon RX 5500M</span> |
| Gaming XLM | | C3 | <span style="color:coral">Radeon RX 5300M</span> |
| Gaming XTX | | C5 | <span style="color:coral">Radeon RX 5500 XT</span> |
| Gaming XT | | C7 | <span style="color:coral">Radeon RX 5500</span> |
| XL? | | CF | <span style="color:coral">Radeon RX 5300?</span> |
| WKS Pro-XL | 7341 | 00 | |
| | 7343 | 00 | |
| WKS Pro-XTM | 7347 | 00 | |
| WKS Pro-XL | 734F | 00 | |

[^1]: <span style="color:skyblue">Radeon Pro 5300M</span>はレビュー記事が見つからなかったため、Apple Boot Camp Software Graphics Drivers内部のファイルから自分が探したもの。そのため信頼度としてはびみょう。  

<hr>
<code>過去Navi14まとめ</code>  

 * [Navi14 Matome](/posts/2019/11/04/navi14-matome/)  
 * [Navi14 Matome Part2](/posts/2019/11/13/navi14-matome-part2/)  
 * [Navi14 Matome Part3](/posts/2019/11/14/navi14-matome-part3/)  
 * [Navi14のちょっとしたまとめ](/posts/2019/12/06/navi14-a-little-matome/)  
