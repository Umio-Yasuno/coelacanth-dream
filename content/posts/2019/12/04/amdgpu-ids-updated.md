---
title: "drm amdgpu.idsがアップデートされる"
date: 2019-12-04T11:50:25+09:00
draft: false
tags: [ "Radeon" ]
keywords: [ "Radeon", "AMDGPU" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

まだマージリクエストの段階ではあるけど変更点まとめ。  
[amdgpu: add new marketing names from 19.30 - Mesa/drm](https://gitlab.freedesktop.org/mesa/drm/merge_requests/33/diffs)  

新規に追加されたGPU名は以下。  

 * Radeon (TM) Pro V320
 * AMD Radeon (TM) Pro WX 3200 Series
 * AMD Radeon RX 5700 XT  50th Anniversary
 * AMD Radeon RX 5700 XT
 * AMD Radeon RX 5700
 * AMD Radeon RX 5500M
 * AMD Radeon RX 5500

AMD Radeon (TM) Pro WX 3200 Seriesは14nm Polarisを使った製品で、（ちょうど10CUであることからPolaris12と思われる。Polaris11を使った製品にはWX4100がある。）  
1スロット/ロープロファイル/補助電源無しでありながら、VRAM 4GB、Mini-DPx4を持つ非常に小型かつプロ向けらしいGPUだ。  

Navi系に関しては以前、個人的にまとめた。  
[AMDGPUのDID、RID、Productをびみょうにまとめた](/posts/2019/11/22/did-rid-product-matome)  
ただそれから追加されたものがあり、  

 > 731F,	C5,	AMD Radeon RX 5700 XT

がそれにあたる。  
RX 5700 XTには元々731F:C1があったが、リビジョンがC5となったことの具体的な違いは不明。  

それとRadeon Pro W5700は今回追加されなかった。  
[Radeon Pro W5700が発表](/posts/2019/11/20/radeon-pro-w5700)  
USB-Cに関するファームウェアも特に追加されてないが、画面出力はともかくUSBコントローラーとしての機能は使えるのだろうか。  
今の所Windows10向けのドライバしかないため、これに関しては後に追加されるかもしれない。  
[Radeon™ Pro W5700 Drivers & Support - AMD](https://www.amd.com/en/support/professional-graphics/radeon-pro/radeon-pro-w5000-series/radeon-pro-w5700)  

そして個人的に謎なのがRadeon (TM) Pro V320。  
名前からしてV340より下位の仮想向けGPUっぽいが、公式には発表されていない。（[AMD Drivers and Support - AMD](https://www.amd.com/en/support)から探ってもV340しか表示されない。）  
仮想向けだとしても、ホスト側から認識される製品名のみで、ゲスト側から認識される際の末尾にMxGPUが付いた名前は追加されていないのが不自然に思える。  
だがTechPowerUPのGPU Databaseに項目が存在することから、名前だけは以前から確認されていたらしい。  
[AMD Radeon Pro V320 - TechPowerUP](https://www.techpowerup.com/gpu-specs/radeon-pro-v320.c3270)  

さらに言うと、Radeon Instinct MI25x2が何なのかとか、そこからRadeonが消える以外でどこか違うのかとか、V340にLが付くことで何が変わるのかとか、  
まあVega10に関して私が知らないことは多い。  

V340Lはゲスト側から複数のV340を扱うときの名前だったりするのだろうか？  



