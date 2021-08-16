---
title: "AMD の新たな APU/SoC 「Mendocino」"
date: 2021-08-12T18:29:12+09:00
draft: false
tags: [ "Mendocino", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

プロプライエタリな BIOS (ファームウェア) の代替を目的としたフリーソフトプロジェクト [Coreboot](https://www.coreboot.org/) における `amdfwtool` に向けて、新たな AMD APU/SoC *Mendocino (メンドシノ、メンダシーノ)* をサポートする最初のパッチが投稿された。  
`amdfwtool` は AMD APU/SoC に必要な複数のファームウェアを単一のモジュールにまとめるツール。  
パッチは [Zheng Bao](https://github.com/fishbaoz) 氏より投稿されている。  

 * [amdfwtool: Add new SOC mendocino (I54492600) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56936)

 > 		enum platform {
 > 			PLATFORM_UNKNOWN,
 > 			PLATFORM_STONEYRIDGE,
 > 			PLATFORM_RAVEN,
 > 			PLATFORM_PICASSO,
 > 			PLATFORM_RENOIR,
 > 			PLATFORM_CEZANNE,
 > 			PLATFORM_MENDOCINO,
 > 			PLATFORM_LUCIENNE,
 > 		};
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/56936/2/util/amdfwtool/amdfwtool.c#b1152> {{< /quote >}}

とは言え、最初のパッチということもあり、パッチの内容は *Mendocino* の名前と分岐先を追加するのが主なものであり、*Mendocino* の素性 {{< comple >}} CPUアーキテクチャ、世代 {{< /comple >}} については触れられていない。  
*new SOC* とあることから *Mendocino* は APU だと考えられるが、それくらい。  

最近では [Barcelo](/tags/barcelo) APU をサポートするパッチも Coreboot に投稿、公開されていたが、そちらは *Cezanne APU* のコードを一部共有していること、GPU部の DeviceID が同時期に AMDGPUドライバーに投稿されていたことから、*Cezanne* 同様に *Zen 3 CPU + Vega GPU* という構成を採る APU と考えられる。  
{{< link >}} [新たな Zen 3 + Vega APU 「Barcelo」 | Coelacanth's Dream](/posts/2021/07/17/amd-barcelo-vega-apu/) {{< /link >}}

オープンソース・プロジェクト、オープンソース・ドライバーに名前が登場し、サポート作業が進められているが、モデルがまだ正式発表されていないものには、[Yellow Carp](/tags/yellow_carp) ([Rembrandt](/tags/rembrandt))、*Barcelo* 、そして今回追加された *Mendocino* 、  
既にカスタム APU/SoC として製品に搭載されている可能性があるが、まだはっきりと確認されてはいないものに [VanGogh](/tags/vangogh)、[Cyan Skilfish](/tags/cyan_skilfish) がいる。  
謎を持つ AMD APU/SoC は現状でこれだけ存在するため、下手に推測するよりも、まずはゆっくりと新たなパッチで投稿されるのを待ち、読み解いていくことを楽しんでいきたいと思っている。  

| AMD APU | CPU Arch | GPU Arch |
| :-- | :--: | :--: |
| Renoir | Zen 2 | Vega (gfx90c) |
| Lucienne | Zen 2 | Vega (gfx90c) |
| Cezanne | Zen 3 | Vega (gfx90c) |
| Barcelo | Zen 3 ? | Vega (gfx90c) |
| *Mendocino* | ? | ? |
| Yellow Carp (Rembrandt) | ? | RDNA 2 (gfx1035) |
| VenGogh | Zen 2 | RDNA 2 (gfx1033) |
| Cyan Skilfish | ? | RDNA (gfx1013) |
