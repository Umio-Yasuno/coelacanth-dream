---
title: "AMD Renoir の兄弟？ 現れるは Lucianne"
date: 2020-06-20T00:46:19+09:00
draft: false
tags: [ "Renoir", "Zen_2" ]
keywords: [ "", ]
categories: [ "Hardware", "APU", "AMD" ]
noindex: false
---

新たな AMD APUのコードネームに *Lucianne* の名前が浮かび上がってきた。  

ChromiumOSを構成する Coreboot へのパッチの中で、[Matt Papageorge](https://chromium-review.googlesource.com/q/owner:matt.papageorge%2540amd.corp-partner.google.com) 氏は Zen系 APU をサポートするコードを追加し、  
その中で *Renoir* に関係した APU として *LCN* という略称が現れた。  
{{< link >}}[util/amdfwtool: Add support for EFS SPI values for F17h and F15h (I87c4d441) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2246284/5){{< /link >}}

氏はコメントの中で、*LCN* は *Lucianne* の略称、そして *Raven* に対する *Picasso* のような存在であり、*Renoir* の次製品(Renoir kicker) と説明している。[^1]  

[^1]: <https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2246284/5/util/amdfwtool/amdfwtool.c#1087>

ただ気になることがあり、パッチの中では *Renoir* と *Lucianne* の `X86_Model` を `30h-3Fh` としているが、  
*Renoir* の `x86_Model` は `60h` だ。[^2]  
`30h-3Fh` では、 `Family 17h Model 31h` に *Castle Peak(Zen 2 TR) / Rome(Zen 2 EPYC)* があるため、それらを指してしまう。  

現時点で最新の Patchset 6 では *LCN* に限らず、構造体変数名からコードネームは取り除かれ、別のコードで処理が実装されているが、`30h-3Fh` という部分は変わっていない。  

[^2]: [hwmon: (k10temp) Add AMD family 17h model 60h PCI match](https://git.kernel.org/pub/scm/linux/kernel/git/tip/tip.git/commit/?h=ras/core&id=279f0b3a4b80660fba6faadc2ca2fa426bf3f7e9)

それと *Lucianne* の元ネタがわからん。自分が無学というのもあるが、検索してみてもしっくり来るようなものはなかった。*Picasso* とか *Dali* とか *Pollock* みたいな関係性を持ったものは。  
[Lucianne Goldberg](https://en.wikipedia.org/wiki/Lucianne_Goldberg) という作家は見つかったが。  

そういうことで怪しい部分を無視して考えるならば、*Lucianne* は *Raven* に対する *Picasso* のように、内部のアーキテクチャは特に変えない *Renoir* のプロセス最適化版、**かもしれない** 、またはそうした APU が計画にあるということだけを書き残し、  
後は深く考えないでおく。  
