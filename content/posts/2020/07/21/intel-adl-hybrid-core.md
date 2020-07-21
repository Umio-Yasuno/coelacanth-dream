---
title: "Intel Alder Lake が ハイブリッドコア構成を取ることが確かに"
date: 2020-07-21T19:58:11+09:00
draft: false
tags: [ "Alder_Lake", "Lakefield" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU" ]
noindex: false
---

Linux Kernel 内の、各 Intelプロセッサのモデルナンバー(`x86_Model`) をまとめた [linux/intel-family.h](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h) に次世代 Intelプロセッサのモデルナンバーを追加するパッチが投稿され、  
その中で、[Alder Lake](/tags/alder_lake) が [Lakefield](/tags/lakefield) 同様にハイブリッドコア構成を取ることが確かとなった。  
{{< link >}} [LKML: Tony Luck: [PATCH] x86/cpu: Add Lakefield, Alder Lake and Rocket Lake to Intel family](https://lkml.org/lkml/2020/7/21/17) {{< /link >}}

   >     +#define INTEL_FAM6_ROCKETLAKE		0xA7
   >     +
   >     #define INTEL_FAM6_SAPPHIRERAPIDS_X	0x8F
   > 
   >     +/* Hybrid Core/Atom Processors */
   >     +
   >     +#define	INTEL_FAM6_LAKEFIELD		0x8A
   >     +#define INTEL_FAM6_ALDERLAKE		0x97
   >     +
   >
   > 引用元: <cite>[LKML: Tony Luck: [PATCH] x86/cpu: Add Lakefield, Alder Lake and Rocket Lake to Intel family](https://lkml.org/lkml/2020/7/21/17)</cite>

同時に [Rocket Lake](/tags/rocket_lake)、そしてようやく *Lakefield* のモデルナンバーも追加されている。  

*Alder Lake* には **Alder Lake-S** と **Alder Lake-P** の 2種類存在することが確認されており、  
{{< link >}} [ChromiumOSへのパッチから Alderlake-S / Alderlake-P を確認 | Coelacanth's Dream](/posts/2020/05/01/vboot-code-add-alderlake/) {{< /link >}}
そして、[linux/intel-family.h](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h) の命名方式に従えば、末尾に `_L` や `_X` が付いてないものはクライアント向けとされているため、今回追加されたモデルナンバーは **Alder Lake-S** にあたるのではないかと思う。  

*Alder Lake* がハイブリッド構成を取るとなると、やはり *Lakefield* に先駆けて GCC に拡張命令セット周りがサポートされたことになるが、  
{{< link >}} [Intel、GCC に Sapphire Rapids と Alder Lake をサポートするパッチを投稿 | Coelacanth's Dream](/posts/2020/07/10/intel-gcc-patch-spr-adl/) {{< /link >}}
その時示された、*Skylake* の対応範囲に `CLDEMOTE /PTWRITE /WAITPK /SERIALIZE` 追加した対応命令が Big Core / small core 両方が対応したものかどうかは気になる所だ。  
*Lakefield* では AVX といった 2種類の内、片方のコアはサポートしているが、片方のコアが対応していない命令のサポートが外されていた。  
示されたのが両方のコアが対応した命令であれば、次世代の Atom系コア(small core) は AVX/2命令をサポートすることとなる。  
これまでの Atom系コアは AVX系命令をサポートしてこなかったため、大きな変化と言えるだろう。  
