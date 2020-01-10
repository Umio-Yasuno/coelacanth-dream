---
title: "Radeon Pro W5700Xが発表される"
date: 2019-12-12T09:41:07+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "Radeon Pro W5700" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

AMDよりRadeon Pro W5700"X"が発表された。  
[AMD Radeon™ Professional GPUs Offer Breakthrough Graphics Performance for Apple Mac Pro](https://www.amd.com/en/press-releases/2019-12-10-amd-radeon-professional-gpus-offer-breakthrough-graphics-performance-for)  

プレリリースにある通り、販売が開始された新Mac Pro向けの製品であり、  ボード形状はMPX。  
PCIe型でも出る可能性はゼロではないが、プレスリリースではやけにMac Proに搭載可能なことを推しているため、自作PC市場に流れることは残念ながら期待できない。  
Mac Proから外されたものが流れてきても、MPXスロットを持つボードがMac Pro以外にないため試すこともできない。  

スペックとしては、Navi10の20WGP（40CU）がフル有効化され、VRAM容量は16GB。  
メモリチップにはGDDR6 16Gb使っていると思われ、Navi10では唯一の16GBを搭載した製品。  
2x16bit接続でメモリバス幅256-bitのまま、16x GDDR6 8Gbで16GBにしている可能性もあるが、  
それだと基板のコストが8x GDDR6 16Gbよりかかるはずで、  
そこまでやるならMPXあたり最大32GBも出してる気がするため、2x16bit接続はないと思われる。  

Two MPXの構成もあり、そちらを選ぶと総VRAM容量は32GBとなる。  
Vega II DuoのようにOne MPXに2GPUとしなかったのは、GDDRだと基板面積が厳しかったからか、  
Navi10に外部へ出すInfinity Fabricが無く、One MPXに収める利点が特になかったか。  

<br>
他のNavi10採用GPUとまとめたスペックは以下、  

| Navi10 Product | <span style="color:skyblue">W5700</span> | <span style="color:tomato">RX 5700</span> | <span style="color:tomato">RX 5700XT</span> | <span style="color:skyblue">W5700X</span> |
| :--- | :---: | :---: | :---: | :---: |
| WGPs | 18 | 18 | 20 | 20 |
| CUs | 36 | 36 | 40 | 40 |
| SPs | 2304 | 2304 | 2560 | 2560 |
| TMUs | 144 | 144 | 160 | 160 |
| ROPs | 64 | 64 | 64 | 64 |
||
| Game Clock (MHz) | ? | 1625 | 1755 | ? |
| Boost Clock (MHz) | 1929 | 1725 | 1905 | 1855 |
||
| Max Memory Size (GB) | 8 | 8 | 8 | 16 |
| Memory Interface (bit) | 256 | 256 | 256 | 256 | 
| Memory Speed (Gbps) | 14 | 14 | 14 | 14 |
| Memory Bandwidth (GB/s) | 448 | 448 | 448 | 448 |
||
| Peak Texture Fill-Rate (GT/s) | 277.78 | 248.4 | 304.80 | 296.8 |
| Peak Pixel Fill-Rate (GP/s) | 123.46 | 110.4 | 121.90 | 118.72 |
| FP16 (TFLOPS) | 17.78 | 15.9 | 20.28 | 11.0 |
| FP32 (TFLOPS) | 8.89 | 7.95 | 10.14 | 9.5 |
| TDP (W) | 190<br>205(+USB) | 150 | 180 | ? |
