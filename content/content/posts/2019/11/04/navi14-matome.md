---
title: "Navi14 Matome"
date: 2019-11-04T04:30:28+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

今月中に（多分）発売されるであろうRX 5500に使われるチップ、Navi14について、わかってるだけの情報をまとめていく。  
## 性能面
2019-10-07にAMDからRX 5500 seriesが発表された。  
[AMD Introduces Radeon™ RX 5500 Series Graphics: Superior Visual Fidelity, Advanced Features and High-Performance Gaming Experiences](https://www.amd.com/en/press-releases/2019-10-07-amd-introduces-radeon-rx-5500-series-graphics-superior-visual-fidelity)  
その際明らかにされたスペックは以下。  
(一部簡略化)  

| Model | Compute<br>Units | Stream<br>Processors | ---TFLOPS--- |---GDDR6---<br>(GB) |    ---GameClock---<br>(MHz)   | ---BoostClock---<br>(MHz) | Memory<br>Intreface |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| RX<br>5500M | 22 | 1408 | Up to 4.6 | 4 | 1448 | 1645 | 128-bit |
| |
| RX 5500<br>series | 22 | 1408 | Up to 5.4 | Up to 8 | 1717 | 1845 | 128-bit |

個人的に重要だと思うのは、最終行がRX 5500のスペックではなく、RX 5500 "series"のスペックだという点だ。  
AMDはこれとは別にRX 5500のスペックも公表しており、  
[RX 5500 Product Spec](https://www.amd.com/en/products/graphics/amd-radeon-rx-5500#product-specs)  
RX 5500 "series"とは最大メモリ容量とGameClockが違うことがわかる。（RX5500はGDDR6: 4GB、GameClock: 1670MHz）  
言い換えれば、RX 5500のそこだけを変えた上位製品を最低でも１つAMDは予定している。  
それが俗に言うRX 5500XTだろう。  

ここから推測と感想となる。  
まず、無印とXTの差はかなり微妙なものになるはずだ。  
BoostClockにどれくらいの頻度で到達するのかは不明だが、そこに差はなく、  
GameClockも47MHzの違いしかないため、その程度であればRX 5500のカスタムモデルがあっさり抜きそうな気がする。  
メモリ容量もXTだと2倍の容量にはなるが、メモリバス幅は無印と同一なため、4GBより多く使うゲーム等で性能に大きく差をつけるものの、そうでない場合は無印と変わらない感じになるだろう。  
メモリバス幅 128-bitで8GBにする方法として、8x8Gbチップをそれぞれ16bitで接続するのと、4x16Gbチップをそれぞれ32bitで接続するものがある。  
前者は基板をそれ用に設計しなければならない上、単純に枚数が多いためコストと消費電力の面で不利となる。  
16bit接続に対応させるのは、16bit接続&16Gbチップにしないといけないようなメモリ容量が必要とされる用途向けの製品だけになると考えている。  

それと少し残念な話となるが、Navi14を使った製品でRX 5500XTより上の製品がゲーミング向けに出るかは怪しい。  
まずNavi14はチップとして24CU持っているが、RX 5500 seriesではその内22CUを有効化して製品としている。  
流れで言えば、5500と5700の間、5600に24CUをフル有効にしたものを製品にしそうだが、現状ゲーミング市場にそれは投入されない可能性が高い。  
ゲーミング向けにNavi14を使ったSKUが3つ確認されており、[Navi14 PROXL DID](https://lists.freedesktop.org/archives/amd-gfx/attachments/20190910/e8054b58/attachment-0001.obj)それぞれ  
Gaming XT、Gaming XTM、Gaming XTXとなっている。  
これらのDeviceIDは7340で、RevisonIDは左からC7、C1、C5。
それと別に、LinuxドライバのAMDGPUの電源管理部分でNavi14のPeakClockに関する記述があり、  
[navi10_ppt.h](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/navi10_ppt.h?h=amd-staging-drm-next)  
[navi10_ppt.c](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/navi10_ppt.h?h=amd-staging-drm-next)  
XTがRX 5500のGameClockに、XTMがRX 5500Mに、XTXがRX 5500 "series"のGameClockに一致する。  
（Navi10の記述もあり、XLがRX 5700、XTがRX 5700XT、XTXがRX 5700XT 50thのGameClockと一致する。）  
つまりGaming向けのSKU3つは既に埋まっている。
このことからRX 5600(仮)が出るかは怪しいと考えた。  
Navi14のSKUにはWKS Pro-と付いたものが他に存在しており、ファームウェアもきっちり分けられている。  
24CU製品が出るとしても、Pro向けやMacBookPro用として出ると思われる。  
ただこれから先SKUが追加されるかもしれないため、絶対の話ではない。  
それが確認された時はまた記事にするつもりだ。  

上位モデルは望み薄だが、下位モデルは追加される可能性が高い。
Navi14のSKUに使われる名前にはXT、XTM、XTXの他にXLM、XLがあり、  
XLMのRevisionIDはC3でGame(Peak)Clockは1181MHzとNavi14の中では最も低い。  
そしてXLのRevisionIDはまだわかっていないが、GameClockは1448MHzとRX 5500Mと同じである。  
XLでは補助電源無しモデルが期待できるかもしれない。  

## インターフェース周り
### RadeonMediaEngine
Navi10と同じVCN2.0が搭載されている。  
RX 5700/XTは、VP9/HEVC decode 4K90/8K24に対応していると発表されており、Navi14でもそれらは制限されることなく有効だと思われる。  
### RadeonDisplayEngine
これもNavi10と同じDCN2.0を搭載しているが、同時出力画面数は5つまでとなっている。
### PCIe 4.0
レーン数はNavi10が16レーンだったのに対し、Navi14では8レーンとなっている。  
RX 5700/XTの時はPCIe 3.0より2倍の帯域を持つため動画編集にも向いているとアピールできたが、8レーンではPCIe 3.0 16レーンと同等の帯域なためできなくなった。 

## 少しマニアックな話
Navi14のもう少し詳細な構成を話す。  
まず、GCNではCore部分を最大4つのShaderEngineに分け、それぞれに最大16CU、最大4RB(RenderBackend)=16ROPを搭載する形になっていたが、  
RDNAでは、GCNでのL1キャッシュとL2キャッシュの間にキャッシュ(128KB)が新たに追加され、GCNでのL1キャッシュはL0キャッシュという名前になった。  
そして、RX 5700XTを例に言えば、5WGP(10CU)と4RBをL1キャッシュをまとめた構成を、ShaderArrayと呼ぶこととなった。  
RDNAにおいてShaderEngineは、ShaderArrayを2つまとめた単位であり、Navi10は2 ShaderEngine、4 ShaderArrayという構成になっている。  
GCNにおいては1 ShaderEngine = 1 ShaderArrayと考えるためそのままだ。  
Navi14に戻す。下の画像を見ると中央に12WGP、WGPの塊すぐ上に対称的に配置された青緑のブロック2つと、青のブロック8つがあることがわかる。  
そうと確かめた訳ではないが、数からして青緑はL1キャッシュ、青はRBと推測される。  
このことから、Navi14は1 ShaderEngine、2 ShaderArray、ShaderArrayあたりのWGPは5か6という構成だと言える。  
L2キャッシュはNavi10の4MB(16x256KB)から2MB(8x256KB)となっているように見える。  
![navi14-die](/image/2019/11/04/navi14-die.webp)
まとめると  

| | ---Navi10--- | ---Navi14--- |
|:--|:--:|:--:|
|ShaderEngine|2|1|
|ShaderArray|4|2|
|WGP|20|12|
|CU|40|24|
|SP|2560|1536|
|ROP|64|32|
|TMU|160|96|
|Memory Interface|256-bit|128-bit|
|Total L1$ (KB) |512|256|
|L2$ (MB) |4|2|
|PCIe Max Lane |x16|x8|
|Transitors (B)|10.3|6.4|
Lithography| 7nm | 7nm |
|DieSize (mm2)|251|158|

と、Navi14はNavi10を概ね半分にしたような構成とわかる。  
WGPが半分より少し多いため、ダイサイズもそういった感じとなっている。（12/20=0.6、251*0.6=150.6）  
ダイ画像を見ての感想となるが、Navi10はL1キャッシュ周りにRBがもう1つ入りそうな隙間が見て取れたのに対し、Navi14ではギチギチに詰めたと感じられる。  
ちなみに、あるチップを半分にしたような構成というのは過去にも似たものがあり、Polaris10に対するPolaris11がそれにあたる。  
これもShaderEngine、L2キャッシュ、メモリインターフェース、PCIeレーンが半分になっており、同時画面出力数も5に減っている。  
さらに言えば、Polaris11は当初から16CUあると発表されていたが、製品として出たRX460では14CUのみ有効化され、RX560で16CU全てが有効になった。  
  
こうなると、Navi14もリネーム時に24CU製品が出そうな気がしてくる。  

### 個人的な話  
このようなコンパクトさに技術を注ぎ込んだようなものはロマンがあって好きだ。  
販売されたらぜひとも購入したいが、今使ってるRX 560で何の不自由もないことのもあってどうなるかわからない。  
価格の問題もある。  
対抗のGTX 1650は現在安いモデルだと1万7千円程で買える。  
祝儀価格込で2万円代後半で済むことを願っているが、GTX 1650Super次第なところもあるため不安だ。
