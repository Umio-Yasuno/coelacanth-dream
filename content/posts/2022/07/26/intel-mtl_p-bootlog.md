---
title: "Intel Meteor Lake-P Bootlog"
date: 2022-07-26T11:32:57+09:00
draft: false
categories: [ "Intel", "CPU", "Hardware" ]
tags: [ "Meteor_Lake", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Github 上の Sound Open Firmware (SOF) プロジェクトにおいて、Intel の Fred Oh 氏により、*Meteor Lake-P (Family: 0x6, Model: 0xAA, Stepping: 0x0)* の Linux Kernel Bootlog がアップロードされている。  

 * [MTL has a kernel RIP on build_sched_domains · Issue #3784 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3784)

## Meteor Lake-P {#mtl_p}
プラットフォームは *Intel Corporation Meteor Lake Client Platform/MTL-P DDR5 SODIMM SBS RVP*、ベースクロックは約 1.2 GHz と検出されている。  

プロセッサ名は *Genuine Intel(R) 0000* であり ES品とされ、ステッピングは `0x0 (A0)` となる。  
CPUID Family/Model は Coreboot *Meteor Lake-M/P* の情報と一致する。  

 > 		[    0.000000] SMBIOS 3.4 present.
 > 		[    0.000000] DMI: Intel Corporation Meteor Lake Client Platform/MTL-P DDR5 SODIMM SBS RVP, BIOS MTLPFWI1.R00.2223.D01.2205300653 05/30/2022
 > 		[    0.000000] tsc: Detected 1200.000 MHz processor
 > 		[    0.000000] tsc: Detected 1190.400 MHz TSC
 >
 > {{< quote >}} [MTL has a kernel RIP on build_sched_domains · Issue #3784 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3784) {{< /quote >}}
 >
 > 		[    0.241340] smpboot: CPU0: Genuine Intel(R) 0000 @ (family: 0x6, model: 0xaa, stepping: 0x0)
 >
 > {{< quote >}} [MTL has a kernel RIP on build_sched_domains · Issue #3784 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3784) {{< /quote >}}

`FPU, XFEATURE, XSAVE, XSTATE` に関する出力メッセージから、*Alder Lake, Raptor Lake* に続き *Meteor Lake-P* でも AVX512 がサポートされていないことが推察される。  

 > 		[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
 > 		[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
 > 		[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
 > 		[    0.000000] x86/fpu: Supporting XSAVE feature 0x200: 'Protection Keys User registers'
 > 		[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
 > 		[    0.000000] x86/fpu: xstate_offset[9]:  832, xstate_sizes[9]:    8
 > 		[    0.000000] x86/fpu: Enabled xstate features 0x207, context size is 840 bytes, using 'compacted' format.
 >
 > {{< quote >}} [MTL has a kernel RIP on build_sched_domains · Issue #3784 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3784) {{< /quote >}}

検出されている CPU数 (スレッド数) だが、アップロードされている 3個の Bootlog で分かれており、16CPU と 20CPU の 2種類のパターンが確認できる。  
以下引用部のメッセージを出力している部分の Linux Kernel のソースコードは `arch/x86/kernel/smpboot.c`。  

 > 		[    0.245426] .... node  #0, CPUs:        #1
 > 		[    0.006567] smpboot: CPU 1 Converting physical 0 to logical package 1
 > 		[    0.006567] smpboot: CPU 1 Converting physical 0 to logical die 1
 > 		[    0.271919]   #2  #3  #4  #5
 > 		[    0.006567] smpboot: CPU 5 Converting physical 1 to logical package 2
 > 		[    0.006567] smpboot: CPU 5 Converting physical 0 to logical die 2
 > 		[    0.347873]   #6  #7  #8  #9 #10
 > 		[    0.006567] smpboot: CPU 10 Converting physical 5 to logical package 3
 > 		[    0.006567] smpboot: CPU 10 Converting physical 0 to logical die 3
 > 		[    0.421872]  #11 #12
 > 		[    0.006567] smpboot: CPU 12 Converting physical 6 to logical package 4
 > 		[    0.006567] smpboot: CPU 12 Converting physical 0 to logical die 4
 > 		[    0.447880]  #13 #14
 > 		[    0.006567] smpboot: CPU 14 Converting physical 7 to logical package 5
 > 		[    0.006567] smpboot: CPU 14 Converting physical 0 to logical die 5
 > 		[    0.473863]  #15
 > 		[    0.483729] smp: Brought up 1 node, 16 CPUs
 > 		[    0.483729] smpboot: Max logical packages: 8
 > 		[    0.483729] smpboot: Total of 16 processors activated (38135.80 BogoMIPS)
 >
 > {{< quote >}} 0725_dmesg01_mtlp_sdw_kenel_rip.log (Linux version 5.19.0-rc2-daily-default-20220724-0-g70f9e7b34a9d) {{< /quote >}}
 >
 > 		[    0.246496] smp: Bringing up secondary CPUs ...
 > 		[    0.246862] x86: Booting SMP configuration:
 > 		[    0.246863] .... node  #0, CPUs:        #1
 > 		[    0.006563] smpboot: CPU 1 Converting physical 0 to logical package 1
 > 		[    0.006563] smpboot: CPU 1 Converting physical 0 to logical die 1
 > 		[    0.276779]   #2  #3  #4  #5
 > 		[    0.006563] smpboot: CPU 5 Converting physical 1 to logical package 2
 > 		[    0.006563] smpboot: CPU 5 Converting physical 0 to logical die 2
 > 		[    0.350763]   #6  #7  #8  #9 #10
 > 		[    0.006563] smpboot: CPU 10 Converting physical 0 to logical die 3
 > 		[    0.421748]  #11 #12
 > 		[    0.006563] smpboot: CPU 12 Converting physical 0 to logical die 4
 > 		[    0.447765]  #13 #14
 > 		[    0.006563] smpboot: CPU 14 Converting physical 0 to logical die 5
 > 		[    0.473749]  #15 #16
 > 		[    0.006563] smpboot: CPU 16 Converting physical 0 to logical die 6
 > 		[    0.498738]  #17 #18
 > 		[    0.006563] smpboot: CPU 18 Converting physical 0 to logical die 7
 > 		[    0.522741]  #19
 > 		[    0.532758] smp: Brought up 1 node, 20 CPUs
 > 		[    0.532758] smpboot: Max logical packages: 10
 > 		[    0.532758] smpboot: Total of 20 processors activated (47671.29 BogoMIPS)
 >
 > {{< quote >}} 0725_dmesg01_mtlp_nocodec_rip_build_sched_domains.log (5.19.0-rc2-daily-nocodec-20220724-0-g70f9e7b34a9d) {{< /quote >}}

これについては別の *Meteor Lake-P* による Bootlog がアップロードされたというよりは、恐らく `CPUID` 命令経由で取得できる APIC ID と Package/Die ID でズレがあり、その結果ではないかと思われる。  
Coreboot の *Meteor Lake-M/P* への対応が主に進めている Subrata Banik 氏によれば、*Meteor Lake* では `XAPIC (MMIO)` と `X2APIC (MSR)` 間の動的な切り替えに対応している。[^mtl-apic]  
検出された CPU数のズレは用いたプラットフォームの BIOS/UEFI が十分に対応していない、Linux Kernel に対応するためのパッチがまだ公開されていないためであることが考えられる。  

[^mtl-apic]: [cpu/x86: Improve help text for `X2APIC_RUNTIME` config (Ia736efa0) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65738/4)

Subrata Banik 氏は以前は Intel に、現在は Google に所属している。  

{{< ref >}}
 * [Intel Raptor Lake-S Bootlog | Coelacanth's Dream](/posts/2022/01/07/intel-rpl_s-bootlog/)
 * [Coreboot に Meteor Lake-M/P の DeviceID が追加される | Coelacanth's Dream](/posts/2022/03/04/mtl-m_p-coreboot/)
 * [4. x86 Topology — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/x86/topology.html)
 * [Local APICについて - 睡分不足](https://mmi.hatenablog.com/entry/2017/03/27/202656)
 * [cpu/x86: Improve help text for `X2APIC_RUNTIME` config (Ia736efa0) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/65738/4)
{{< /ref >}}
