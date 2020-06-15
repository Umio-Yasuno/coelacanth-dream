---
title: "Renoirの構造推測"
date: 2019-11-09T00:50:45+09:00
draft: false
tags: ["Radeon", "GCN", "Raven", "GFX9", "gfx909", "Renoir", "Zen2" ]
keywords: [ "Radeon", "Renoir", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

Renoirの個人的な推測。  

![renoir-guess](/image/2019/11/09/renoir-guess_1.webp)  
自分で作っておいて言うのもあれだが、この画像にはいくつかの問題がある。  
CCDは実際のサイズに沿っておらず、適当に正方形としている。  
IO HUB Controllerが 64B/C となっているのは、第三世代Ryzenの図に従ったのだが、  
それまで 32B/Cだったものが 64B/C になった理由を私が理解していない。  
[判明した第3世代Ryzenの内部構造を大解説　AMD CPUロードマップ - ASCII.jp](https://ascii.jp/elem/000/001/882/1882171/index-3.html)  
PCIeGen3からGen4で帯域が倍になったからだろうか?  

その点を承知で見て欲しい。  

まずGPUをIOD側に統合した理由だが、  
[dcn21_resource.c](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/dcn21/dcn21_resource.c?h=amd-staging-drm-next#n153)  
この部分のコードにて、fabricclk(fclk)、ディスプレイ周りのクロック、メモリ転送クロックが1つのステートでまとめて管理されている。  
そのことから同一のSMU(System Management Unit)で管理している、=同一のダイに収まっている、と考えた。  
GPUを別ダイにしたところで、メモリが大きなボトルネックとなるためやる意味はそこまでないはずだ。  
それに別ダイにしてしまうと、GPU-IODのバスが電力を消費するため、モバイルが主戦場のAPUでやるとは思いにくい。  
そして、RDNAが選択肢にある中、APUも含めて14/12nmでも設計されたことのあるVega（GFX9）を選んだということは、  
RenoirのGPU部分が14/12nmである確率が高いと思われる。  

メモリとfclkに関しても同一部分からだ。  

メモリは .drm_speed_mts の値が最大4266.0となっており、  
これはLPDDR4Xがターゲットとする最高クロックと一致する。  
競合のモバイルCPU、IcelakeがLPDDR4Xをサポートしているため、AMDも最新APUで対応するのは当然と言えるだろう。  

.fabricclk_mhz が1600.0までとなっているが、第三世代Ryzenがデフォルトでは1800MHzまで、OCで1900MHzとかだったのを考えると抑えてると見える。  
大方省電力目的だろう。  

ここからは根拠は特になく、推測ばかりとなる。  

IOD+GFXは12nmで製造するはずだが、さすがにcIODのスペックそのままにGPUを追加したりはしないだろう。  
ある程度I/Oを削ると思われる。  
RyzenG(Raven Ridge)でも、Ryzen(Summit/Pinnacle Ridge)ではGPU用にあったx16がx8に、NVMe用にあったx4がx2になっている。（チップセットとの接続はx4のまま。）  
ただこれは製品としての差別化であり、実際はここまで減っていないかもしれない。  
ダイサイズがSummit/Pinnacleの192mm2から、Ravenでは210mm2に増加している。  
第三世代Ryzenに使われているcIODは125mm2。  
パッケージングの問題もあるため、あまり大きくはできないだろう。  
どのくらいの大きさにするかはキリがないだろうから考えないでおく。  
削りはするだろうとだけ。  

### 追伸
Aboutページにメールアドレス（画像）を追加した。  
私はやらないと言ってすぐに手のひらを反す人間だと実感した。  
