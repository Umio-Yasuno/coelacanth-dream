---
title: "Navi14を搭載した16-inch MacBook Proがいきなり発表"
date: 2019-11-13T22:57:20+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14", "MacBookPro" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

2019-11-13、AppleはMacBook Pro 16インチモデルを発表した。  
これまでの15インチよりも1インチ大型化し、CPUには最高でIntel Core i9 8-core、Base 2.4GHz/Boost 5.0GHzを、  
GPUにはRadeon Pro 5300M 4GB、Radeon Pro 5500M 4/8GBが用意されている。  

[Navi14 Matome Part2](/posts/2019/11/13/navi14-matome-part2)  
前の記事の表で言うと、5500MがWKS Pro-XTM、5300MがWKS Pro-XLMにあたると思われる。  
[MacBook Pro 16-inch model TechSpecs](https://www.apple.com/macbook-pro-16/specs/)  

 ~~AppleのサイトにもAMD公式サイトにもまだ詳細なスペックが載っていないため、WGP数は不明だ。~~  
OverViewの方に最大24CUと記載されていた。  
同時にメモリ帯域は189GB/sとあり、  
GDDR6メモリクロックは11.8125Gbps、簡単にして12Gbpsとデスクトップ向けが14Gbpsなのに対し控えめだ。  
RDNA 7nmにおいてはGPUクロックを下げるよりもメモリクロックを下げた方が効率が良いということだろうか。  

実性能において、Radeon Pro 5500M 8GBは、  
Radeon Pro 560X、Radeon Pro Vega 20と比べてどの処理も高速と謳われている。  
特にRadeon Pro 560Xに対しては最大2.1倍の性能らしい。  

それと不評だったバタフライキーボードから、それ以前のシザー方式に戻したとか。  

(追記)  
[Customize MacBook Pro - Apple](https://www.apple.com/shop/buy-mac/macbook-pro?product=MVVJ2LL/A&step=config)  
カスタマイズ段階のヘルプ部分から、Radeon Pro 5300Mは20CUと判明。  
(追記2)  
公式サイトにもRadeon Pro 5300M/5500Mが、  
そしてRX 5300Mのスペックも追加された。  
[RX 5300M](https://www.amd.com/en/product/8976)  
FP32性能から、Radeon Pro 5500MのBoostClockは約1300MHz、  
Radeon Pro 5300MのBoostClockは約1250MHzと算出できる。  

RX 5300MのBoostClockは1445MHzで、  
GameClockからしてSKUはXLMと推測される。  
驚くべき点はMax Memory Sizeが3GB、Bandwidthが168GB/sということから96-bit構成になっていることだ。    
Productをまとめた表を作るつもりだが、大きくなると思われるため別記事、[Navi14 Matome Part3](/posts/2019/11/14/navi14-matome-part3)として公開する。  