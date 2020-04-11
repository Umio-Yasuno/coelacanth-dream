---
title: "Ryzen搭載Chromebookの情報を整理する"
date: 2020-04-10T20:40:19+09:00
draft: true
tags: [ "Raven", "Raven2" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

## 目次 {#table-of-content}

 * [Ryzen APU](#ryzen-apu)
 	* [Picasso](#picasso)
	* [Raven2 /Dali /Pollock](#raven2)
 * [Board](#board)
	* [Zork](#zork)
 	* [Ezkinil](#ezkinil)
	* [Morphius](#morphius)
	* [Trembyle](#trembyle)
	* [Dalboz](#dalboz)

## Ryzen APU {#ryzen-apu}
Chromebookに搭載されるRyzen APUには以下のSKUが確認されている。

 * Ryzen 7 3700C (Picasso)
 * Ryzen 5 3500C (Picasso)
 * Ryzen 3 3250C (Raven2/Dali)
 * Athlon Silver 3050C (Raven2/Dali)
 * Athlon Gold 3150C (Raven2/Dali)

*Ryzen 7 3700C* 、*Ryzen 5 3500C* に関しては DeviceID:RevisionID が同じ数字セグメントを持つ U-Series、 *Ryzen 7 3700U* 、*Ryzen 5 3500C* と一致し、先日確認されたベンチマーク結果を見てもベースクロックが一致するため、仕様としては特に変わらないものと推察される。[^4][^5]  

[^4]: <https://twitter.com/TUM_APISAK/status/1248112071641726977><br>&emsp;&emsp;[Google zork - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15363201)

[^5]: <https://twitter.com/TUM_APISAK/status/1248132832796401670><br>&emsp;&emsp;[Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/1653443)

 * [Rework map_oprom_vendev to add revision check and mapping (I2978a569) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455)
	* <https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/4/src/soc/amd/picasso/northbridge.c>

U-Series を元にした仕様推測、というかコピペ

 * Ryzen 7 3700C[^7]
	* CPU: 4-Core/8-Thread, Base 2.3GHz, Boost 4.0GHz
	* GPU: 10CU, 1400MHz
 * Ryzen 5 3500C[^8]
	* CPU: 4-Core/8-Thread, Base 2.1GHz, Boost 3.7GHz
	* GPU: 8CU, 1200MHz
 * Ryzen 3 3250C[^9]
 	* CPU: 2-Core/4-Thread, Base 2.6GHz, Boost 3.5GHz
	* GPU: 3CU, 1200MHz
 * Athlon Silver 3050C[^10]
 	* CPU: 2-Core/2-Thread, Base 2.3GHz, Boost 3.2GHz
	* GPU: 2CU, 1100MHz
* Athlon Gold 3150C[^11]
	* CPU: 2-Core/4-Thread, Base 2.4GHz, Boost 3.3GHz
	* GPU: 3CU, 1000MHz

[^7]: [AMD Ryzen™ 7 3700U Mobile Processor | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-3700u#product-specs)
[^8]: [AMD Ryzen™ 5 3500U Mobile Processor | AMD](https://www.amd.com/en/products/apu/amd-ryzen-5-3500u#product-specs)
[^9]: [AMD Ryzen™ 3 3250U | AMD](https://www.amd.com/en/products/apu/amd-ryzen-3-3250u#product-specs)
[^10]: [AMD Athlon™ Silver 3050U | AMD](https://www.amd.com/en/products/apu/amd-athlon-silver-3050u#product-specs)
[^11]: [AMD Athlon™ Gold 3150U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u#product-specs)

### Picasso {#picasso}
GF 12nmプロセスで製造され、CPUは *Zen/+アーキテクチャ* で最大4-Core/8-Thread、L3cache 4MB、  
GPUは *Vegaアーキテクチャ ([gfx902](/tags/gfx902))* で最大11CU、2RBE(8ROP)、L2cache 1MBの規模となるAPU。[^6]

[^6]: <https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L888>

### Raven2 /Dali /Pollock {#raven2}
GF 14nmプロセスで製造され、CPUは *Zenアーキテクチャ* 最大2-Core/4-Thread、L3cache 4MB、  
GPUは *Vegaアーキテクチャ ([gfx909](/tags/gfx909))* 最大3CU、1RBE(4ROP)、L2cache 512KBの規模となるAPU。[^12]  
*gfx909* は *gfx902* で確認されていたバグを修正したもの。  
I/O部の規模は *Picasso* より小さい。  

名前が3つあるように書いたが、個人的にはまず[Raven2](/tags/raven2)が基本としてあり、[Dali](/tags/dali)と[Pollock](/tags/pollock)はそのマイナーバージョンという認識でいる。  

[^12]: <https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L905>

## Board {#board}
### Zork {#zork}
*Zork* はChromebookにおけるRyzen APUのリファレンスボードの立ち位置にあり[^2]、*Zork* を元にした異なるボードが複数存在する。  
*Ezkinil* 、*Morphius* 、*Trembyle* 、*Dalboz* がそれにあたる。  
ちなみにそれらコードネームは、1980年から始まるコンピュータゲーム、Zorkシリーズに登場するキャラクターから取られていると思われる。  

 * [Ezkinil](https://www.thezorklibrary.com/history/ezkinil.html)  
 * [Morphius | Zork Wiki | Fandom](https://zork.fandom.com/wiki/Morphius)  
 * [Wizard Trembyle | Zork Wiki | Fandom](https://zork.fandom.com/wiki/Wizard_Trembyle)  
 * [Dalboz of Gurth | Zork Wiki | Fandom](https://zork.fandom.com/wiki/Dalboz_of_Gurth)

自分はプレイしたことがないためピンと来ないが、1つの作品だけから取らないあたり、コードネームを付けている人は相当のファンなのだろうか。  
それ以外のボードもある作品内のキャラクターから取られていることが多い。[^3]  
[Debian](https://www.debian.org/)のコードネームがトイ・ストーリーのキャラクターから取られているのは有名な話で、この名付け方はプログラマー、ハッカーに共通するユーモアと言えるだろう。  

[^2]: [new_variant: add reference board name to create_coreboot_variant.sh (I9ac7ad18) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/dev-util/+/2044976)
[^3]: <https://github.com/coreboot/chrome-ec/tree/master/baseboard>

ボードのPCIe接続のデバイス、映像出力の仕様は、  
**Picasso** の場合、NVMe SSD (PCIeGen3 *4レーン*)、WLAN (PCIeGen3 1レーン)、SD Reader (PCIeGen3 1レーン)、  
映像出力は最大4画面、内1画面はノートPCの搭載ディスプレイ(eDP)、外部への出力はHDMI、2x USB-C(DP)、  
**Raven2/Dali** の場合、NVMe SSD (PCIeGen3 *2レーン*)、WLAN (PCIeGen3 1レーン)、SD Reader (PCIeGen3 1レーン)、  
映像出力は最大3画面、内1画面はノートPCの搭載ディスプレイ(eDP)、外部への出力は2x USB-C(DP)、  
となっているようにChromiuOSのソースコードからは読める。[^13]

[^13]: [Dalboz: tuning PCIe topology make DUT can boot up from SSD (I9113b54d) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2066389)

*Dalboz* 以外のボード、*Ezkinil* 、*Moephius* 、*Trembyle* それぞれの間から大きな違いは見られず、あるのはセンサーやファン、入力インターフェイス、バッテリーといった細かい違いとされる。  

### Ezkinil {#ezkinil}

バッテリーはAP18C7M[^20]。容量は3834mAh(55.9Wh)[^21]。  
バッテリーから *Ezkinil* はAcerのボードと思われる。  

[^20]: [Ezkinil: Add new SMP battery (Ib290b109) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2055265)
[^21]: [Acer AP18C7M, 4ICP5/57/79 15.4V 3834mAh original batteries - $71.81 : Laptop battery shop](https://www.laptop-battery-shop.com/acer-ap18c7m-4icp55779-154v-3834mah-original-batteries-p-7493.html)

USBと映像出力の仕様は、2x USB-A(5Gbps)、3x USB-C(5Gbps)、1x HDMIとされる。[^24]

[^24]: <https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2142848/1/board/ezkinil/board.h#101>

### Morphius {#morphius}
入力インターフェイスにトラックポイントがあるとされている。[^15]  
トラックポイントと言えば、今はLenovoブランドのThinkPadが連想されるが、既にLenovoはThinkPadの名を冠したChromebookを出しており、Ryzenを搭載したChromebookなThinkPadの登場も期待できる。  

[^15]: [morphius: add gpio for PS/2 interface (Ic71c3b61) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2077697)

バッテリーは SMP SB10x63140、または Sunwoda L18D3PG1 。[^16]  
調べた所、後者はあっさりと見つかり、バッテリー容量は3735mAh(42Wh)。[^17]  
Lenovo専用のバッテリーということだが、トラックポイントと合わせて考えるに、*Morphius* はLenovo用のボードということなのだろうか。  

[^16]: [morphius: Add batteries configuration (I258058ca) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/1966815)
[^17]: [Canada L18D3PG1 Battery For Lenovo Laptop Li-Polymer 11.25v 42Wh Black](https://www.canada-laptop-battery.com/canada-battery-lenovo-7895.html)

USBと映像出力の仕様は、1x USB-A(5Gbps)、3x USB-C(5Gbps)で内1つが映像出力に対応、1x HDMIとされる。[^25]

[^25]: <https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2142848/1/board/morphius/board.h#109>

### Trembyle {#trembyle}
入力インターフェイス等で、これといった特徴は見つけられなかった。  

バッテリーはAP18F4M[^19]。容量は6850mAh(52Wh)[^18]。  
*Trembyle* は *Ezkinil* 同様にAcer用のボードだろうか。バッテリーからして *Ezkinil* より大きいサイズのノートPC、高性能向けのボードと考えられる。  

[^18]: [Acer AP18F4M, 2ICP5/54/90-2 7.6V 6850mAh original batteries - $62.39 : Laptop battery shop](https://www.laptop-battery-shop.com/acer-ap18f4m-2icp554902-76v-6850mah-original-batteries-p-7161.html)
[^19]: [Trembyle: Use correct battery settings. (I5dc993bf) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/1845782)

USBと映像出力の仕様は、3x USB-A(10Gbps)、3x USB-C(10Gbps) 内1つが映像出力に対応、1x HDMIとされ、10Gbpsに対応という所からも *Trembyle* が上位製品ということを窺える。[^27]  

[^27]: <https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2142848/1/board/trembyle/board.h#98>

### Dalboz {#dalboz}
ソケットからして他と同じ *FP5 BGAソケット* ではなく、*FT5 BGAソケット* を採用している。[^1]  
その *FT5 BGAソケット* は、メモリをシングルチャネルとし、SATAインターフェイスは無しとパッケージサイズを小さくし、配線数も減らしているものと思われる。[^14]元々 *FP5 BGAソケット* は *Raven* のための仕様であったため、規模を小さくした *Raven2* には必要のない部分があった。  
ただSATAに関しては、後のパッチセットでは記述がなくなっているため、いまいち確定していないところがある。  

[^14]: <https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051509/1/src/soc/amd/picasso/Kconfig>

消費電力も通常時で最大4.8W、ブースト時最大9Wに制限されており、冷却設計の簡易化によるコスト削減、バッテリー持続時間に期待が持てる。[^1]  

[^1]: [mb/google/zork: update power parameters to 4.8w for dalboz (I711d1109) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2135098)

そしてバッテリーは3種類あり、SMP L19M3PG1、LGC L19L3PG1、Celxpert L19C3PG1。その中で仕様が見つかったのは L19L3PG1のみ。  
容量は4123mAh(47.6Wh)。[^21]  

バッテリーから *Dalboz* はLenovoのボードと思われるが、残念ながらトラックポイントが存在することを示すコードは無かった。  

[^21]: [Lenovo L19L3PG1 11.55V 4123mAh original batteries - $64.31 : Laptop battery shop](https://www.laptop-battery-shop.com/lenovo-l19l3pg1-1155v-4123mah-original-batteries-p-7520.html)

*Morphius* のコードにはファンのパロメーターが記述されていたのに対し[^24]、*Dalboz* には無いことから、*Dalboz* はファンレスモデルである可能性がある。[^23]  

[^23]: <https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2142848/1/board/morphius/board.c>
[^24]: <https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2142848/1/board/dalboz/board.c>

USBは、3x USB-A(5Gbps)、2x USB-C (5Gbps)、1x HDMIとされる。  

[^26]: <https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2142848/1/board/dalboz/board.h#91>
