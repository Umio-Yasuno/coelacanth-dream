---
title: "Alder Lake は PCIe Gen5 に対応、Tiger Lake-H は Gen4 20-Lane を備える"
date: 2021-01-01T22:36:52+09:00
draft: false
tags: [ "Tiger_Lake", "Alder_Lake" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

{{< pindex >}}

 * [Alder Lake-P は PCIe Gen5 に対応](#adl_p-pci_gen5)
 * [Tiger Lake-H は PCIe Gen4 20-Lane を備える](#tgl_h-gen4x20)

{{< /pindex >}}

## Alder Lake-P は PCIe Gen5 に対応 {#adl_p-pci_gen5}

Coreboot の *Alder Lake-P* サポートに向けたあるパッチが投稿され、そのコメントスレッドの中で、*Alder Lake-P* は PCIe Gen5 に対応、8-Lane 1ポート を備えることが分かった。  
それに加えて PCIe Gen4 4-Lane も 2ポート備える。  

 >     in the EDS, I see:
 >     "The ADL-P processor PCI Express* has two interfaces:
 >     • One 8-lane (x8) port supporting PCIE to gen 5.0 or below.
 >     • Two 4-lane (x4) port supporting PCIE gen 4.0 or below"
 >
 > {{< quote >}} [soc/intel/alderlake: Revise PCIE port config (I0b390e43) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/48340) {{< /quote >}}

当初、{{< comple >}}  google.com と chromium.org とでメールアドレスが分かれてはいるが {{< /comple >}} 同じ資料を所持しているはずの開発者間で、その情報はどこにあるのか、といったやり取りがあり、若干怪しい情報となっていたが、  
その後、その質問をした開発者の方が EDS (External Design Specification) で確認できたとコメントし、確かな情報となった次第だ。  
警戒しすぎと思われるかもしれないが、過去に WikiChip を引き合いに、[Raven2/Dali](/tags/dali) の CPU部が *Zen 2 アーキテクチャ* だとしてパッチが投稿されたことがあったためだ。[^dali]  

[^dali]: [いかに Zen APU は複雑か | Coelacanth's Dream](/posts/2020/02/16/raven-family-complex/)

*Alder Lake-P* はモバイル向けとされ[^adl_p-mobile]、CPU側で PCIe Gen5 8-Lane x1、PCIe Gen4 4-Lane x2 というのは些か過剰に思えなくもない。  
現行世代のモバイル向け *Tiger Lake* は、CPU側で PCIe Gen4 4-Lane の規模となる。  
*Tiger Lake* は、実際は PCIe Gen4 4-Lane x2 を持つが、UP3/UP4パッケージでは 1ポートが予約分とされ、使用できないようになっている。[^tgl-up3_up4]  

[^adl_p-mobile]: [Alder Lake-P を搭載する Chromebookボード 「Brya」、そして Alder Lake-M | Coelacanth's Dream](/posts/2020/12/02/adl-p-chromebook-board-adl-m/)

PCIe Gen5 8-Lane の目的を考えると、4-Lane x2 のような分割に対応せず、かつ NVMe SSD 向けに最大 4-Lane としなかったことから、dGPU も想定したもの、というのが浮かぶ。  
一応、分割に対応しないと言っても使い切れないだけで、NVMe SSD 4-Lane を接続することはできる。  

2021年が始まったばかりなのに、こんな事を言うのはどこか少し気が引けるが、*Alder Lake* (-S と -P のどちらかは不明) は 2021年後半に市場へ投入される予定にある。  
サーバー向けである *Sapphire Rapids* も同時期に出荷予定とされ、*Sapphire Rapids* は PCIe Gen5 をベースとする CXL (Compute Express Link) に対応し、スーパーコンピューター Aurora は CXL で構築される。
{{< comple>}} 厳密に言うと、Aurora に用いられるのは、これまた CXL をベースとした {{< xe >}} Link だが、ややこしい。{{< /comple >}}  
そのため *{{< xe class="hpc" >}}* もまた PCIe Gen5/CXL に対応しており、同時期に登場する他の Intel dGPU も PCIe Gen5 に対応している可能性がある。あくまで可能性。  

現在 Intel dGPU では *{{< xe class="lp" >}}* をベースとする **Iris Xe MAX** がリリースされているが、1080p をターゲットとした製品であり、性能としてはエントリー帯にある。  
ハイエンドモバイルのような、ゲーミング性能では 1080p よりも上の解像度をターゲットとしたプラットフォームを、*Alder Lake-P* と dGPU {{< comple >}} 時期と性能を考えれば {{< xe class="hpg" >}} が該当するかもしれないが {{< /comple >}} で構築することが計画されているのかもしれない。  

*Alder Lake-P* と *Alder Lake-M* の差別化において、PCIe Gen5 8-Lane の有効無効が使われる可能性も考えられる。  

## Tiger Lake-H は PCIe Gen4 20-Lane を備える {#tgl_h-gen4x20}

*Tiger Lake-H* の各ログがアップロードされたことは以前取り上げたが、その時見落としているものがあった。  
{{< link >}} [Tiger Lake-H のブートログ、lscpu、lspci …… | Coelacanth's Dream](/posts/2020/12/23/tgl-h-log-lscpu-lspci/) {{< /link >}}
実は lspci は root権限で実行されたもので、通常よりも多くの情報が出力されていた。  
そして、そこから *Tiger Lake-H* の持つ PCIe Gen4 のレーン数を知ることができた。  

 >     00:01.0 PCI bridge [0604]: Intel Corporation Device [8086:9a01] (rev 04) (prog-if 00 [Normal decode])
 >     ~~~~~~
 >     		DevSta:	CorrErr- NonFatalErr- FatalErr- UnsupReq- AuxPwr+ TransPend-
 >     		LnkCap:	Port #2, Speed 16GT/s, Width x16, ASPM L1, Exit Latency L1 <16us
 >          ClockPM- Surprise- LLActRep+ BwNot+ ASPMOptComp+

 >     00:01.1 PCI bridge [0604]: Intel Corporation Device [8086:9a05] (rev 04) (prog-if 00 [Normal decode])
 >     ~~~~~~
 >     		LnkCap:	Port #3, Speed 16GT/s, Width x4, ASPM L1, Exit Latency L1 <16us
 >     			ClockPM- Surprise- LLActRep+ BwNot+ ASPMOptComp+
 > {{< quote >}} [[BUG] Dell TGL-H Machine failed to load sof-tgl-h.ri · Issue #2662 · thesofproject/linux](https://github.com/thesofproject/linux/issues/2662) {{< /quote >}}
 
注目したいのは `LnkCap` の行。Speed はリンクスピードの意で、PCIe Gen4 は 16GT/s と表示される。[^pci-gen4]  
Width はリンク数で、以上のことから *Tiger Lake-H* は PCIe Gen4 16-Lane と 4-Lane のポートを持ち、計 20-Lane を備えることが分かる。  
*Alder Lake-P* の所でも触れたが、モバイル向けの *Tiger Lake_L* は PCIe Gen4 4-Lane x2 を持つが、UP3/UP4パッケージでは PCIe Gen4 4-Lane x1 に制限され、もう 1つのポートは予約分とされていた。[^tgl-up3_up4]  
*Tiger Lake-H* の PCIe Gen4 16-Lane は dGPU用だとして、4-Lane のポートが NVMe SSD用と思われるが、*Tiger Lake UP3/UP4* とは異なり解放されるかは不明。  
また、デスクトップ向け *Rocket Lake-S* も PCIe Gen4 x16 + x4 を持ち、*Tiger Lake-H* はそれと同規模の PCIeレーン数を備えることとなる。  

[^tgl-up3_up4]: 11th Generation Intel® Core™ Processor (UP3 and UP4) Datasheet, Volume 1 of 2 <br> [Intel® Core™ i7-1185G7 Processor (12M Cache, up to 4.80 GHz, with IPU) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/208664/intel-core-i7-1185g7-processor-12m-cache-up-to-4-80-ghz-with-ipu.html?wapkw=tiger%20lake)
[^rkl-pcie]: [Intel Rocket Lake-S の概要を発表　―― Cypress Cove の詳細はまだ、AV1 HWエンコードには非対応 | Coelacanth's Dream](/posts/2020/10/30/intel-rkl/)
[^pci-gen4]: [PCI: Add decoding for 16 GT/s link speed · torvalds/linux@1acfb9b](https://github.com/torvalds/linux/commit/1acfb9b7ee0b1881bb8e875b6757976e48293ec4)

PCH は、 DeviceID を TGP_LP のものと照らし合わせたが一致しなかったため、*Tiger Lake-H* 用の PCH と思われる。  

*Tiger Lake-H* と直接の関係は無いが、同 lspci のログによると dGPU に NVIDIA GPU (DeviceID: 0x24b8) が接続されていた。  
自分が NVIDIA GPU に詳しくないというのもあるが、DeviceID を [The PCI ID Repository](https://pci-ids.ucw.cz/) で調べても出てこなかった。[^pciid-10de]  
オーディオコントローラーの DeviceID はあり、Geforce RTX 3070 等に採用されている GA104 のものと一致したため (DeviceID: 0x228b)、GA104 ベースの GPU だと思われはするが。  
CPU だけでなく、GPU もエンジニアサンプリング品だったのだろうか？  

[^pciid-10de]: <https://pci-ids.ucw.cz/read/PC/10de>

| | Tiger Lake | Tiger Lake-H | Alder Lake-P | Alder Lake-M |
| :-- | :--: | :--: | :--: | :--: |
| CPU side PCIe | Gen4<br>4-Lane x2 | Gen4<br> 16-Lane + 4-Lane | Gen5 8-Lane<br>Gen4 4-Lane x2 | ? |
| PCH | TGP_LP | TGP_H | ADP_P | TGP_LP? |

{{< ref >}}

 * [coreboot/pci_ids.h at master · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/master/src/include/device/pci_ids.h)

{{< /ref >}}
