---
title: "Chromebookに搭載されるPollockはFT5ソケット ――DDR4シングルチャンネル、SATA無し――"
date: 2020-02-12T22:54:30+09:00
draft: false
tags: [ "Picasso", "Raven2", "Dali", "Pollock" ]
keywords: [ "Chromebook", "Picasso", "Dali", "Pollock", "Ryzen", "Athlon" ]
categories: [ "Hardware", "APU" ]
noindex: false
---

ChrominumOSへの新たなパッチにて、Pollockは他のモバイル向けや組み込み向けのZenプロセッサーで使われてきたFP5 BGAソケットではなく、  
FT5 BGAソケットが使われることが明らかとなった。  

[soc/amd/picasso: Add Kconfig option for chip footprint (Ia4663d38) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051509)  

本来Pollock (Raven2)はDDR4デュアルチャンネル分のメモリコントローラーを備えているが、FT5 BGAソケットではそれをシングルチャネルへ減らし、また、SATAインターフェイスも無いとされている。  

関連するパッチではFT5 BGAソケットでサポートされる最大CPU数（スレッド数）が4までとされていることから、現状Raven2ベースのSKUのみに採用されると思われる。  
[mb/google/zork: Set and use the AMD_FT5 and AMD_FP5 Kconfig options (Ib0da70e2) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051513/1)  

V1000シリーズ (Raven)とR1000シリーズ (Raven2)の仕様を元に考えると、  
Raven2は、Raven (Picasso)と比べて、最大画面出力が1つ少なく(4 &rarr; 3)、PCIe Gen3レーン数は8レーン少ない(16 &rarr; 8)。  
FT5 BGAソケットがRaven2を前提に設計されているならば、これらの分のピンも減らし、パッケージサイズをより小さくしているはずだ。  

既存のA4-9120C/A6-9220Cもシングルチャネル構成だったため、Pollockがその後継的な立ち位置にあり、TDP 6Wという予想が当たっている可能性が高くなった。  

メモリ帯域がFP5 BGAソケットから単純に半分となり、GPU性能には結構影響が出そうだが、コストやターゲットとなる価格帯を考えればちょうど良いのかもしれない。  

<br>
個人的なことを言えば、Raven2の中でFT5 BGAソケットを採用するのがPollock、とかだと複雑なRavenファミリーが少しは整理されるのだが、  
それはChromebookに採用されるであろうDaliとPollockの間でしか決まってないため、なんとも言えない。  
