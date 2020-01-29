---
title: "Navi14のダイ観察 & 推測"
date: 2020-01-26T11:16:05+09:00
draft: false
tags: [ "Navi10", "GFX10" ]
keywords: [ "", ]
categories: [ "Hardware", "GPU" ]
noindex: false
---

[Fritzchens Fritz - Flickr](https://www.flickr.com/photos/130561288@N04/)氏が先日のNavi10のダイショットからあまり間を空けずに、Navi14のダイショットも公開した。  
[AMD@7nm@RDNA_1th_gen@Navi14@Radeon_RX_5500_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/49437016132/)  
既にネット上ではGPUに詳しい方達が推測したダイアグラムを上げているが、自分のためにもとりあえず作ってみることとした。  

{{% figure src="/image/2020/01/26/navi14-dieshot-guess.webp" title="Navi14推測" caption="Photo Credit: Fritzchens Fritz" %}}

やはりというか自分がチキンなだけだが、左付近の大きめのブロックはDisplay PHYだった。Navi14は5 PHY持つことからほぼ間違いないだろう。  

USB-Cコントローラーは、左中よりほんの少し下辺りが怪しい。  
Navi10もNavi14もそこでDisplay PHYが2基をまとめるようになっており、付近の回路/ブロックも大体同じに見える。  
USB-Cで映像出力もする都合上、そこで信号を混ぜたり切り替えられたりできるようにしてそうだが、やっぱり確信はない。  

VCN 2.0はDisplay PHY、2つの塊の間か、左上にありそうだが、どちらかと言えば左上な**気**がする。するだけ。  

{{% figure src="/image/2020/01/26/navi10-14-compare.webp" title="Navi14 /Navi10 比較" caption="Photo Credit: Fritzchens Fritz"%}}

Navi14とNavi10のダイショットを実際の比に合わせて並べたもの。  
PCIeGen4 PHYとGDDR6 32-bit PHYの間にNavi10/Navi14 共通のブロックがあるが、個人的な予想ではPSP(Platform Security Processor)ではないかと思う。  

HDCPのため、暗号化等を行なうPSPはマルチメディア部の近くに配置した方が都合が良いはずだ。  

こうしてみると、I/Oやマルチメディア部はDisplay PHY 1基とPCIeGen4 4-Lane PHY 2基の違い以外はほとんど一緒であることがわかる。  