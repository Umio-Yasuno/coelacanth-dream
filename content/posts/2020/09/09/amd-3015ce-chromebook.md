---
title: "Chromebook向け Pollock APU、AMD 3015Ce"
date: 2020-09-09T23:51:44+09:00
draft: false
tags: [ "Chromebook", "Pollock", "Raven2" ]
# keywords: [ "", ]
categories: [ "AMD", "APU", "Hardware" ]
noindex: false
# summary: ""
---

Geekbench の実行結果から、Chromebook向けとなる[AMD Pollock APU](/tags/pollock)のSKU名、**AMD 3015Ce** が明らかになった。  
{{< link >}} <https://browser.geekbench.com/v5/cpu/3664531> {{< /link >}}

## AMD 3015Ce の名が出てくるまで

これまでの振り返りとなるが、**AMD 3015Ce** という名前が出てくるまでにはだいぶ時間が掛かっている。  

まず、*Pollock* というコードネームが OSS 上で初めて出てきたのは、2020/01/08 に投稿された Linux Kernel へのパッチである。  
{{< link >}} [[PATCH] drm/amd/display: add Pollock IDs, fix Pollock & Dali clk mgr construct](https://lists.freedesktop.org/archives/amd-gfx/2020-January/044548.html) {{< /link >}}
この時は [Raven2](/tags/raven2) の別リビジョンということくらいしか分からず、何が *Pollock* たらしめるのかは不明だった。  
その約一ヶ月後、Chromebook向けの [Picasso](/tags/picasso)、[Dali](/tags/dali) をベースとする Zen/+ APU が判明し[^fp5-chromebook-sku]、その中に *Pollock* も含まれてはいた。しかし、SKU名は無く、`Samples` または `Production` とあるだけだった。[^plk-samples-production]  
それから *FT5 BGA パッケージ* の一部仕様と、その *FT5 BGA* を採用する APU こそが *Pollock* であると判明した。[^plk-ft5]  
それでもまだ *Pollock* の具体的な SKU名は不明なままにあり、  
*Pollock* を搭載するChromebookボード、コードネーム **Dalboz** の実行結果も以前より Geekbench 上に出現していたが、CPU名は `Intel Pentium II/III` となっていた。恐らくサンプル品を用いたものだろう。[^gb5-dalboz-samples]  

[^plk-ft5]: [Chromebookに搭載されるPollockはFT5ソケット ――DDR4シングルチャネル、SATA無し | Coelacanth's Dream](/posts/2020/02/12/amd-pollock-ft5/)

Chromebook から少し外れて、現地時間 2020/08/04 に Lenovo より **AMD 3015e** を搭載する教育機関向けのラップトップが発表された。[^lenovo-amd-3015e]  
現状 *Pollock* を搭載する製品は、その時 Lenovo から発表された **Lenovo 100e 2nd Gen** 、**Lenovo 300e 2nd Gen** のみとなっている。  

そして今回 **AMD 3015Ce** という具体的な SKU名を伴う結果が現れた。  
これにより、ようやく Zen系APU (*Picasso /Dali /Pollock*) を搭載するChromebook製品登場の予感がしてくる。  

Intel は既に [Jasper Lake](/tags/jasper_lake)、[Tiger Lake](/tags/tiger_lake) の Corebootサポートを進めており、それらのリファレンスボードを元に派生ボードを開発されている。  
{{< link >}} [Intel 10nm プロセッサを搭載した Chromebook ボード | Coelacanth's Dream](/posts/2020/05/13/intel-10nm-processor-chromebook/) {{< /link >}}
それと比べると、最新世代の 7nm [AMD Renoir](/tags/renoir) ではなく、12/14nm の *Picasso /Dali /Pollock* 搭載 Chromebook が開発中というのは少し遅れているようにも思えるが、Chromebookの主なターゲット層を考えれば、性能、コスト的にちょうど良いのだろう。  
*Renoir* よりダイサイズの小さい、コストが小さい APU が出てくれば、次世代 AMD プロセッサ搭載 Chromebook の登場に期待が持てるはずだ。  

[^fp5-chromebook-sku]: [AMD Zen APUを搭載したChromebook登場か ――Ryzen 7 3700C、Ryzen 5 3500C | Coelacanth's Dream](/posts/2020/02/09/amd-zen-chromebook/)
[^plk-samples-production]: […/northbridge.c · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/3/src/soc/amd/picasso/northbridge.c#320)
[^gb5-dalboz-samples]: <https://browser.geekbench.com/v5/cpu/2323968>
[^lenovo-amd-3015e]: [Lenovo Enhances Portfolio of Education Solutions to Meet Evolving Demands of Hybrid Learning | Lenovo StoryHub](https://news.lenovo.com/pressroom/press-releases/lenovo-enhances-portfolio-of-education-solutions-to-meet-evolving-demands-of-hybrid-learning/)

## AMD 3015Ce の仕様は {#amd-3015ce-spec}

**AMD 3015Ce** の CPU/GPU仕様は、他のChromebook向け APU から見られるパターンから、**AMD 3015e** と同じと思われる。  
Geekbench の結果から確認できる CPUクロックはセグメントナンバーが一致する Athlon/Ryzen U-Series SKU と一致し、GPU の RevisionID も U-Series と同じものが割り振られていることから変わらないと考えられる。  

ただ、**AMD 3015Ce** に関しては、対応する **AMD 3015e** より少し低い TDP 仕様となるかもしれない。  

**AMD 3015Ce** を搭載するボード **Dalboz** は、デフォルトの TDP、持続可能なブースト状態で 4.8W、最大時で 9W、デフォルトから最大へ、最大からデフォルトへの移行時で 6W の電力制限が設定されている。  
{{< link >}} [mb/google/zork: update power parameters to 4.8w for dalboz (I711d1109) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2135098) {{< /link >}}

TDP 6W の **AMD 3015e** と CPU/GPU 仕様は同じと考えられるが、ブーストの余裕を減らすことで TDP を下げるアプローチを取ったのだろう。  

*Picasso/Dali* を搭載する Chromebook ボードには、デフォルトの 15W ではなく cTDP の下限である 12W を設定しているボードもあり、バッテリー持続時間や性能等、ボード間で異なる特徴を出している。  
*Pollock* も TDP を 4.8W か 6W かで選択できる仕様となっているのではないかと思う。  

| Zen Chromebook Board | APU | TDP | Slow PPT | Fast PPT |
| :-- | :--: | :--: | :--: | :--: |
| Berknip | *Picasso/Dali* | 12W | 20W | 24W |
| Dalboz | *Pollock* | 4.8W | 6W | 9W |
| Dirinboz | *Pollock* | 4.8W | 6W | 9W |
| Ezkinil | *Picasso/Dali* | 12W | 20W | 24W |
| Morphius | *Picasso/Dali* | 12W | 20W | 24W |
| Trembyle | *Picasso/Dali* | 15W | 25W | 30W |
| Woomax | *Picasso/Dali* | 15W | 25W | 30W |

| Pollock SKU | Core/Thread | CPU Base/Boost | GPU CU | GPU Clock | TDP |
| :-- | :--: | :--: | :--: | :--: | :--: |
| AMD 3015e[^amd-3015e] | 2/4 | 1.2GHz/2.3GHz | 3 | 0.6GHz | 6W |
| AMD 3015Ce[^amd-3015ce-gb5] | 2/4 | 1.2GHz/2.3GHz | 3? | 0.6GHz? | 4.8W? |

[^amd-3015e]: [AMD 3015e | AMD](https://www.amd.com/en/products/apu/amd-3015e#product-specs)
[^amd-3015ce-gb5]: [Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/3664531)
