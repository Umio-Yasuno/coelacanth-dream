---
title: "AMD A9-9820 CPU は Jaguarアーキテクチャ"
date: 2020-07-01T04:27:23+09:00
draft: false
tags: [ "Cato", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

知っている人には今更な内容かも。  

以前 CHUWI が発表した Mini PC [AeroBox](https://www.chuwi.com/product/items/Chuwi-AeroBox.html) に搭載される未だ謎の多いプロセッサ、**A9-9820** だが、  
Geekbench の実行結果が 2020/06/24 にアップロードされていた。  
{{< link >}}[Default string Cato - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15585159), [Default string Cato - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15585185) {{< /link >}}

発表されたその時は、**A9-9820** を Bulldozer系アーキテクチャ(Family 15h) ではないかと考えたが、Geekbench の実行結果を見ると `Family 22(0x16) Model 38(0x26)` となっており、Jaguar系アーキテクチャ(Family 16h) であるようだ。{{< comple >}}(やっぱ自分の推測ってロクに当たらない){{< /comple >}}  
{{< link >}} [CHUWIよりAMD A9-9820搭載MiniPC発売予定 & 推測 | Coelacanth's Dream](/posts/2020/03/14/chuwi-amd-a9-9820/) {{< /link >}}

## まだまだ謎ばかり
前回違うと考えた、ゲーム機向けチップの転用が逆に正しかったのかははっきりとせず、曖昧なままだ。  

メモリ情報が正しく取得されておらず、帯域も異様に低くなっている。CPUの実行結果であっても、GPU情報が取得されている場合があるのだが、それもさっぱり。  

*Xbox One SoC* だとしたら L3/L4キャッシュに ESRAM 32MB が認識されているのでは、と最初考えたが、HotChips 25 での発表によると、ESRAM 32MB はグラフィクス専用であり、CPUからは直接アクセス出来ないようになっているため、恐らく *Xbox One SoC* で実行しても ESRAM 32MB は認識されないだろう。[^hc25-xbox-one]  
*Xbox One SoC* はメインメモリに DDR3 4ch を持つが、前述したようにメモリ情報が正しく取得されていないため、はっきりしない。  

[^hc25-xbox-one]: <https://www.hotchips.org/wp-content/uploads/hc_archives/hc25/HC25.10-SoC1-epub/HC25.26.121-fixed-%20XB1%2020130826gnn.pdf>

`Northbridge: AMD ID1546 00` を手掛かりに、[The PCI ID Repository](http://pci-ids.ucw.cz/)を確認しに行ったが、`Kryptos/Cato/Garfield/Garfield+/Arlene/Pooky Root Complex` とあり、これまたはっきりしない。[^pciid-1022-1546]  
その一連のコードネームも謎で、*Kryptos* が *Xbox One SoC* [^xbox-one-kryptos]を、*Cato* が **A9-9820** 等のベースとなるチップを指すらしいが、他は不明。  
ただ Root Complex の PCI ID はそれぞれ別のチップで共用することがあり、*Raven* と *Raven2* がそうなっている。[^pciid-1022-15d0]  
そのため、*Kryptos /Xbox One SoC* == *Cato* ではない(!=) と考えている。{{< comple >}}これも外れるかも{{< /comple >}}  

[^xbox-one-kryptos]: [AMD XBox One APU](http://www.cpu-world.com/CPUs/Jaguar/AMD-XBox%20One%20APU.html)
[^pciid-1022-1546]: <http://pci-ids.ucw.cz/read/PC/1022/1546>
[^pciid-1022-15d0]: <http://pci-ids.ucw.cz/read/PC/1022/15d0>

**A9-9820** 実物を搭載した製品が出てきてくれれば手っ取り早いのだが、CHUWI AeroBox は 2020/03/12 に近日発売とアナウンスしたきり、続報が無い。  
