---
title: "Intel PCH Matome"
date: 2019-12-02T01:32:23+09:00
draft: false
tags: [ "PCH" ]
keywords: [ "Intel", "PCH" ]
categories: [ "Hardware", "Chipset", "Intel" ]
---

Linux Kernelを元にPCHをまとめてみた。  
GPUと同様に、「末尾」のLakeは省略している。  
最左列は見にくいため、略称で対応させて見る方がわかりやすいと思う。  

| Intel PCH | a.k.a. | Memo | Pair | Device ID |
| :--- | :---: | :---: | :---: | ---: |
| Sunrise Point | **SPT** | | Skylake / Kaby | 0xA100 |
| Sunrise Point LP | **SPT_LP** | | Skylake / Kaby / Coffee(Amber) | 0x9D00 |
| Kaby Lake PCH | **KBP** | compat SPT | Skylake / Kaby / Coffee(Amber) | 0xA280 |
| Cannon Lake PCH | **CNP** | | Cannon / Coffee | 0xA300 |
| Cannon Lake LP PCH | **CNP_LP** | | Cannon / Coffee (Whiskey) | 0x9D80 |
| Comet Lake PCH | **CMP** | compat CNP, Mobile | Coffee / (Comet) | 0x0280 |
| Comet Lake 2 PCH | **CMP2** | compat CNP, Desktop | Coffee / (Comet) | 0x0680 |
| Comet Lake V PCH | **CMP_V** | base KBP, compat SPT | Coffee (Amber / Comet) | 0xA380 |
| Ice Lake PCH | **ICP** | | Ice | 0x3480 |
| Mule Creek Canyon PCH | **MCC** | | Elkhart | 0x4D00 |
| Tiger Lake LP PCH | **TGP** | | Tiger | 0xA080 |
| Tiger Lake LP 2 PCH | **TGP2** | | Tiger | 0x4380 |
| Jasper Lake PCH | **JSP** | | Elkhart | 0x4D80 |
| Jasper Lake 2 PCH | **JSP2** | | Elkhart | 0x3880 |

参考: [intel_pch.c - drn-intel](https://cgit.freedesktop.org/drm-intel/tree/drivers/gpu/drm/i915/intel_pch.c)  、 [intel_pch.h - drn-intel](https://cgit.freedesktop.org/drm-intel/tree/drivers/gpu/drm/i915/intel_pch.c)  

ソース内には「Comet Lake PCH」ではなく「CometPoint」と書かれていたりするが、下記リンクのようなことを言って修正しているのだから前者が正しいと思い、表ではそう記述した。  
[Fix PCH names for KBP and CNP.](https://cgit.freedesktop.org/drm-intel/commit/drivers/gpu/drm/i915?id=23247d715d3cf679cac24d1c4de2d76774a2a495)  

<br>
読み込んだ感想としては、Coffeeがややこしい。  
Whiskey（ウイスキー）、Amber（琥珀）、Comet（彗星）が入り混じるコーヒーとは遠くかけ離れた存在と化している。  
<https://gitlab.freedesktop.org/mesa/drm/blob/master/intel/i915_pciids.h#L527>  
PCHではそこへさらにCannonが撃ち込んでくる。  

SPT_LP、KBP、CNP、CNP_LP、CMP、CMP2、CMP_VのペアとなるGPUに、どれもCoffeeがあるが、  
そのCoffeeがどれになるかは、埋めはしたもののあまり自信が持てない。  
マーケティングでの名前がわかればまだやりようがあるが、Z390がCNPだとしか。  
H310Cのような22nmでありながら、Coffee Lakeに対応しているチップセットもあるため、それでもやりにくそうな気がする。  

そんな中で、CNP_LPのCoffeeをWhiskeyとしたのは下記リンクより。  
[Whiskey LakeとAmber Lakeの正体 - PC Watch](https://pc.watch.impress.co.jp/docs/column/ubiq/1140369.html)  

 > 今回のWhiskey LakeのPCHは、Skylake、Kaby Lake、Kaby Lake Refreshまで使われてきたSkylake世代で導入されたPCHではなく、その後継でCNL-PCHと呼ばれる、Cannon Lake用に開発された新しい14nmプロセスルールで製造されているPCHになる。

また、

 > また、IntelはAmber LakeのPCHは、最新のPCH(つまりCNL-PCH)ではなく、1世代前のPCH(つまりはSKL-PCH)であることをほぼ認めている。

ともあることから、SPT_LP、KBPのCoffeeはAmberなのだろうか？  
そうなるとCMP_VはAmberにも対応するということになる。   
CMP、CMP2はCNPと互換性があるとのことから、普通のCoffeeにも対応すると考えられる。  

<br>
パズルをやっている気分。  

Coffeeを抜ければ多少なりともすっきりすると思いたいが、既にElkhartから嫌な予感がする。  
