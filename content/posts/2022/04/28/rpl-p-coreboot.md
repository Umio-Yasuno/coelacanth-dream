---
title: "各種 OSS で Raptor Lake-P のサポートが進められる"
date: 2022-04-28T10:01:23+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Raptor_Lake", "Alder_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Bora Guvendik 氏より、Coreboot に *Raptor Lake-P* の CPUID/DeviceID (PCI ID) を追加するパッチが投稿されている。  

 * [soc/intel: Add Raptor Lake device IDs (I39e655de) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/63750/7)

Coreboot には他に、ベースは *Brya* ボードだが、*Alder Lake-P/M* の代わりに *Raptor Lake-P* を搭載する *Skolas* ボードのサポートを追加するパッチも投稿されている。  
Coreboot では *Raptor Lake-P* より先に、*Meteor Lake-P/M* の CPUID/DeviceID を追加するパッチが投稿されており、モバイル向け *Rocket Lake* のように表には出てこない可能性を考えたが違っていたらしい。  
{{< link >}} [Coreboot に Meteor Lake-M/P の DeviceID が追加される | Coelacanth's Dream](/posts/2022/03/04/mtl-m_p-coreboot/) {{< /link >}}

 * [mb/google/brya: Add new skolas baseboard (Iec100306) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/63891)

他 OSS でも *Raptor Lake-P* の CPUID Model、GPU DeviceID を追加するパッチが投稿されており、サポートが進められている。  

## CPUID {#cpuid}

*Raptor Lake-P (J0)* の CPUID は `0xb06a2 (Family: 0x6, Model: 0xba, Stepping: 0x2)` となり、これは [intel/dptf](https://github.com/intel/dptf) (Intel (R) Dynamic Tuning for Chromium OS) にある情報とも[^dptf-cpuid]、Linux Kernel に追加された情報とも一致する。  
{{< link >}} [N バリアントも存在する Alder Lake | Coelacanth's Dream](/posts/2021/11/16/coreboot-intel-adl_n/) {{< /link >}}

 > 		#define CPUID_RAPTORLAKE_P_J0		0xb06a2
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/63750/7/src/include/cpu/intel/cpu_ids.h#62> {{< /quote >}}

[^dptf-cpuid]: <https://github.com/intel/dptf/blob/e1f10f989223720ccb6b2519f8d96435925407c0/Common/esif_ccb_cpuid.h#L108-L112>

Linux Kernel では、*Alder Lake-N* と *Raptor Lake-P* の CPUID Model が追加され、同時に *N, P* バリアントがより細分化されたモバイルセグメントを示すことが、Intel の "Tony Luck" 氏より説明されている。  
*Alder Lake-N* も CPUID Model については、ドキュメントでは別の Model、`0xBF` が記載され、コンパイラ等ではそれに従っているが、今後修正されるのだろうか。 [^adl-n]  

 * [[PATCH] x86/cpu: Add new Alderlake and Raptorlake CPU model numbers - Luck, Tony](https://lore.kernel.org/all/YlS7n7Xtso9BXZA2@agluck-desk3.sc.intel.com/)

[^adl-n]: [Intel Alder Lake と Rocket Lake に追加される CPUID Model | Coelacanth's Dream](/posts/2022/01/07/intel-adl-rkl-new-model/)

## GPU {#gpu}

*Raptor Lake-P* の GPU部については、概ね *Alder Lake-P/M/N* と同じであり、現状 DeviceID を追加するのみの対応が取られている。  
デスクトップ向けの *Raptor Lake-S* では、*Alder Lake-S* から GuC submission がデフォルトで有効化されるという変更点があったが[^rpl-s-gpu]、*Alder Lake-P/M/N* では最初から GuC submission が有効化されていた。  
このことも *Raptor Lake-P* の対応にあたって変更が小さい理由にあると思われる。  

[^rpl-s-gpu]: [Alder Lake-S と同じ GPU IP となる Raptor Lake-S | Coelacanth's Dream](/posts/2021/11/15/intel-rpl/)

 * [[Intel-gfx] [PATCH V2] drm/i915/rpl-p: Add PCI IDs](https://lists.freedesktop.org/archives/intel-gfx/2022-April/295950.html)
 * [Add RPL-P DIDs (#98) · intel/gmmlib@c70167b](https://github.com/intel/gmmlib/commit/c70167bee652c7f4cc656d651a1705ffb6bcc0c9)
 * [Add RPL-P device IDs · intel/compute-runtime@3e19f39](https://github.com/intel/compute-runtime/commit/3e19f39b8d784b3eff6e5ef18bad63e68e0a53d5)
