---
title: "姿くらます未確認Navi"
date: 2019-11-07T19:38:47+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi22", "Navi23", "GFX10" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

Navi 12_LITE/21_LITE/22/23部分のコードが削除された。  

少し話題にはなっていたものの名前だけで、憶測と妄想ばかりが飛び交っていたが、  
[Update pal from commit: 92f2b4c](https://github.com/GPUOpen-Drivers/pal/commit/39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0#diff-59fcb3e9c87bb72d45d55342763cf388)  

最新のコミットでその部分が削除された。  
[Update pal from commit: 25b4a35](https://github.com/GPUOpen-Drivers/pal/commit/76c5b997630e558158dbdd8ca24a120071068631#diff-59fcb3e9c87bb72d45d55342763cf388)  
(src/core/imported/addrlib/src/amdgpu_asic_addr.h)  
(src/core/hw/gfxip/gfx9/chip/gfx10_merged_sdma_packets.h)  

コミットメッセージで特に触れないあたりミスだったのだろうか。  

情報が名前と chip_external_rev のレンジしかないため、それ以上の話は避けるが、  
Navi22 (0x32, 0x3C)  
Navi23 (0x3C, 0x46)  
というのはこれを見るに、Naviファミリーにおいて現行世代より後のものと見てよさそうだ。  
[amdgpu_asic_addr.h - Mesa](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/addrlib/src/amdgpu_asic_addr.h#L98)  
まあNavi2xというので察しはつきそうだが。  