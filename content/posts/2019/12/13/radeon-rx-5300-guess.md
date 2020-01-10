---
title: "RX 5300のスペック推測"
date: 2019-12-13T13:49:50+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

残るNavi14のSKUはXL、WKS Pro-XLとなった。  
前者はRX 5300、後者はRadeon Pro W5300とかの名前で出るのではないかと思う。  
XTの有無は、XLXというSKUは確認されていないため、どっちかだけだと思うがそうなるとXTが付く意味がなくなる。  
そういう訳で「RX 5300」だと個人的には考えているが、それで書き続けるとややこしいため、以下Navi14 XLとする。  

Navi14 XLで確定しているのはGame Clockだけで、他の部分をRX 5300Mのスペックを元に考えると、  
まずコア周りはそのままでRX 5500等と同一、メモリ周りはバス幅を96-bitにするのではないだろうか。  

96-bitならばメモリチップを1枚減らすことができ、それはそのままコストの低下に繋がる。  
性能こそ下がりはするが、RDNAの構成ならばROPを減らさずに済むため、グラフィック性能への影響はある程度抑えられる……というのは以前書いた。
[Navi14のちょっとしたまとめ](/posts/2019/12/06/navi14-a-little-matome/#rop%E3%81%AE%E7%8A%A0%E7%89%B2%E3%81%AA%E3%81%8F%E3%83%A1%E3%83%A2%E3%83%AA%E3%83%90%E3%82%B9%E5%B9%85%E5%89%8A%E6%B8%9B-l2%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E4%B8%80%E9%83%A8%E7%84%A1%E5%8A%B9)  
メモリそのままで性能をCU数で調整するよりはコストは下がり、エントリー向けとして出すならば安い方がユーザーに受けるだろう。  

消費電力がどうなるかは、まずGame Clockが同じRX 5500Mが――イマイチ公式の情報がないが――TDP 85Wとなっており、  
メモリチップ1枚で大きく変わるとも思えないため、Navi14 XLも概ね85Wと思われる。  
基本補助電源1x 6-pin、Boost Clock次第で補助電源無しがあるかどうか。  

現状Radeonの補助電源無しでトップの性能を持つのはRX 560だろうか?  
せっかくなので最大クロックを1080MHzに制限して使っているRX 560を以下の表に加えてみた。  

並べて今更に思ったがここまで差があるとは思わなかった。  
メモリバス幅こそ減ったものの、GDDR6 14Gbpsによって1.5倍の帯域、グラフィック周りは2倍以上、FP16はPacked実行のおかげで約3.7倍の演算能力を期待できる。  
FP16は個人的なもので言うと、[waifu2x-ncnn-vulkan](https://github.com/nihui/waifu2x-ncnn-vulkan)で使ったりするため、結構嬉しいかもしれない。  

RX 5300Mのスペックを元に推測したスペック表  

| Navi14 | <span style="color:tomato">My RX 560</span> | <span style="color:tomato">RX 5300M</span> | Navi14 XL (guess) |
| :--- | :---: | :---: | :---: |
| WGPs | N/A | 11 | 11 |
| CUs | 16 | 22 | 22 |
| SPs | 1024 | 1408 | 1408 |
| TMUs | 64 | 88 | 88 |
| ROPs | 16 | 32 | 32 |
|| 
| Game Clock (MHz) | N/A | 1181 | 1448 |
| Boost Clock (MHz) | 1080 | 1445 | ? |
||
| Max Memory Size (GB) | 4 | 3 | (3/6)? |
| Memory Interface (bit) | 128 | 96 | 96? |
| Memory Speed (Gbps) | 7 | 14 | 14 |
| Memory Bandwidth (GB/s) | 112 | 168 | 168? |
||
| Peak Texture Fill-Rate (GT/s) | 69.12 | 127.20 | ? |
| Peak Pixel Fill-Rate (GP/s) | 17.28 | 46.20 | ? |
| FP16 (TFLOPS) | 2.21 | 8.14 | ? |
| FP32 (TFLOPS) | 2.21 | 4.07 | ? |
| Typical Board Power (W) | <48 | ? | ~85? |

過去Navi14まとめ  
[Navi14 Matome](/posts/2019/11/04/navi14-matome/)  
[Navi14 Matome Part2](/posts/2019/11/13/navi14-matome-part2/)  
[Navi14 Matome Part3](/posts/2019/11/14/navi14-matome-part3/)  
[Navi14のちょっとしたまとめ](/posts/2019/12/06/navi14-a-little-matome/)  
