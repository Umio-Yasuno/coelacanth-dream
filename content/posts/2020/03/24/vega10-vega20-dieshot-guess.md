---
title: "Vega10/Vega20のダイ観察 & 推測"
date: 2020-03-24T20:57:11+09:00
draft: false
tags: [ "Radeon", "Vega10", "Vega20", "DieShot", "gfx900", "gfx906" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

いつかやろうと思っていて放置していた[Vega10](/tags/vega10)、[Vega20](/tags/vega20)のユニット推測を、毎度お馴染み[Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/)氏によるダイショットを元に行なった。  

## Vega10
{{< figure src="/image/2020/03/24/vega10-dieshot-guess.webp" title="Vega10 推測" caption="ダイサイズ: 509.73mm<sup>2</sup><br>画像出典: [AMD@14nm@GCN\_5th\_gen@Vega10@Radeon\_RX\_Vega\_64@ES-Sample@\_\_… | Flickr](https://www.flickr.com/photos/130561288@N04/40482186211/)" >}}

元画像は色付きであり、そのままだと字が非常に読みにくくなるため画像に補正を掛けた。それでもまだぼやけて分かりにくい所があるが。  

[GPUOpen-Drivers/pal: Platform Abstraction Library](https://github.com/GPUOpen-Drivers/pal)からの情報では[^1]、

[^1]: <https://github.com/GPUOpen-Drivers/pal/blob/e642f608a62887d40d1f25509d2951a4a3576985/src/core/os/nullDevice/ndDevice.cpp#L835>

 * 4-ShaderEngine
 * ShaderEngineあたりのRBE (RenderBackend) 数は4基
 * 〃 のCU (ComputerUnit) 数は16基
 * TCC (L2 cache) ブロック数は16基 (ブロックあたり256KB、計4MB)

[RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeondisplayhardware)からの情報では、

 * Display Controller数は6基

となっており、ダイショットを元にした推測も基本それに沿っている。  
見にくくなってしまったが、右上 6基が *Display PHY/Interface* 、その下の6基が *Display Engien* としている。  

### 疑問点
AMD GCNアーキテクチャでは、最大4CUで I$(Instruction Cache) 32KB と K$(Scalar Cache) 16KBを共有している。  
*Vega10* では ShaderEngineあたり16CUということから、最大の4CUで共有し、I$K$のブロックは計4基となるはずだが[^2]、  
ダイショットでは6基あるように見えた。  
単なら私の見間違い、勘違いか、歩留まり向上策の1つか。  

それと *Geometry Processor* と *HBM2 Controller* の間にバッファかキャッシュらしきユニットがあるが、正体は不明。  
*HBM2 Controller* がある反対側にもあるため、それのバッファとは違うように思う。  
近くに配置されている *RBE* か *Geometry Processor* に関連した、ラスタライザ系のユニットだったりするのだろうか？  

[^2]: <https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf#page=27>

## Vega20
{{< figure src="/image/2020/03/24/vega20-dieshot-guess.webp" title="Vega20 推測" caption="ダイサイズ: 330.93mm<sup>2</sup><br>画像出典: [AMD@7nm@GCN\_5th\_gen@Vega20@Radeon\_VII@\_-\_@\_\_\_DSCx2\_polysil… | Flickr](https://www.flickr.com/photos/130561288@N04/48243282516/)" >}}

[GPUOpen-Drivers/pal: Platform Abstraction Library](https://github.com/GPUOpen-Drivers/pal)からの情報では *Vega10* と特に変わらず、[^3]

[^3]: <https://github.com/GPUOpen-Drivers/pal/blob/e642f608a62887d40d1f25509d2951a4a3576985/src/core/os/nullDevice/ndDevice.cpp#L871>

 * 4-ShaderEngine
 * ShaderEngineあたりのRBE (RenderBackend) 数は4基
 * 〃 のCU (ComputerUnit) 数は16基
 * TCC (L2 cache) ブロック数は16基 (ブロックあたり256KB、計4MB)

[RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeondisplayhardware)からの情報ではこちらも、

 * Display Controller数は6基

となっており、ダイショットを元にした推測も基本それに沿っている。  
推測をしている中で気付いたが、GPUのコア部、ShadeEngine内の配置はまったく *Vega10* と同じだった。I$K$ が6基あるように見える点も。  
そういった発見があるからダイショットを眺めるのは楽しい。  

感想としては、*Vega10* と比べてコア部こそ微細化の効果が見られるが、他GPUと接続するための *Infinity Fabric (XGMI)* 、さらに *HBM2* 2スタック分のインターフェース追加等、主に微細化が効きにくい I/O のパッド部にダイサイズが引っ張られて大きくなっているように感じた。  
[Navi10](/tags/navi10) のダイショットからはCUからWGPへの基本単位変更もあり、*Vega20* よりも詰めている印象を受ける。  
[Navi10のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/01/22/navi10-dieshot-and-guess/)  
設計の最適化に掛けた時間にも結構な違いがあったりするのだろうか？  

## Vega10とVega20の比較
{{< figure src="/image/2020/03/24/compare-vega10-vega20.webp" title="L: Vega10 / R: Vega20" caption="Process: GF 14nm / TSMC 7nm (N7)<br>DieSize: 509.73mm<sup>2</sup> / 330.93mm<sup>2</sup><br>CU Size(推定): 3.744mm<sup>2</sup> / 1.592mm<sup>2</sup> " >}}

*Vega20* のShaderEngineのサイズが *Vega10* のほぼ半分になっているところに、TSMC 7nm(N7)移行の効果が見て取れる。  
CUのサイズもほぼ半分となっていた。  
それに対し、画像を見てもわかりやすいが、*HBM2 PHY*、*PCIeGen4 4-Lane PHY* 等、I/Oの物理層となる部はほとんどサイズが変わっていない。  

## 余談
*GCN/GFX9* 系であり、8-ShaderEngine構成、AGPR増設、8-SDMAといった変更点がコードの記述から確認されている [Arcturus](/tags/arcturus) ではどうなるかは気になる所だが、高コスト&一般向けではないことから、公式以外からのダイショット公開は期待できそうにない。  
*Arcturus* が8-ShaderEngine / 最大128(120)CUだとして、内部構成はどのようになるのだろう？  
I/OのPHYを抜いた *Vega20* をそのまま縦に積む形になのだろうか。その場合、L2$、RBE(あるのかはまだ不明だが)の配置と、フロントエンド部(ACE/HWS/GDS/Geometry等)の規模がどうなるか。  
とにもかくにも楽しみだ。  

それとお金を稼ぐ手段として、AMDGPUの解説をまとめた同人誌的なものを出す計画を思い付いた。解説範囲はGFX9、GFX10あたり、頑張ってGFX8も、とかだろうか。さすがにGCNアーキテクチャの最初からとなると新しく情報を集めなければならないため、時間がかかり過ぎる。  
実を言うと、興味を持ち始めたのはそう昔ではなく、知識や情報量に自信が持てるのはGFX9からだ。ハードウェア界隈ではまだまだ新人。  

それはともかく、やるとしたら電子本限定。自分一人ではミスや誤字が怖いが、協力を持ち掛けるための人脈が無い。報酬等の心配もある。  
まだ計画段階であるため、何らかの進展があったらまた報告すると思う。  
