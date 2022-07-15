---
title: "AMD Sabrina SoC の名が Mendocino SoC に置き換えられる"
date: 2022-07-15T13:06:18+09:00
draft: false
categories: [ "Software", "AMD", "APU" ]
tags: [ "Sabrina", "Mendocino", "Coreboot" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Google のソフトウェアエンジニアである Jon Murphy 氏により、これまで AMD *Sabrina* SoC として Coreboot でサポートが進められてきた SoC のコードネームを *Mendocino* に置き換えるパッチが投稿されている。  

 * [Sabrina: Remove embargoed name (Ib82b3453) · Gerrit Code Review](https://review.coreboot.org/c/amd_blobs/+/65864/1)
 * [treewide: Remove embargoed name (I2d0f76fd) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65861/6)
 * [chipset-mendocino: Add initial mendocino chipset overlay (I6461a700) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/overlays/board-overlays/+/3727882/9)

Jon Murphy 氏のコミットメッセージによれば、*Mendocino* の名前は禁止されていたため、それを採用する Chromebook ボード *Skyrim* への参照に使用することができなかった。  
Coreboot では *Mendocino* の代わりに *Sabrina* を使用してきたが、こうした別のコードネームを使用し続けることは、今後 *Mendocino* を搭載した製品が出ることを考えると、サポート状況を確認することを長期的に渡って困難にしてしまう。  
そのため、*Sabrina* のコードネームを *Mendocino* に置き換え、統一することにしたという。  
AMD 公式では *Mendocino* のコードネームで正式発表したことも、要因としては大きいだろう。  

 > 		'Mendocino' was an embargoed name and could previously not be used
 > 		in references to Skyrim.  coreboot has references to sabrina both
 > 		in directory structure and in files. This will make life difficult
 > 		for people looking for Mendocino support in the long term. The code
 > 		name should be replaced with "mendocino".
 >
 > {{< quote >}} [Sabrina: Remove embargoed name (Ib82b3453) · Gerrit Code Review](https://review.coreboot.org/c/amd_blobs/+/65864/1) {{< /quote >}}

*Mendocino* のコードネームが禁止されていた理由については触れられていない。  
過去に Intel が CPU のコードネームに用いていたからなのか、単に AMD の正式発表を前に本来のコードネームを使ってサポートを進めることができなかったのか。  

*Mendocino* のスペックについては、CPU は *Zen 2 アーキテクチャ* 4-Core/8-Thread、GPU は *RDNA 2/GFX10.3 (GC 10.3.7/gfx1037)*、メディアエンジンは VCN 3.1.1、PCIe 4-Lane、メモリには LPDDR5 5500MT/s をサポートしている。[^lpddr5]  

[^lpddr5]: [util/spd_tools: Limit memory speed to 5500 Mbps for Sabrina (Ie3507898) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65708/2)

 > 		/*
 > 		 * Mendocino DXIO Descriptor: Used for assigning lanes to PCIe engines, configure
 > 		 * bifurcation and other settings. Beware that the lane numbers in here are the
 > 		 * logical and not the physical lane numbers!
 > 		 *
 > 		 * Mendocino DXIO logical lane to physical PCIe lane mapping:
 > 		 *
 > 		 * logical | physical
 > 		 * --------|------------
 > 		 * [00:03] | GPP[03:00]
 > 		 *
 > 		 * Different ports mustn't overlap or be assigned to the same lane(s). Within
 > 		 * ports with the same width the one with a higher start logical lane number
 > 		 * needs to be assigned to a higher PCIe root port number; ports of the same
 > 		 * size don't have to be assigned to consecutive PCIe root ports though.
 > 		 */
 >
 > {{< quote >}} […/platform_descriptors.h · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65861/6/src/vendorcode/amd/fsp/mendocino/platform_descriptors.h#b187) {{< /quote >}}
