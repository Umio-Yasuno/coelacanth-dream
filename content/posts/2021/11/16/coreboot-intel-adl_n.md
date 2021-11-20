---
title: "N バリアントも存在する Alder Lake"
date: 2021-11-16T00:13:26+09:00
draft: false
tags: [ "Alder_Lake", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

プロプライエタリな BIOS (firmware) の置き換えを目指すフリーソフトウェアプロジェクト [Coreboot](https://github.com/coreboot/coreboot) に、*Alder Lake-N* の CPUID, DeviceID を追加するパッチが投稿、公開されている。  

 * [soc/intel/common: Include Alder Lake device IDs (I0974fc6e) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59306)

## CPUID {#cpuid}

Intel CPU では CPUID における `Model` がマーケットセグメント別に割り当てられていることが多く、例えば *Alder Lake-M/P* はどちらもモバイル向けであり、`Model: 0x9A (ALDERLAKE_L)` が割り当てられている。[^intel-fam]
{{< link >}} [ALDERLAKE_L と Alder Lake-P/M | Coelacanth's Dream](/posts/2021/02/09/alderlake_l/) {{< /link >}}

[^intel-fam]: <https://github.com/torvalds/linux/blob/fbdb5e8f2926ae9636c9fa6f42c7426132ddeeb2/arch/x86/include/asm/intel-family.h>

今回のパッチによれば *Alder Lake-N* には、デスクトップ向けの `ALDERLAKE_S (Model: 0x97)` ともモバイル向けの `ALDERLAKE_L (Model: 0x9A)` とも異なる、`Model: 0xBE` が割り当てられている。  

 > 		#define CPUID_ALDERLAKE_S_A0		0x90670
 > 		#define CPUID_ALDERLAKE_A0		0x906a0
 > 		#define CPUID_ALDERLAKE_A1		0x906a1
 > 		#define CPUID_ALDERLAKE_A2		0x906a2
 > 		#define CPUID_ALDERLAKE_A3		0x906a4
 > 		#define CPUID_ALDERLAKE_N_A0		0xb06e0
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/59306/1/src/include/cpu/intel/cpu_ids.h> {{< /quote >}}

デスクトップ向けでもモバイル向けでもないことから、*Alder Lake-N* は組み込み向けのプロセッサではないかと思われる。  
ただ [intel-family.h](https://github.com/torvalds/linux/blob/fbdb5e8f2926ae9636c9fa6f42c7426132ddeeb2/arch/x86/include/asm/intel-family.h) にはまだ `ALDERLAKE_N (Model: 0xBE)` は追加されておらず、他 *Alder Lake* 同様に *Golden Cove + Gracemont* のハイブリッドアーキテクチャを採るのか、組み込み向けとして処理性能を安定させるため *Gracemont* だけの構成とするのかなどは不明。  

余談だが、 Intel Corporation の Githubレポジトリの 1つ [intel/dptf](https://github.com/intel/dptf) (Intel (R) Dynamic Tuning for Chromium OS) には既に *Meteor Lake M & P, N, S* の CPUID が記載されており、モバイル向けに M & P、デスクトップ向けに S、そして恐らく組み込み向けだろう N の 3つのセグメントに分かれる方針は *Meteor Lake* にも用いられると思われる。  

 > 		#define CPUID_FAMILY_MODEL_MTL_M_P	0x000A06A0		// Meteor Lake M & P
 > 		#define CPUID_FAMILY_MODEL_MTL_N	0x000A06B0		// Meteor Lake N
 > 		#define CPUID_FAMILY_MODEL_MTL_S	0x000606C0		// Meteor Lake S
 > 		#define CPUID_FAMILY_MODEL_RPL_S	0x000B0670		// Raptor Lake S
 > 		#define CPUID_FAMILY_MODEL_RPL_P	0x000B06A0		// Raptor Lake P
 > {{< quote >}} <https://github.com/intel/dptf/blob/e1f10f989223720ccb6b2519f8d96435925407c0/Common/esif_ccb_cpuid.h#L108-L112> {{< /quote >}}


## GPU {#gpu}

GPU部の DeviceID も `0x46A0/0x46A3` の 2つが追加されている。

 > 		#define PCI_DEVICE_ID_INTEL_ADL_M_GT1			0x46c0
 > 		#define PCI_DEVICE_ID_INTEL_ADL_M_GT2			0x46aa
 > 		#define PCI_DEVICE_ID_INTEL_ADL_M_GT3                   0x46c3
 > 		#define PCI_DEVICE_ID_INTEL_ADL_N_GT1			0x46A0
 > 		#define PCI_DEVICE_ID_INTEL_ADL_N_GT2			0x46A3
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/59306/1/src/include/device/pci_ids.h#3885> {{< /quote >}}


Mesa3D 内の [iris_pci_ids.h](https://gitlab.freedesktop.org/mesa/mesa/blob/main/include/pci_ids/iris_pci_ids.h) では、2つの DeviceID は Alder Lake GT2 に当たるため、*Alder Lake-N* の GPU部は *Alder Lake-M/P* 同様に最大 96EU の構成とされる。  
また *Alder Lake-N* の DeviceID の上に `PCI_DEVICE_ID_INTEL_ADL_M_GT3` があるが、この `DeviceID: 0x46C3` も Alder Lake GT2 に当たる。そのため製品的な意味では GT3 なのかもしれないが、最大構成としては Alder Lake GT2 (96EU) だと思われる。  

あるいは、*Jasper Lake* の GPU部の DeviceID にも GT1 から GT4 があるが、Intel のドキュメントによれば `GT4 (0x4E55)` は 16EU、`GT3 (0x4E61)` は 24EU、`GT2 (0x4E71)` は 32EU、`GT1 (0x4E51)` は予約された ID にあたるため、単に連番以上の意味は持たないかもしれない。[^jsl-ids]

 > 		#define PCI_DEVICE_ID_INTEL_JSL_GT1			0x4E51
 > 		#define PCI_DEVICE_ID_INTEL_JSL_GT2			0x4E71
 > 		#define PCI_DEVICE_ID_INTEL_JSL_GT3			0x4E61
 > 		#define PCI_DEVICE_ID_INTEL_JSL_GT4			0x4E55
 >
 > {{< quote >}} <https://github.com/coreboot/coreboot/blob/381860454fb5a1a6ffc4c8d1fdf3f021f75cbcbc/src/include/device/pci_ids.h#L3857-L3860> {{< /quote >}}

[^jsl-ids]: [Compute Global Device ID - 005 - ID:633935 | Intel® Pentium® Silver and Intel® Celeron® Processors Datasheet, Volume 1](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/servers/platforms/intel-pentium-silver-and-intel-celeron-processors-datasheet-volume-1-of-2/005/compute-global-device-id/)

{{< ref >}}
 * [linux/intel-family.h at master · torvalds/linux](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h)
 * [include/pci_ids/i965_pci_ids.h · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/include/pci_ids/i965_pci_ids.h)
 * [include/pci_ids/iris_pci_ids.h · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/main/include/pci_ids/iris_pci_ids.h)
 * [adlrvp: Remove retimer support for N SKU (I0e57fc19) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/3143407)
 * [adlrvp-n : configure typec ports redriver (I78defe36) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/3220931)
 * [Release 9.0.10600.23029 · intel/dptf@e1f10f9 · GitHub](https://github.com/intel/dptf/commit/e1f10f989223720ccb6b2519f8d96435925407c0#diff-51d9502b993a0aa6c66e220b368af805042dc92ec7d724840c89364fa787394f)
 * [coreboot/pci_ids.h at 381860454fb5a1a6ffc4c8d1fdf3f021f75cbcbc · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/381860454fb5a1a6ffc4c8d1fdf3f021f75cbcbc/src/include/device/pci_ids.h)
{{< /ref >}}
