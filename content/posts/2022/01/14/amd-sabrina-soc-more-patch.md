---
title: "AMD Sabrina SoC に向けたさらなるパッチ"
date: 2022-01-14T06:18:51+09:00
draft: false
tags: [ "Sabrina", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

先日に続いて Coreboot に、[Felix Held](https://github.com/felixheld) 氏より、*AMD Sabrina SoC* (`Family 17h Model A0h (0x008A0F00)`) のサポート追加に向けたパッチが投稿されている。  

 * [topic:amd-sabrina-soc · Gerrit Code Review](https://review.coreboot.org/q/topic:amd-sabrina-soc)
 > 		#define SABRINA_A0_CPUID	0x008a0f00
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/61087/2/src/soc/amd/sabrina/include/soc/cpu.h#6> {{< /quote >}}

{{< pindex >}}
 * [AMD Chausie](#chausie)
    * [UARTコントローラ](#uart)
    * [CPPC2 をサポートせず](#cppc2)
    * [PCIe](#pcie)
    * [SATAコントローラを持たず](#sata)
{{< /pindex >}}

## AMD Chausie {#chausie}

*Sabrina SoC* のメインボード名は *AMD Chausie*。コードとしては、*AMD Chausie* ボード部に *FP6パッケージ* を採用する *AMD Majolica* を、FSP (Firmware Support Package) も *Cezanne SoC* をベースにしている。コードファイルのほとんどをコピーし、名前だけを置き換えた形となっている。[^copy-from]  
コードファイルを *Cezanne* 関連からコピーした理由に、[Felix Held](https://github.com/felixheld) 氏は、ひとまずビルドを成功させることを挙げている。  
そのためベース部の情報が多く、*Sabrina SoC* 、*AMD Chausie* 固有の情報はまだ読みにくいところがある。パッケージも、*Cezanne* 同様に FP6 が使われるかどうかはまだ判然としない。  

[^copy-from]: [vc/amd/fsp/sabrina: add as a copy of vc/amd/fsp/cezanne (Ib3bf5059) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61076) <br> [mb/amd/chausie: add mainboard as copy of mb/amd/majolica (Ic7b18f7a) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61079/3)

それでも確かな部分を読んでいくと、*Sabrina SoC* では一部の機能、I/O が削減され、また規模が増やされた部分がわずかに存在する。  

### UARTコントローラ {#uart}
*Sabrina SoC* では UART (Universal Asynchronous Receiver Transmitter) コントローラが、*Cezanne* から 3基増やされている。  

 * [soc/amd/sabrina/include/aoac_defs: add additional UARTs (Id9876719) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61082/4)
* [soc/amd/sabrina: add additional UART controllers (I628b1a7a) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61086/3)

*Cezanne* 、というより *Zen 2/3 APU (Renoir /Lucienne /Cezanne /Barcelo)* では UARTコントローラは 2基だったため、合計で 5基となる。  
*Zen/+ APU (Raven /Picasso /Raven2 [Dali /Pollock])* では 4基だったため、それよりも 1基増やされた。  
UART はデバイス間のシリアル通信に使われるため、組み込み向け製品への採用を想定しているのかもしれない。  

### CPPC2 をサポートせず {#cppc2}
*Sabrina SoC* では CPPC2 (Collaborative Processor Performance Control 2) をサポートしないとし、*Cezanne SoC* からコピーした CPPC2 関連のコードファイルを削除している。  
CPPC は電源管理機能の 1つであり、CPPC2 ではクロックの選択が高速化されている。CPPC2 のサポートは性能と電力効率、両方の向上に効果があるとされ、AMD CPU/SoC では *Zen 2 アーキテクチャ* の世代から実装されている。  
だがそれを *Sabrina SoC* ではサポートしないという。  

*Sabrina SoC* は CPUID Family は 17h (23) であり、アーキテクチャは *Zen/+/2* のどれかと見られるが、CPPC2 をサポートしないことから *Zen/+* の可能性も充分に考えられるようになった。  
ただ、今になって CPU部が *Zen/+ アーキテクチャ* の APU/SoC が新規に登場するというのも不自然だが、CPPC2 をサポートしない *Zen 2 アーキテクチャ* というのもどこかちぐはぐに感じられる。  

 * [soc/amd/sabrina: drop CPPC code (I71a1b071) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61096)

### PCIe {#pcie}
*Sabrina SoC* では *Zen 2/3 APU* と比べて規模が抑えられているように見える。  

Dicrete GPU用の PCIe GPP (General Purpose Port) Bridge 1基でそこから出せるポート数は 1ポート。*Zen 2/3 APU* では PCIe GFX/GPP Bridge から 3ポートに分割して出すことが可能だった。  
*Zen/+ APU* でも PCIe GFX/GPP Bridge 1基から 1ポートという仕様だったため、それらと同様の仕様に戻したとも見られる。  
それ以外の PCIe GPP を合わせて 6ポートとなり、*Raven2 (Dali /Pollock)* 以外の *Zen系 APU* が持つ 7ポートより 1ポート少ない。  
以上はポート数であり、全体の PCIeレーン数は不明。  
とはいえ、分割可能なポート数を減らしていることと *Sabrina SoC* が内蔵 GPU を持つ APU であることを考えると、*Zen 2/3 APU* より小さいと思われる。  

 > 		-/* PCIe GFX/GPP Bridge device 1 with up to 3 ports */
 > 		+/* PCIe GFX/GPP Bridge device 1 with no ports */
 > 		 #define PCIE_GPP_BRIDGE_1_DEV	0x1
 > 		 
 > 		-#define PCIE_GPP_1_0_FUNC	1
 > 		-#define PCIE_GPP_1_0_DEVFN	PCI_DEVFN(PCIE_GPP_BRIDGE_1_DEV, PCIE_GPP_1_0_FUNC)
 > 		-#define SOC_GPP_1_0_DEV		_SOC_DEV(PCIE_GPP_BRIDGE_1_DEV, PCIE_GPP_1_0_FUNC)
 > 		-
 > 		-#define PCIE_GPP_1_1_FUNC	2
 > 		-#define PCIE_GPP_1_1_DEVFN	PCI_DEVFN(PCIE_GPP_BRIDGE_1_DEV, PCIE_GPP_1_1_FUNC)
 > 		-#define SOC_GPP_1_1_DEV		_SOC_DEV(PCIE_GPP_BRIDGE_1_DEV, PCIE_GPP_1_1_FUNC)
 > 		-
 > 		-#define PCIE_GPP_1_2_FUNC	3
 > 		-#define PCIE_GPP_1_2_DEVFN	PCI_DEVFN(PCIE_GPP_BRIDGE_1_DEV, PCIE_GPP_1_2_FUNC)
 > 		-#define SOC_GPP_1_2_DEV		_SOC_DEV(PCIE_GPP_BRIDGE_1_DEV, PCIE_GPP_1_2_FUNC)
 > 		-
 > 		-/* PCIe GPP Bridge device 2 with up to 7 ports */
 > 		+/* PCIe GPP Bridge device 2 with up to 6 ports */
 >
 > {{< quote >}} [Diff - cfa7f283968621d31a49c5179496e72ec7504ce5^! - coreboot - Gitiles](https://review.coreboot.org/plugins/gitiles/coreboot/+/cfa7f283968621d31a49c5179496e72ec7504ce5%5E%21/#F0) {{< /quote >}}

### SATAコントローラを持たず {#sata}
*Sabrisa SoC* は SATAコントローラを持たないとされている。  
*Raven2 (Pollock)* が搭載可能な小型、省電力クラスのパッケージ、*FT5* では SATAインターフェイスを持たなかったが、それに近い。  
{{< link >}} [Chromebookに搭載されるPollockはFT5ソケット ――DDR4シングルチャネル、SATA無し | Coelacanth's Dream](/posts/2020/02/12/amd-pollock-ft5/) {{< /link >}}
だが *Raven2 (Pollock)* 自体は SATAコントローラを 2基持ち、*FT5* パッケージの仕様としてインターフェイスを廃した形であるため、SoC 自体に SATAコントローラを持たない *Sabrina* とは色が異なる。  
最近は NVMe SSD がメインストレージになってきているが、割り切った変更であるようにも思える。  
*Sabrina SoC* は、内部ストレージをそう搭載しないモバイル向けに開発された SoC なのだろうか。  

 > 		soc/amd/sabrina/fsp_m_params: drop sata_enable UPD write
 > 		
 > 		There are no SATA controllers on the Sabrina SoC. The UPD field will be
 > 		removed later as a part of the initial UPD header update.
 >
 > {{< quote >}} [soc/amd/sabrina/fsp_m_params: drop sata_enable UPD write (Iedefd9f1) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61091/3) {{< /quote >}}

*Sabrina SoC* については、まだ AMDGPUドライバ等へのパッチが公開されていないため、謎がまだまだある存在ではあるが、I/O の規模から *Raven2 (Dali /Pollock)* に似ているように思う。  
*Raven2 (Dali /Pollock)* は CPU *Zen* 2-Core、GPU *Vega (gfx909)* 3CU という、小規模で、低コスト省電力が必要なプラットフォームに向けた APU/SoC。  
*Zen 2 アーキテクチャ* を採用する *Renoir APU* からは、CPU 8-Core の APU がメインとなり、更新されるのも上位帯に向けたものだった。*VanGogh APU* は *Zen 2* 4-Core、*RDNA 2* 8CU であり、小規模な APU と言えるかもしれないが、カスタム APU であるため、一般向け製品に採用される見込みは薄い。  
そうした中で、*Sabrina SoC* は *Raven2* 以降の小規模 APU となるかもしれない。  

| | Raven /Picasso<br>/Raven2 (Dali) | Renoir /Cezanne<br>/Barcelo | Raven2<br>(Pollock) | Sabrina |
| :-- | :--: | :--: | :--: | :--: |
| Package | FP5 | FP6 | FT5 | ? |
| Base board name | Mandolin | Majolica | Cereme | Chausie |
| UART controller | 4 | 2 | 4? | 5 |
| SATA controller | 2 | 0 | 2 | 0 |
| PCIe GPP | 7 | 7 | 5 | 6 |

{{< ref >}}
 * [よく分かる！ シリアル通信基礎講座 | 組込み技術ラボ](https://emb.macnica.co.jp/articles/8191/)
{{< /ref >}}
