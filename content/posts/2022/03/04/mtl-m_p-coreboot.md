---
title: "Coreboot に Meteor Lake-M/P の DeviceID が追加される"
date: 2022-03-04T16:58:58+09:00
draft: true
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Meteor_Lake", "Coreboot" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Wonkyu Kim 氏より、Coreboot に *Meteor Lake-M/P* の CPUID/DeviceID (PCI ID) を追加するパッチが投稿されている。  

* [soc/intel/common: Include Meteorlake device IDs (Ie71abb70) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/62581/1/)
* [topic:MTL_UPSTREAM · Gerrit Code Review](https://review.coreboot.org/q/topic:MTL_UPSTREAM)

*Meteor Lake-M/P* の Family, Model, Stepping を示す CPUID は `0xA06AX, (X = Stepping)`。Model は `0xAA (170)` となる。  
今回のパッチで追加された *Meteor Lake-M/P* の CPUID は、[intel/dptf (Intel (R) Dynamic Tuning for Chromium OS)](https://github.com/intel/dptf) 内に記述されているものと一致する。  
少し *Meteor Lake* から外れるが、現時点で *Raptor Lake-P* のサポートを追加するパッチは Coreboot や Linux Kernel の各種ドライバーに投稿されていない。以下の *Raptor Lake-P* の CPUID Model: `0xBA (186)` は、モバイル向け *Rocket Lake* のように内部的に割り当てられた仮の値なのだろうか。  

 > 		#define CPUID_METEORLAKE_1_A0		0xa06a0
 > 		#define CPUID_METEORLAKE_2_A0		0xa06a1
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/62581/1/src/include/cpu/intel/cpu_ids.h> {{< /quote >}}

 > 		#define CPUID_FAMILY_MODEL_MTL_M_P	0x000A06A0		// Meteor Lake M & P
 > 		#define CPUID_FAMILY_MODEL_MTL_N	0x000A06B0		// Meteor Lake N
 > 		#define CPUID_FAMILY_MODEL_MTL_S	0x000606C0		// Meteor Lake S
 > 		#define CPUID_FAMILY_MODEL_RPL_S	0x000B0670		// Raptor Lake S
 > 		#define CPUID_FAMILY_MODEL_RPL_P	0x000B06A0		// Raptor Lake P
 >
 > {{< quote >}} <https://github.com/intel/dptf/blob/e1f10f989223720ccb6b2519f8d96435925407c0/Common/esif_ccb_cpuid.h#L108-L112> {{< /quote >}}

* [soc/intel/common: Add the Primary to Sideband bridge library (I63f58584) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61283)
