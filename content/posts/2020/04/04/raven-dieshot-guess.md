---
title: "Raven Ridgeのダイ観察 & 推測"
date: 2020-04-04T06:11:26+09:00
draft: false
tags: [ "Radeon", "DieShot" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU"]
noindex: false
---

*Zen* + *Vega* なAPU、*Raven Ridge (以下 Raven)* のダイ観察 & ユニット推測。  
画像出典は例の通り[Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/)氏により撮影され、パブリック・ドメインで公開されているダイショット。  

それと今回、CSSだけで画像の一部分を拡大する方法を用いてみたが、頑張ってはみたものの、いかんせん調整が難しく、一部環境ではおかしく見える可能性があることをご容赦願いたい。  

{{< fig src="/image/2020/04/04/raven-dieshot.webp" title="Raven 推測 *全体*" caption="ダイサイズ: 208.84mm<sup>2</sup><br>画像出典: [AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G@YD2200C5M… | Flickr](https://www.flickr.com/photos/130561288@N04/39716562275)" >}}
AMDの発表ではダイサイズが 209.78mm<sup>2</sup>だが、誤差の範囲だろう。[^1]  
GPUが統合されたことで機能は増えたが、*Zeppelin*[^2] が2CCX(8-Core)構成だったのに対し、*Raven* では1CCXとし、またCCXあたりのL3キャッシュを4MBに減らしている。  
Threadripper、EPYCの製品で活かされたパッケージ内の他ダイ接続用のインターフェイスも、PCI Expressと共用である他ソケットへの接続インターフェイスも排されたことで、*Raven* は *Zeppelin* のダイサイズ: 213mm<sup>2</sup>よりもほんのわずかに小さいダイとなっている。  

[^1]: [Delivering a new level of Visual Performance in and SoC – AMD Raven Ridge APU](https://www.hotchips.org/hc30/1conf/1.05_AMD_APU_AMD_Raven_HotChips30_Final.pdf#page=4)
[^2]: Ryzen /Threadripper 1000シリーズ、EPYC 7001シリーズに採用されているダイ。CPUアーキテクチャは *Zen* 、製造プロセスはGF14nm FinFet。

{{< fig src="/image/2020/04/04/raven-dieshot.webp" imgstyle="width: 48vmax; height: 64vmax; object-position: 100% 16%; object-fit: cover" title="Raven 推測 *ダイ右側 ShaderEngine付近*" caption="画像出典: [AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G@YD2200C5M… | Flickr](https://www.flickr.com/photos/130561288@N04/39716562275)" >}}
GPUの規模は [pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal)[^3] によると、

 * 1-ShaderEngine
 * RBE(RenderBackend)数は2基
 * 総CU数は11基
 * TCC(L2cache)ブロック数は4基(ブロックあたり256KB、計1MB)

[^3]: <https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L888>

HotChips30での発表資料ではRBE数が4基であるように書かれているが[^4]、後からいくらでも修正可能なソースコードが最新の状態で、*RBEは2基* 、とあるためそちらを信用する。  
ダイショットからこれとこれがRBE、と示せればいいのだが、*Raven* のようなSoCはダイサイズを抑えるため、内部のユニットが複雑に配置されており、はっきりと判別するのは難しかった。  

[^4]: [Delivering a new level of Visual Performance in and SoC – AMD Raven Ridge APU](https://www.hotchips.org/hc30/1conf/1.05_AMD_APU_AMD_Raven_HotChips30_Final.pdf)<br>&emsp;&emsp;<https://www.hotchips.org/hc30/1conf/1.05_AMD_APU_AMD_Raven_HotChips30_Final.pdf#page=5>

L1 I$(32KB) /K$(16KB)は2CU、または3CUで共有する構成となっていた。CU 11基に対してL1 I$/K$ 4基ということから、共有するCU数の基本単位は3基だろう。  
この3基で共有、というのは *Vega10* 、 *Vega20* のダイショットからも見て取れた。[^5]  
*GFX9* で共通した設計部分なのだろうか。もしかしたら *Raven* よりもCPU、GPU、I/O、全体の規模を減らし、ダイをより小さくした *Raven2* のCU数が3基であることの要因かもしれない。  

[^5]: [Vega10/Vega20のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/03/24/vega10-vega20-dieshot-guess/)

{{< fig src="/image/2020/04/04/raven-dieshot.webp" imgstyle="width: 48vmax; height: 32vmax; object-position: 64% 30%; object-fit: none" title="Raven 推測 *ダイ中央*" caption="画像出典: [AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G@YD2200C5M… | Flickr](https://www.flickr.com/photos/130561288@N04/39716562275)" >}}
ダイ中央に配置された4基のユニットだが、これはダイ内の各ユニットの接続に使われている *Infinity Fabric* の *Transport Layer Switch* と考えている。  

最初は4基であることからディスプレイコントローラだと思っていたが、他のGPUのダイショットを見るに、それらしきユニットは画像のようにSRAMをそう多くは内包していない。  
HotChips30の発表資料に *Trasport Layer Switch* が4基存在するような図表があり、そういった高速なデータ通信に関連するユニットであれば、バッファだろうSRAMを多く含むのは自然なはずだ。  

{{< fig src="/image/2020/04/04/raven-dieshot.webp" imgstyle="width: 100%; height: 12vmax; object-position: 40% 100%; object-fit: cover" title="Raven 推測 *ダイ下部 I/O*" caption="画像出典: [AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G@YD2200C5M… | Flickr](https://www.flickr.com/photos/130561288@N04/39716562275)" >}}
[RadeonFeature](https://www.x.org/wiki/RadeonFeature/)から、

 * Display Controller数は4基(最大映像出力数: 4)

*Raven* ベースの製品である Ryzen Embedded V1000シリーズの仕様から、[^6]  

 * 最大4つのUSB 3.1(10Gbps)
 * 2つのUSB Type-C、DisplayPort出力とPower Deliveryに対応
 * 最大2つのSATAポート
 * 最大16レーンのPCIeGen3

ということがわかっており、それを元にしている。  
左から、USB 3.1インターフェイス(以下I/F) 4基、ディスプレイI/F 2基とUSB Type-C I/F2基[^8]、PCIeGen3 2-LaneとSATAの共用I/F 2基、ディスプレイI/F 2基、と推測した。  
USB Type-C I/Fの上にあるユニットが、他のGPUで見られるディスプレイコントローラの形状、構造に近いため、そうであると考えた。ディスプレイI/Fとの距離も近い。  

USB Type-Cに関しては似た配置が *Navi10* 、*Navi14* でも見られる。[^7]  

[^6]: <https://www.amd.com/system/files/documents/v1000-family-product-brief.pdf>
[^7]: [Navi10のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/01/22/navi10-dieshot-and-guess/)<br>&emsp;[Navi14のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/01/26/navi14-dieshot-and-guess/)
[^8]: <https://www.hotchips.org/hc30/1conf/1.05_AMD_APU_AMD_Raven_HotChips30_Final.pdf#page=15>

## 余談
いつか *Raven2* と *Renoir* のダイショットも見てみたいが、自分はそのための技術を持ってないため、ひたすらに待つ。  
*Renoir* は総CU数が8基だが、GFX9系列であることを考えるとL1 I$/K$は3基だろうか。後は *Raven2* のRBE数が1基であることから、どれがRBEなのか特定したり、*Transport Layer Switch* の数は *Raven* と *Raven2* とで同じなのか、後は何と言ってもどのようにユニットを詰め込んでいるのかと、確かめてみたいことはたくさんある。  
