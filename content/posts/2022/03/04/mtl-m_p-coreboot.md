---
title: "Coreboot に Meteor Lake-M/P の DeviceID が追加される"
date: 2022-03-04T16:58:58+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Meteor_Lake", "Coreboot" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Wonkyu Kim 氏より、Coreboot に *Meteor Lake-M/P* の CPUID/DeviceID (PCI ID) を追加するパッチが投稿されている。  

* [soc/intel/common: Include Meteor Lake device IDs (Ie71abb70) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/62581/6/)
* [topic:MTL_UPSTREAM · Gerrit Code Review](https://review.coreboot.org/q/topic:MTL_UPSTREAM)

*Meteor Lake-M/P* の Family, Model, Stepping を示す CPUID は `0xA06AX, (X = Stepping)`。Model は `0xAA (170)` となる。  
今回のパッチで追加された *Meteor Lake-M/P* の CPUID は、[intel/dptf (Intel (R) Dynamic Tuning for Chromium OS)](https://github.com/intel/dptf) 内に記述されているものと一致する。  
少し *Meteor Lake* から外れるが、現時点で *Raptor Lake-P* のサポートを追加するパッチは Coreboot や Linux Kernel の各種ドライバーに投稿されていない。以下の *Raptor Lake-P* の CPUID Model: `0xBA (186)` は、モバイル向け *Rocket Lake* のように内部的に割り当てられた仮の値なのだろうか。  

 > 		#define CPUID_METEORLAKE_A0_1		0xa06a0
 > 		#define CPUID_METEORLAKE_A0_2		0xa06a1
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/62581/1/src/include/cpu/intel/cpu_ids.h> {{< /quote >}}

 > 		#define CPUID_FAMILY_MODEL_MTL_M_P	0x000A06A0		// Meteor Lake M & P
 > 		#define CPUID_FAMILY_MODEL_MTL_N	0x000A06B0		// Meteor Lake N
 > 		#define CPUID_FAMILY_MODEL_MTL_S	0x000606C0		// Meteor Lake S
 > 		#define CPUID_FAMILY_MODEL_RPL_S	0x000B0670		// Raptor Lake S
 > 		#define CPUID_FAMILY_MODEL_RPL_P	0x000B06A0		// Raptor Lake P
 >
 > {{< quote >}} <https://github.com/intel/dptf/blob/e1f10f989223720ccb6b2519f8d96435925407c0/Common/esif_ccb_cpuid.h#L108-L112> {{< /quote >}}

*Meteor Lake* は CPU, GPU, SoC, IO という 4種類の Tile で構成されることが Intel Investor Meeting 2022 で発表されている。[^mtl-tile]  
従来の Intelプラットフォームでは、PCIe Root Port (RP)、インターフェイス等は CPUダイのアンコア部と PCHダイに実装されていたが、*Meteor Lake* では SoC Tile と IO Tile に実装される。  
そして以下引用部の Device ID マクロ名から、IO Tile への追加の PCIe RP、インターフェイスの実装は *Meteor Lake-P* になると思われる。  
*Meteor Lake-M/P* 間で SoC Tile は共通し、IO Tile は異なるというような実装になるのだろうか。その場合、従来の PCHダイは IO Tile に統合される形となる。  

 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP1			0x7e38
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP2			0x7e39
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP3			0x7e3a
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP4			0x7e3b
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP5			0x7e3c
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP6			0x7e3d
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP7			0x7e3e
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP8			0x7e3f
 > 		#define PCI_DID_INTEL_MTL_SOC_PCIE_RP9			0x7e4d
 > 		#define PCI_DID_INTEL_MTL_IOE_P_PCIE_RP10		0x7eca
 > 		#define PCI_DID_INTEL_MTL_IOE_P_PCIE_RP11		0x7ecb
 > 		#define PCI_DID_INTEL_MTL_IOE_P_PCIE_RP12		0x7ecc
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/62581/6/src/include/device/pci_ids.h#3423> {{< /quote >}}

[^mtl-tile]: [Notices and Disclaimers - 2022-Intel-Investor-Meeting-Client.pdf](https://download.intel.com/newsroom/2022/corporate/2022-Intel-Investor-Meeting-Client.pdf)

マクロ名から推察できる *Meteor Lake-M/P* の違いには他に USB4/Thunderbolt が挙げられ、*Meteor Lake-M* では DeviceID が RP には 2つ、DMA は 1つとなっているのに対し、*Meteor Lake-P* では RP に 4つ、DMA は 2つとなっている。  
USB4/Thunderbolt に使用可能なレーン数や接続可能なデバイス数に違いがあるものと思われる。  

 > 		#define PCI_DID_INTEL_MTL_M_TBT_RP0		0x7eb4
 > 		#define PCI_DID_INTEL_MTL_M_TBT_RP1		0x7eb5
 > 		#define PCI_DID_INTEL_MTL_P_TBT_RP0		0x7ec4
 > 		#define PCI_DID_INTEL_MTL_P_TBT_RP1		0x7ec5
 > 		#define PCI_DID_INTEL_MTL_P_TBT_RP2		0x7ec6
 > 		#define PCI_DID_INTEL_MTL_P_TBT_RP3		0x7ec7
 > 		#define PCI_DID_INTEL_MTL_M_TBT_DMA0		0x7eb2
 > 		#define PCI_DID_INTEL_MTL_P_TBT_DMA0		0x7ec2
 > 		#define PCI_DID_INTEL_MTL_P_TBT_DMA1		0x7ec3
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/62581/6/src/include/device/pci_ids.h#3423> {{< /quote >}}

{{< ref >}}
* [soc/intel/common: Add the Primary to Sideband bridge library (I63f58584) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61283)
* [Intel Investor Meeting 2022](https://www.intel.com/content/www/us/en/newsroom/resources/investor-day-2022.html)
{{< /ref >}}
