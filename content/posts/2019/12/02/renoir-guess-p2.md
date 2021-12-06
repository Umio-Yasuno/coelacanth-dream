---
title: "Renoirの構造推測 Part2"
date: 2019-12-02T23:51:20+09:00
draft: false
tags: ["Radeon", "GCN", "Raven", "GFX9", "gfx909", "Renoir", "Zen_2" ]
keywords: [ "Radeon", "Renoir", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

Renoir推測の更新。  

以前はZen2 CCDに、GPUを統合したIODを組み合わせると予想していたが、  
[Renoirの構造推測](/posts/2019/11/09/renoir-guess/)  
どうもそうでない可能性が高くなった。  

きっかけはこのツイート。  
<https://twitter.com/TUM_APISAK/status/1201295232916062208>  
[APISAK - Twitter](https://twitter.com/TUM_APISAK)氏はリーカー、というよりはベンチマーク結果から未発表のハードウェアを見つけ出す達人のように私は思う。  

ツイートの画像だが、恐らくSiSoftware Sandraの実行結果で、左はハードウェア名、右はスペックを表しているように読める。  

"Celadon-RN Renoir"という名前自体は以前にも確認されており、この画像が初出という訳ではない。  
[[PATCH] drm/amdkfd: fix the missed asic name while inited renoir_device_info](https://lists.freedesktop.org/archives/amd-gfx/2019-September/039826.html)  

スペックだが、512SPはそのままの意味、8CというのはCU数だと思われる。  
8 [CU] * 64 [SP] = 512 [SP] で計算も合う。  
そして1.75GHzだが、GPUクロックを意味する、はずだ。  

似たような記述をされるハードウェアにVega M GLがあり、  
[Details for Component AMD Radeon RX Vega M GL - SiSoftware](https://ranker.sisoftware.co.uk/show_device.php?q=c9a598d994d0f0a2c3a7c2adc3e3b1e9c99ffa9dfcdc91b1f6ba9cfbc6ebdafc8eb383a5ccf1d7bf82a4dce1c7a2c7facaec9fa29a&l=en)  
CU数の後、一様に550MHzとある。  
550MHzというのは最低GPUクロックだと思われるが、Vega M GLのPPTableを読まない限り確定ではない。  
CPUクロックである可能性は、Intel CPUでも最低クロックは800MHzとかで、Zen2だと2.2GHzとかなので無いと思われる。  
APUであるRenoirの最低GPUクロックが1.75GHzなんてのは高すぎるが、  
テスト用に省電力機能を無効にした結果、最高クロックが取得されたと考えれば不自然ではない。  

ただGPU L1$が、L2と表示されていたり、リストの方ではメモリクロックと思われる500MHzをSpeedとしていたりと、SiSoftware Sandraの見方がよくわからない。  

まあとりあえず、Renoirの最高GPUクロックは1.75GHzとして話を進める。  

### 7nmダイに統合?
この1.75 GHz(1750 MHz)というクロックだが、14/12nmではまず無理だ。  
dGPUになるが、14nm Vega64で1.546 MHz、12nm RX590で1545 MHzなため、7nmでないと1750 MHzは出せない。  
そしてGPUが7nmということは、以前のIOD側にGPUが統合されるという推測は外れていた、となる。  
改めて考えると、以前の推測はかなりガバガバだ。  
既にAMDはAPUにおいてチップレット構造は取り入れないと語っている。  
[AMD: “No Chiplet APU Variant on Matisse, CPU TDP Range same as Ryzen-2000” - AnandTech](https://www.anandtech.com/show/13852/amd-no-chiplet-apu-variant-on-matisse-cpu-tdp-range-same-as-ryzen2000)  
それでもチップレット構造として推測したのは、CCDを使い回すことができ、7nmより相対的にコストが低い14/12nm IODを新たに立ち上げるだけで済み、  
LPDDR4をサポートするとは言え、メモリ帯域がボトルネックとなるのだからAPUでGPU部分を強化してもあまり意味がないと思ったからだ。  
モバイル向けで不利となるダイ間接続による消費電力増は、まあ fclk 抑えることでどうにかなるんじゃないかとテキトーに考えていた。  

数が必要となるモバイル市場において、7nmで新たにダイを起こすことのコストはあまり考えなくてもいいのだろうか？  

### 推測図
前回作った画像を1ダイ構造に変更したが、あまり詰めていないのはご愛嬌。  
![renoir-guess-p2](/image/2019/12/03/renoir-guess-p2.webp)  

7nmなのにNaviではなくVegaにしたのは、NaviだとL1キャッシュ 128KB等でGPU部が必要とするダイサイズが大きくなってしまうからと思われる。  

2CCXの8C/16Tとなっているが、図を使い回した結果なので自信はあまりない。  
RenoirにRyzen 7/9が付いた名前があった、という「噂」はあるが、根拠にするには微妙なのでここでは無視する。  
コスト的にも消費電力的にもダイサイズは抑えたいだろうし、Zen2ではまだCCX単位が4C/8Tなため、現行APUと同じ4C/8Tで続けるかもしれない。  
今の所、対抗となるIce Lakeも4C/8Tであり、CPU性能はZen2で十分戦えるはずで、  
GPU性能もクロック向上とLPDDR4で問題ないだろう。  
（上記と若干矛盾しそうだが、dGPUのGDDR5/6程ではないとは言え、DDR4よりは性能が出る。）  

IO部も個人的な考えではPCIeGen4は必要ないように思えるが、まあわからない。  
PCIeGen3とするなら、IO HUBへの接続が32B/Cで済むだろうし、まだ発熱の厳しいPCIeGen4 SSDがモバイル向けで支持されるかは怪しいはず。Ice LakeもGen3止まりだ。  
だが、発表されるのが2020年CESだとしても、実際に出るまではもっと時間がかかるため、その時の需要を考えればGen4になるかもしれない。  


#### DC Patches - 2 Dec 2019
これを書いてる最中、少し久しぶりな気がするDC（Display Core）へのパッチが投稿された。  
[[PATCH 00/51] DC Patches - 2 Dec 2019](https://lists.freedesktop.org/archives/amd-gfx/2019-December/043394.html)  
内容にはRenoir用のパッチが含まれ、レイテンシが削減されたりと開発がある程度順調なことを窺える。  
[[PATCH 26/51] drm/amd/display: update p-state latency for renoir when using lpddr4](https://lists.freedesktop.org/archives/amd-gfx/2019-December/043419.html)  
[[PATCH 32/51] drm/amd/display: update sr latency for renoir when using lpddr4](https://lists.freedesktop.org/archives/amd-gfx/2019-December/043423.html)  
Daliのもあり、中身としてはほぼRaven2だという話がより強固となった。  
[[PATCH 04/51] drm/amd/display: Fix Dali clk mgr construct](https://lists.freedesktop.org/archives/amd-gfx/2019-December/043395.html)
