---
title: "Navi14 Matome Part2"
date: 2019-11-13T20:49:31+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

Navi14のSKU、DeviceID、RevisionID、GameClock、Productをまとめた表。  
CSSのtable部をいじったことで実用的になったため掲載する。  

| SKU | DeviceID | RevisionID | GameClock (MHz) | Product |
|:---:|:---:|:---:|:---:|:---:|
| Gaming XT | 7340 | C7 | 1670 | RX 5500 (11WGP) |
| | | F4 | | CU Test Mode? |
| Gaming XTM | 7340 | C1 | 1448 | RX 5500M (11WGP) |
| | | F2 | | CU Test Mode? |
| Gaming XTX | 7340 | C5 | 1717 | RX 5500XT (11WGP) |
| | | F6 | | CU Test Mode? |
| XLM? | | C3 | 1181 | RX 5300M (11WGP) |
| | | F3 | | CU Test Mode? |
| XL? | | | 1448 | |
| WKS Pro-XTM | 7347 | 00 | ? | Pro 5500M (12WGP) |
| WKS Pro-XL | 7341 | 00 | ? | |
| WKS Pro-XLM | 734F | 00 | ? | Pro 5300M (10WGP) |
<br>
WKS Pro-XLMのRevisionIDははっきりと出ていないが、  
[add support for wks firmware loading](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=73e5bc910eb3e697f0bf33aa63ad64549db4212a)  
デバイスを判定する部分のコードが、DeviceIDが7340以外かつRevisionIDが00のものはwksが付いたファームウェアを読み込む、  
となっていることから、WKS Pro-XLMもRevisionIDは00と推測される。  

あくまで個人による推測であり、確定情報ではない。  
元となる情報は出来る限り載せているため、そちらを確かめた上で判断してもらいたい、というのが自分のスタンス。  
[Navi14 Matome](/posts/2019/11/04/navi14-matome)  
### ShadeArrayについて
1WGP=2CUということで、NaviでもCU数を基準に考えてしまいがちだが（私がそう考えてた）、  
CUの片方だけをオフになんてことはできない（はず）なので、WGPを基準に考える方が正確だ。  

Navi14で言えば、チップとして12WGP（24CU）中、11WGP（22CU）を有効にされた製品が公式から発表されている。  
そして、Navi14は1ShaderEngine、2ShaderArrayという構成であり、  
ShadeArrayあたりのWGP数は5、または6となる。  
そしてこのことからShaderArrayあたりのWGP数は同数（=ShaderArrayが対称）である必要はないということがわかる。  
だから何だと言われそうだが、これは今後の推測に役立つだろうし、頭の片隅に置いておけばShaderArrayあたり11CUなんて勘違いもせずに済むかもしれない。
