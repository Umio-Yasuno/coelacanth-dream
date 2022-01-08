---
title: "Intel Raptor Lake-S Bootlog"
date: 2022-01-07T20:59:19+09:00
draft: false
tags: [ "Raptor_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Intel GFX CI に次世代 Intel デスクトップ向けプロセッサ *Raptor Lake-S* の Bootlog がアップロードされていた。  

 * [igt@core_hotunplug@unbind-rebind on bat-rpls-1@IGT_6321 Test details](https://intel-gfx-ci.01.org/tree/drm-tip/IGT_6321/bat-rpls-1/igt@core_hotunplug@unbind-rebind.html)

プラットフォーム名は **Intel Corporation Raptor Lake Client Platform/RPL-S ADP-S DDR5 UDIMM CRB**、Base Clock 1.8 GHz で動作している。  

 > 		<6>[    0.000000] DMI: Intel Corporation Raptor Lake Client Platform/RPL-S ADP-S DDR5 UDIMM CRB, BIOS RPLSFWI1.R00.2397.A01.2109300731 09/30/2021
 > 		<6>[    0.000000] tsc: Detected 1800.000 MHz processor
 > 		<6>[    0.000000] tsc: Detected 1804.800 MHz TSC
 >
 > {{< quote >}} <https://intel-gfx-ci.01.org/tree/drm-tip/IGT_6321/bat-rpls-1/boot0.txt> {{< /quote >}}

CPU Name は **Genuine Intel(R) 0000** になっており、ES品とされる。  
CPUID Model は 0xB7、これは以前に投稿されたパッチの情報と一致し、確かに *Raptor Lake-S* だと言える。  
{{< link >}} [Alder Lake-S と同じ GPU IP となる Raptor Lake-S | Coelacanth's Dream](/posts/2021/11/15/intel-rpl/) {{< /link >}}

 > 		<6>[    0.781389] smpboot: CPU0: Genuine Intel(R) 0000 (family: 0x6, model: 0xb7, stepping: 0x0)
 >
 > {{< quote >}} <https://intel-gfx-ci.01.org/tree/drm-tip/IGT_6321/bat-rpls-1/boot0.txt> {{< /quote >}}

*Alder Lake* 同様にハイブリットアーキテクチャを採用していると思われるが、PMU (Performance Monitoring Unit) ドライバがまだ *Raptor Lake-S* に対応していないため、コア構成は読み取れない。  
32 CPUs (Threads) が認識されており、*Alder Lake-S* の 24 Threads (Golden Cove: 8C/16T + Gracemont: 8C/8T) から増加していることは確認できる。  

 > 		<6>[    0.784998] x86: Booting SMP configuration:
 > 		<6>[    0.785013] .... node  #0, CPUs:        #1  #2  #3  #4  #5  #6  #7  #8  #9 #10 #11 #12 #13 #14 #15 #16 #17 #18 #19 #20 #21 #22 #23 #24 #25 #26 #27 #28 #29 #30 #31
 > 		<6>[    0.895973] smp: Brought up 1 node, 32 CPUs
 > 		<6>[    0.895973] smpboot: Max logical packages: 1
 > 		<6>[    0.895973] smpboot: Total of 32 processors activated (115507.20 BogoMIPS)
 >
 > {{< quote >}} <https://intel-gfx-ci.01.org/tree/drm-tip/IGT_6321/bat-rpls-1/boot0.txt> {{< /quote >}}

また AVX2 までの対応となっており、AVX512 には対応していない。  
*Alder Lake* の *Golden Cove* コアには *Sapphire Rapids* と同じ範囲の AVX512 が実装されているが、*Gracemont* コアが有効な時は ISA を対称とする関係で AVX512 は無効化されていた。[^adl-avx512]  
この点が *Raptor Lake-S* でも続くと見られるが、まだ ES品の段階であり、今後どういった方針を採るかは変更される可能性がある。  

 > 		<6>[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
 > 		<6>[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
 > 		<6>[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
 > 		<6>[    0.000000] x86/fpu: Supporting XSAVE feature 0x200: 'Protection Keys User registers'
 > 		<6>[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
 > 		<6>[    0.000000] x86/fpu: xstate_offset[9]:  832, xstate_sizes[9]:    8
 > 		<6>[    0.000000] x86/fpu: Enabled xstate features 0x207, context size is 840 bytes, using 'compacted' format.
 >
 > {{< quote >}} <https://intel-gfx-ci.01.org/tree/drm-tip/IGT_6321/bat-rpls-1/boot0.txt> {{< /quote >}}

[^adl-avx512]: [Intel Disabled AVX-512, but Not Really - The Intel 12th Gen Core i9-12900K Review: Hybrid Performance Brings Hybrid Complexity](https://www.anandtech.com/show/17047/the-intel-12th-gen-core-i912900k-review-hybrid-performance-brings-hybrid-complexity/2)
