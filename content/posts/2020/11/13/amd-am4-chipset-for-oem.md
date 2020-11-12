---
title: "AMD、OEM向け AM4チップセットの仕様を公開"
date: 2020-11-13T02:57:13+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "Chipset" ]
noindex: false
# summary: ""
toc: false
---

いつかに OEM向けの新たな小型PC向けチップセット *PRO 500* を記事に取り上げた時 {{< comple >}} 辿ってみたら 10ヶ月近く前だった {{< /comple >}}、OEM向けチップセットは出回ることが少なく、また資料もほとんど公開されないため、わからないことが多いと書いたが、  
{{< link >}} [AMDの新しい小型PC向けチップセット「PRO 500」 | Coelacanth's Dream](/posts/2020/01/28/amd-pro-500-chipset/) {{< /link >}}
AMD AM4チップセットのページを見た所、USB や PCIeレーンの数、OC といった、OEM向けチップセットの仕様が追加されていた。最近出てきた *PRO 565* の仕様も記載されている。  

 * [Socket AM4 Chipset | AMD](https://www.amd.com/en/products/chipsets-am4)

以前 *PRO 500* の情報を探す中で、HP の資料から、*B300* が一度 *B500* となってから、**Ryzen 3000シリーズ (Matisse)** 向けに *PRO 500* と名前を変えたのではないかと書いたが、I/O の仕様を見ても *B300* は *PRO 500* とほとんど同じで、**Ryzen 3000シリーズ (Matisse)** から対応している CPU からの USB 3.2 Gen2 が利用可能という点だけ異なっている。  
{{< link >}} [AMD 「PRO 500」チップセット続報 & まとめ | Coelacanth's Dream](/posts/2020/02/01/amd-pro-500-follow-up/) {{< /link >}}

*PRO 565* については、CPU からの USB 3.2 Gen2 4ポートが利用可能、PCIe Gen4 には、CPU と直接接続される GPU と NVMe SSD のみ対応と、コンシューマ向けの *B550* とほぼ同じ仕様となっている。  
*B550A* は *X370/X470* をベースに、CPU からの USB 3.2 Gen2 4ポートに対応させたといった感じで、見比べてみるとちょっとした発見があって面白い。  

製造プロセスや TDP が記載されていないことは少し残念だが、OEM向けチップセットの仕様が明らかになってことで、どこかすっきりしたものを感じる。  

| Model Number | Total USB | USB 3.2 Gen2 (CPU) | SATA | Graphics | NVMe | PCIe Gen | PCIe Lanes<br> (Total/Usable) | OC |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| PRO 565 | 14 | 6 (4) | 8 | 1x16/2x8 | 1x4 | Gen4<br>(Graphics & NVMe only) | 38/30 | N |
| PRO 500 | 4 | 4 (4) | 2 | 1x16 | 1x4 | Gen3 | 24/24 | N |
| PRO 560 | 18 | 2 | 10 | 1x16/2x8 | 1x4 | Gen3 | 40/32 | N |
| B550A | 18 | 6 (4) | 10 | 1x16/2x8 | 1x4 | Gen3 | 40/32 | Y |
| X300 | 4 | 0 | 2 | 1x16/2x8 | 1x4 | Gen3 | 24/24 | Y |
| B300 | 4 | 0 | 2 | 1x16 | 1x4 | Gen3 | 24/24 | N |
| A300 | 4 | 0 | 2 | 1x16 | 1x4 | Gen3 | 24/24 | N |
