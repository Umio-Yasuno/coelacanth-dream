---
title: "混沌とするミッドレンジGPUの価格"
date: 2020-01-18T11:11:09+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "Radeon RX 5600" ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

あと少しで AMD RX 5600 XT が販売開始という中、NVIDIA RTX 2060 の値下げによりミッドレンジGPUの価格が混沌としてきた。  

### 新モデルは投入せずに値下げで迎撃を狙うNVIDIA
RX 5700シリーズにはRTX 20 Superを、RX 5500 XT には GTX 1660 Super を投入してきたNVIDIAだが、  
今回、GTX 1660 Ti 超えをアピールする RX 5600 XT に対して、さすがに RTX 2060 Ti と言ったモデルを追加したりはせず、RTX 2060 を $320.00 から $299.00 に下げることで対応してきた。  
[NVIDIA GeForce RTX 2060 now costs 299 USD - VideoCardz.com](https://videocardz.com/84309/nvidia-geforce-rtx-2060-now-costs-299-usd)  
EVGA から RTX 2060 "KO" というモデルが出たりしているが、特に RTX 2060 から仕様を変えたという話はなく、単に値下げに合わせた新規モデルと見て良さそうだ。  

既に出ている RX 5600 XT の1モデルは価格がデュアルファンで $299.99、トリプルファンで $319.99 となっており、  
[XFX RX 5600 XT Thicc II PRO 6GB - Amazon.com](https://www.amazon.com/dp/B083FWMZ8Z)  
[XFX RX 5600 XT Thicc III PRO 6GB - Amazon.com](https://www.amazon.com/dp/B083FXQMS8)  
RTX 20 Super に対して、販売前に急遽 RX 5700シリーズの価格を下げたように、AMD側にもまた動きが求められることは必至だ。  
そしてそれは既に見え始めている。  

### 値下げではなく仕様変更で対応か
SAPPHIREのPulse RX 5600 XT の製品ページにてスペックが変更されており、変更後は  

 * Game Clock: 1560 MHz -> 1615 MHz  
 * Boost Clock: 1620 MHz -> 1750 MHz  
 * Memory Speed: 12 Gbps -> 14 Gbps  
 * Power Comsumption: 150 W -> 160W

となった。  
同じくPulseシリーズのRX 5700と比べると、Game Clockはまだ85 MHzの差があるが、Boost ClockとMemory Speedでは並ぶこととなる。  
[PULSE RX 5600 XT 6G GDDR6 - sapphiretech.com](https://www.sapphiretech.com/en/consumer/pulse-radeon-rx-5600-xt-6g-gddr6)  
[PULSE RX 5700 8G GDDR6 - sapphiretech.com](https://www.sapphiretech.com/en/consumer/pulse-radeon-rx-5700-8g-gddr6)  
[Sapphire quietly makes Radeon RX 5600 XT Pulse faster - VideoCardz.com](https://videocardz.com/newz/sapphire-quietly-makes-radeon-rx-5600-xt-pulse-faster)  

 > OEMベンダーがGDDR6 14Gbpsを搭載することは許されない

という話だったが、そうも言っていられないのだろう。  
[AMD、Radeon RX 5600シリーズの詳細について説明 - マイナビニュース](https://news.mynavi.jp/article/20200114-953980/)  

他社のモデルではまだ変更を確認できていないが、レビュワーに新VBIOSが配られたことから仕様変更があったことは間違いないだろう。  
[Katsuaki Kato (KTU)さんのツイート: "vBIOS入れ替えるのか……" - Twitter](https://twitter.com/kato_kats/status/1217639168165044224)  
[VideoCardz.comさんのツイート: "It's funny because it's true. AMD: Please update your RX 5600 XT BIOS for review: REVIEWERS:… " - Twitter](https://twitter.com/VideoCardz/status/1218232335176552449)  

NVIDIAが値下げを、AMDが仕様変更による性能向上をおこなったことでお互いに差別化の要因が薄くなってしまった。  
NVIDIAだと、RTX 2060 ($299) と GTX 1660 Ti ($279) の価格差は $20 だけとなり、  
AMDだと、RX 5600 XT の性能を上げたことで RX 5700 との性能差が縮まった。  
また、Amazon.com では上記と同じXFXの RX 5700 が $309.99 であることを考えると、差別化という点でNVIDIAより深刻だ。  
[XFX Rx 5700 8GB - Amazon.com](https://www.amazon.com/dp/B07XVMXBQW)  

ユーザー目線で見ればさらに混沌としてくる。  
企業が健全な競争をしている、より高性能なGPUが安く手に入る、と言えば聞こえは良いが、半導体製品は――ペースが遅くはなっているが――プロセスの微細化やアーキテクチャの改良で「待て」ばより良いものが手に入るのが常であり、  
さらには大作ゲームが軒並み延期していることも考慮すると、下手すれば両方が一時客を逃すということになりかねない。  

<span class="hide">んじゃどうすりゃいいのさ、と言われそうだし自分でも思うが、少なくともAMDはRX 5000シリーズの各製品投入のペースを早めるべきだったと思う。  
RX 5500 XT がわかりやすい例だ。発表してから実際に出てくるまでに、NVIDIAからGTX 1660/1650 Superを出されてしまった。  
Navi10ベースなのだから出そうと思えば RX 5600 XT を早くに出すことは可能だったはずで、そうしなかったのは疑問を抱かずにはいられない。</span>

どういった結果が導かれるかはまだわからないが、この混沌は引き摺り続けるだろう。  
お互いに他の自社製品を巻き込まずうまいこと他社製品と正面でぶつかる形にできなかったものか。  
