---
title: "Vega12を搭載したChromebookが開発中か"
date: 2020-04-12T12:47:09+09:00
draft: false
tags: [ "Vega12", "Chromebook" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

ChromiumOSへのパッチから、ボード *Mushu* がdGPUに *Vega12* を搭載することがわかった。[^4]  

[^4]: [mushu: enable amdgpu (Ic7ce7b60) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/2117044)

## Vega12
GFX Coreは[GFX9](/tags/gfx9)であり、名の通り正真正銘 Vega世代のGPU。  
GPUの構成としては、4-ShaderEngine、総CU数 20基、総RBE数 8基(32 ROP)、L2cache 2MB[^1]。  
メモリにはHBM2を1スタック(1024-bit)使用している。  
[Vega10](/tags/vega10)よりも小規模とはいえ確実に *GFX9/Vega* 世代であることと未だコンシューマ向けGPUでは採用例が少ないHBM2を搭載することから一部で注目と期待を浴びていたが、現在[Vega12](/tags/vega12)ベースの製品を組み込んでいるのはMacBook Pro 2018年モデルのみとなっている。  

| Vega12[^1][^2] | |
| :--- | :---: |
| ShaderEngine | 4 |
| Total CU / SP | 20 / 1280 |
| Total TMU | 80 |
| Total ROP | 32 |
| Total L2$ | 2 MB |
| Memory Type<br>/ Bus Width | HBM2<br>/ 1024-bit |
| GFX Core | GFX9 |
| GPU ID | gfx904 |
| DCE | v12.0 |
| UVD | v7.0 |
| VCE | v4.0 |
| CMOS | 14nm |

[^1]: [pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L854)
[^2]: [RadeonFeature](https://www.x.org/wiki/RadeonFeature/)

## Mushu
*Hatch* をベースボードとするIntel CPU搭載のボード。[^3]  
CPUに関しては、PCHが *Cannon Lake PCH* とされることから *Coffee Lake* か *Whiskey Lake* と考えられる。メモリにDDR4-2666 16Gを想定していることから前者の可能性のが高い。[^5]  
その他判明している仕様は、ディスプレイ解像度 2400x1600(3:2)[^9]、デュアルファン[^6]。  
バッテリーから、DellのChromebookボードと思われる。[^7]  

[^3]: [mb/google/hatch: Add mushu variant (I81b5bf96) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/1934468)<br><https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/1934468/9/src/mainboard/google/hatch/variants/mushu/overridetree.cb>
[^9]: [bmpblk: Add mushu board (Ie0bd4ba4) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/bmpblk/+/1986097)
[^5]: [mb/google/hatch: Add mushu variant (I81b5bf96) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/1934468)<br><https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/1934468/9/src/mainboard/google/hatch/variants/mushu/overridetree.cb>
[^6]: [Mushu : add dual fan control by EC (I41ba92ad) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2073740)
[^7]: [board/mushu/battery.c - chromiumos/platform/ec - Git at Google](https://chromium.googlesource.com/chromiumos/platform/ec/+/refs/heads/master/board/mushu/battery.c#35)

## 嬉しいニュース
*Vega12* には前から多数のDeviceID、SKUが確認されていたが[^8]、上述したように現在世に出てるのはMacBook Pro 2018に搭載された **Radeon Pro Vega 16** 、**Radeon Pro Vega 20** のみだ。  
それらがまた *Mushu* に搭載されることとなるか、それとも隠れていた *Vega12* ベースの製品が出てくるかはまだわからないが、  
*Vega12* がMacBook Proだけに終わらない、ということは個人的に嬉しいニュースだ。

[^8]: [AMDGPU Database: Device ID/ Revision ID/ Product Name | Coelacanth's Dream](/posts/2019/12/30/did-rid-product-matome-p2/#vega12-gfx904)
