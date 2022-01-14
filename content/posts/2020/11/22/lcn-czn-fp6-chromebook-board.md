---
title: "Lucienne/Cezanne APU を搭載する Chromebookボード　―― Guybrush、Mancomb"
date: 2020-11-22T04:43:01+09:00
draft: false
tags: [ "Lucienne", "Cezanne", "Chromebook", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

先日、Coreboot に *Renoir /Lucienne /Cezanne* 関連の DeviceID が追加されていたが、  
{{< link >}} [Coreboot に Renoir と Lucienne/Cezanne らしき DeviceID が追加される | Coelacanth's Dream](/posts/2020/11/19/rn-lcn-czn-coreboot-did/) {{< /link >}}
それに関連してか、ChromiumOS関連のレポジトリに、*Cezanne* プラットフォームと、それを基にする 2つのボード、**Guybrush** と **Mancomb** の情報が追加されている。  
{{< link >}} [chipset-cezanne: Add initial cezanne chipset overlay (I093f302f) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/2546286) {{< /link >}}
{{< link >}} [Guybrush: Add initial Guybrush overlay (I33a515e8) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/2546293) {{< /link >}}
{{< link >}} [Mancomb: Add initial Mancomb overlay (I8d2b7384) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/2546294) {{< /link >}}

`chipset-cezanne` という書き方ではあるが、意味としては *FP6パッケージ* を指し、*Cezanne* に限らず、*Lucienne* も搭載可能と思われる。2つの APU は主に CPUアーキテクチャが異なるが、同時期に登場すると見られている。  
*FP6パッケージ* を採用し、*Renoir* を搭載したノートPC製品が既に出ているが、それよりも前世代の Chromebook向け AMD APU では、*Raven* をスキップし、*Picasso (12nm)* と *Raven2 (Dali/Pollock, 14nm)* を採用したことから、  
同様に *Renoir* をスキップ、*Lucienne* と *Cezanne* を Chromebook に採用するのだろう。  
*Lucienne/Cezanne* は、I/O、GPU部は *Renoir* とほとんど変わらず、パッケージについても特に変わらないとされる。  

TDP 6W以下をターゲットとしたパッケージと APU(SoC) には、*FT5パッケージ* と [Pollock](/tags/pollock) がいるが、これらの後継については現時点で自分が持っている情報は無い。  
{{< link >}} [AMD Pollock APU データベース | Coelacanth's Dream](/posts/2020/06/14/amd-pollock-apu-database/) {{< /link >}}

また、*FP5パッケージ* を採用する AMD リファンレンスボードのコードネームは **Mandolin** 、*FT5パッケージ* は **Cereme** だったが、*FP6パッケージ* は **Majolica** であるようだ。  
{{< link >}} [Majolica: Add initial Majolica overlay (Iaace83b8) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/2546287) {{< /link >}}

ボードのコードネームについてだが、前世代の *Picasso* または *Raven2 (Dali)* を搭載する *FP5パッケージ* 採用 Chromebook のリファンレンスボードは、ゲームから取ったであろう **Zork** というコードネームが付けられていた。  
そこから派生し、各社が開発、製品化するボードもまた、Zorkシリーズに登場するキャラクターからコードネームを取っていた。  
そして *FP6パッケージ* 採用のボードでは、ゲーム **Monkey Island** から取ることに決めたらしい。[^fp6]  
**Zork** と **Monky Island** はどちらもアドベンチャーゲームであり、**Zork** は 1980年[^zork]、**Monkey Island 1 (The Secret of Monkey Island)** は 1990年[^monkey-island]に登場している。  
この調子で行くと、次は **Myst (2000年)** とかだろうか？  

[^fp6]: [Majolica: Add initial Majolica overlay (Iaace83b8) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/2546287)
[^zork]: [Zork - Wikipedia](https://en.wikipedia.org/wiki/Zork)
[^monkey-island]: [The Secret of Monkey Island - Wikipedia](https://en.wikipedia.org/wiki/The_Secret_of_Monkey_Island)
