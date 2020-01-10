---
title: "消えたV340、継ぐのは誰か？"
date: 2019-11-11T12:42:04+09:00
draft: false
tags: [ "Radeon", "GCN", "GFX9", "gfx900", "Vega10", "gfx908", "Arcturus" ]
keywords: [ "Radeon", "Vega10", "Arcturus" ]
categories: [ "Hardware", "GPU", "AMD" ]
---

こんなタイトルにしておきながら小松左京「継ぐのは誰か？」を未だ読んだことがない。  
載ってる日本SF傑作選2を買いはしたけれど。  

さて、正確にいつからかはわからないがAMDのProduct Resource CenterからRadeon Pro V340が消えた。  
<https://www.amd.com/en/products/specifications/professional-graphics>  
V340の主な用途だった仮想用途のページからも消えるという念入り具合。  
<https://www.amd.com/en/graphics/workstation-virtual-graphics>  
最近消えたもので言えば、[MI60](https://www.amd.com/en/product/8291)もだが、それはまだスペックが残っている。  

まずV340が何なのかと言うと、Vega10チップを64CU中56CUを有効化し、HBM2を2スタック（2048-bit）16GBを載せたパッケージ2つを1枚のカードに収めたものだ。  
もっと簡単に言えば、Vega56のHBM2を16GBにして2つ載せたDualGPU。  
Typical Board Powerは<300W、冷却はパッシブでラックサーバーに組み込むことを前提としていたカードと思われる。  
最大でカードあたり32の仮想マシンをサポートしていた。  
DualGPU、VegaということでInfinity Fabricによる接続を期待したが、特にアピールされてないあたりそうではないのかもしれない。  
仮想用途ということでコヒーレントを取る必要もそこまでなかったのだろうか。  

このV340が消えたということは何かしらの後継製品がありそうだが、  
<https://www.amd.com/en/graphics/workstation-virtual-graphics>  
このページに製品が載ってないあたり、はっきりとは決まってないのだろう。  
[MI50](https://www.amd.com/en/products/professional-graphics/instinct-mi50#product-specs)が仮想用途でも使えるということで発表されてはいるので、一応の後継とみなすことはできる。  
[AMD Radeon Instinct MI60 Session 5 Nov 6 2018 Next Horizon - Youtube](https://www.youtube-nocookie.com/embed/m0h6-VfH3Xo?start=140)  

さらにはV340が姿を消したのと同時期に、MI50(32GB)が追加されている。  
メモリ容量だけならV340と同等だが、単純なCU数では7168 vs 3840と半分より少し多いくらいで、  
サポートする仮想マシン数も恐らくV340よりは少ないはずだ。（あって、1GPUだから16VMsとかのはず）  
そうなると後継というより本命までの一時しのぎのようにも思える。  

その本命が誰になるか。  
今の所製品はまだ出ておらず、VF用のDeviceIDが確認されているのはNavi12とArcturusがある。  
このどちらかではArcturusが本命になると思われる。  
CU数ではV340を越えてくると現情報では考えられ、VCNも2つ搭載している。  
（仮装用途ではクライアントに動画で画面を送信するためHWによるエンコード支援が必要となる。）  
ただ[3Dエンジン無し](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=2065aa5494e465c0efb20fffd607db6cbfa74067)でそれが可能なのかどうかまでは知識が及ばずわからない。  

Navi12を使った製品はMI6/8のようなTBP150/175Wを置き換えるものと予想される。  
さすがにもうGDDR6でDualGPUとかやると配線や基板が厳しいからやらない、はず。  

### 追伸
Youtubeの埋め込みはページが重くなりそうなのでやらないことにする。  