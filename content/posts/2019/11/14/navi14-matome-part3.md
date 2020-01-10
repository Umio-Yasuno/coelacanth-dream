---
title: "Navi14 Matome Part3"
date: 2019-11-14T00:47:27+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

RX 5500XTも入れたが、現状スペックは推測である。  

| Navi14 Product | <span style="color:crimson">RX 5300M</span> | <span style="color:cornflowerblue">Pro 5300M</span> | <span style="color:crimson">RX 5500M</span> | <span style="color:cornflowerblue">Pro 5500M</span> | <span style="color:crimson">RX 5500</span> || <span style="color:crimson">RX 5500XT?</span>
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| WGPs | 11 | 10 | 11 | 12 | 11 | | 11 |
| CUs | 22 | 20 | 22 | 24 | 22 | | 22 |
| SPs | 1408 | 1280 | 1408 | 1536 | 1408 | | 1408 |
| TMUs | 88 | 80 | 88 | 96 | 88 | | 88 |
| ROPs | 32 | 32 | 32 | 32 | 32 | | 32 |
||
| GameClock (MHz) | 1181 | ? | 1448 | ? | 1670 | | 1717 |
| BoostClock (MHz) | 1445 | 1250 | 1645 | 1300 | 1845 | | 1845 |
||
| Max Memory Size (GB) | 3 | 4 | 4 | 8 | 4 | | 8 |
| Memory Interface (bit) | 96 | 128 | 128 | 128 | 128 | | 128 |
| Memory Speed (Gbps) | 14 | 12 | 14 | 12 | 14 | | 14 |
| Memory Bandwidth (GB/s) | 168 | 192 | 224 | 192 | 224 | | 224 |
||
| Peak Texture Fill-Rate (GT/s) | 127.2 | 100.0 | 144.76 | 114.4 | 162.36 | | 162.36 |
| Peak Pixel Fill-Rate (GP/s) | 46.2 | 40 | 52.60 | 41.6 | 59.00 | | 59.00 |
| FP16 (TFLOPS) | 8.14 | 6.4 | 9.26 | 8.0 | 10.4 | | 10.4 |
| FP32 (TFLOPS) | 4.07 | 3.2 | 4.63 | 4.0 | 5.2 | | 5.2 |
||
|SKU| XLM | WKS Pro-XLM | GamingXTM | WKS Pro-XTM | GamingXT | | GamingXTX | 

モバイル向けProのGameClockは外れていた。  
GamingXTMとWKS Pro-XTMが同じと少しでも考えたのは浅はかだった。  
そもそも実際の判定はRevisionIDで行われているのだから、DeviceIDもRevisionIDも違うWKSが適用される訳がなかった。  
[navi10_ppt.c](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/navi10_ppt.c?h=amd-staging-drm-next#n1573)

残るSKUはXL、WKS Pro-XLとなる。  
XLは、RX 5300として（GameClock 1448MHzということから補助電源無しが可能なのを期待しているがどうなるか）、  
WKS Pro-XLは、モバイル向けでないApple用のNavi14として出ると考えているが、  
後者はiMacにでも搭載されるのだろうか。  

他Navi14まとめ  
[Part1](/posts/2019/11/04/navi14-matome)  
[Part2](/posts/2019/11/13/navi14-matome-part2)  
