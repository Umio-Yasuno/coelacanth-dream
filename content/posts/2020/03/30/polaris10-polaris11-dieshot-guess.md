---
title: "Polaris10/Polaris11のダイ観察 & 推測"
date: 2020-03-30T00:51:08+09:00
draft: false
tags: [ "Radeon", "DieShot" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

次いで、Polaris10、Polaris11のユニット推測も、毎度お馴染み[Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/)氏によるダイショットを元に行なった。  

## Polaris10
{{< figure src="/image/2020/03/30/polaris10-dieshot.webp" title="Polaris10 推測" caption="ダイサイズ: 243.78mm<sup>2</sup><br>画像出典:[AMD@14nm@GCN\_4th\_gen@Polaris\_10@Radeon\_RX\_470@1622\_M60J5.0… | Flickr](https://www.flickr.com/photos/130561288@N04/43550225514)" >}}

ようやく自分なりの色分けとかやり方がわかってきた気がする。  
それはともかく、[GPUOpen-Drivers/pal: Platform Abstraction Library](https://github.com/GPUOpen-Drivers/pal)からの情報では[^1]、

 * 4-ShaderEngine
 * ShaderEngineあたりのRBE(RenderBackend)数は4基
 * 〃 のCU(ComputeUnit)数は9基
 * TCC(L2cache)ブロック数は8基(ブロックあたり256KB、計2MB)

[^1]: <https://github.com/GPUOpen-Drivers/pal/blob/e642f608a62887d40d1f25509d2951a4a3576985/src/core/os/nullDevice/ndDevice.cpp#L716>

[RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeondisplayhardware)からの情報では、

 * Display Controller数は6基

となっている。L1 I$(32KB)/K$(16KB)は3CUで共有する構成となっていた。  

## Polaris11
{{< figure src="/image/2020/03/30/polaris11-dieshot.webp" title="Polaris11 推測" caption="ダイサイズ: 129.40mm<sup>2</sup><br>画像出典: [AMD@14nm@GCN\_4th\_gen@Polaris\_11@Radeon\_RX\_460@1628\_NAA2Y.1… | Flickr](https://www.flickr.com/photos/130561288@N04/42217737082)" >}}
129.40mm<sup>2</sup>と小型なダイだけあって、内部はより入り組んだ印象を受けた。  

[GPUOpen-Drivers/pal: Platform Abstraction Library](https://github.com/GPUOpen-Drivers/pal)からの情報では[^2]、

 * 2-ShaderEngine
 * ShaderEngineあたりのRBE(RenderBackend)数は2基
 * 〃 のCU(ComputeUnit)数は8基
 * TCC(L2cache)ブロック数は4基(ブロックあたり256KB、計1MB)

[^2]: <https://github.com/GPUOpen-Drivers/pal/blob/e642f608a62887d40d1f25509d2951a4a3576985/src/core/os/nullDevice/ndDevice.cpp#L734>

[RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeondisplayhardware)からの情報では、

 * Display Controller数は5基

となっている。L1 I$(32KB)/K$(16KB)は4CUで共有する構成となっていた。  

## Polaris10とPolaris11の比較
{{< figure src="/image/2020/03/30/compare-polaris10-polaris11.webp" title="L : Polaris10 / R : Polaris11" caption="Process: GF 14nm / GF 14nm<br>DieSize: 243.78mm<sup>2</sup> / 129.40mm<sup>2</sup><br>画像出典:[AMD@14nm@GCN\_4th\_gen@Polaris\_10@Radeon\_RX\_470@1622\_M60J5.0… | Flickr](https://www.flickr.com/photos/130561288@N04/43550225514)<br>,[AMD@14nm@GCN\_4th\_gen@Polaris\_11@Radeon\_RX\_460@1628\_NAA2Y.1… | Flickr](https://www.flickr.com/photos/130561288@N04/42217737082)" >}}
同プロセスで製造されているのだから、当然各ユニットのサイズもほぼ変わらず。  
特筆することと言えば、L1 I$/K$を共有するCU数が Polaris10: 3基、Polaris11: 4基と違うことから、微妙に配置を変えていたことだろうか。高さは変わっておらず、内部だけの変更で対応していた。  
ただそれは、3CU分に詰め込められるものを4CU分に引き伸ばしているとも取れ、そういった点でもGCNアーキテクチャには無駄があったと言える。  
RDNAアーキテクチャでは、共有する数が2CU(1WGP)と固定された。これにより設計をわずかでも弄る手間が減ったのではないかと思う。  

Polaris11とI/OやL2$の規模は変わらないが、CU数を減らし、ダイサイズも小さくなっているPolaris12も比較してみたかったが今回は無理だった。  
どのように配置を工夫したのかはとても興味深いが……
