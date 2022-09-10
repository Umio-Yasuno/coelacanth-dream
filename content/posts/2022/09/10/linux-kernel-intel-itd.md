---
title: "Linux Kernel に Intel Thread Director を実装するパッチ"
date: 2022-09-10T23:07:01+09:00
draft: false
categories: [ "Software", "Intel", "CPU" ]
tags: [ "Alder_Lake", "Raptor_Lake", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel のソフトウェアエンジニア Ricardo Neri 氏により、Linux Kernel のスケジューラに Intel Thread Director (ITD) を実装するパッチが投稿されている。  
Ricardo Neri 氏はこれまでにも Linux Kernel における *Alder Lake* のスケジューリングを改善するパッチ[^smt]や HFI (Hardware Feedback Interface) を活用するパッチを投稿している。  
パッチは RFC (Request for Comments) の段階にあり、現在のパッチがそのまま取り込まれるのではなく、いくつかバージョンを重ねた後にメインラインに取り込まれると思われる。  

[^smt]: [Linux Kernel における Alder Lake のスケジューリング改善 | Coelacanth's Dream](/posts/2022/08/28/linux-kernel-alder_lake-sched/)

 * [[RFC PATCH 00/23] sched: Introduce classes of tasks for load balance - Ricardo Neri](https://lore.kernel.org/lkml/20220909231205.14009-1-ricardo.neri-calderon@linux.intel.com/)

用語関係としては、*Alder Lake* からサポートしている EHFI (Enhanced Hardware Feedback Interface) を活用したスケジューリング機能が ITD と認識していたが、パッチでは EHFI を使わず ITD で統一されている。  
おそらくは先に HFI (Hardware Feedback Interface) を活用するパッチがメインラインに取り込まれているため、HFI との混同を避けるために ITD を使っていると思われる。[^intel-hfi]  
HFI は最初の Intel ハイブリッドプロセッサ *Lakefield* からサポートされており、スケジューラに対するヒントとして、性能と電力効率それぞれの相対的なレベルを示す値 (8-bits, 0-255) を論理プロセッサ (スレッド数) 分提供する機能となる。  
ITD (EHFI) は HFI から実行中のスレッドや命令の種類を示す Class の情報が追加され、各 Class に性能と電力効率のレベルが存在する。  
Class を自動で分類し、Class ごとに性能と電力効率の状況を示すレベルを出力するまでがハードウェア側の機能となる。  
また実行中の Class は `MSR (IA32_THREAD_FEEDBACK_CHAR, 0x17D2)` から読み取ることができる。  
現在の Class の分類が、次の Class の分類に影響を与えるのを防ぐため、履歴をリセットする `HRESET` 命令も用意されている。  

[^intel-hfi]: [Linux Kernel に Intel HFI をサポートするパッチ | Coelacanth's Dream](/posts/2022/01/02/intel-hfi/)

ITD (EHFI) の機能フラグビットは `CPUID[LEAF=0x6].EAX[Bit23]` となり、サポートしてる Class の数は `CPUID[LEAF=0x6].ECX[Bit15-08]` から確認することができる。  

 > 		 #define X86_FEATURE_HFI			(14*32+19) /* Hardware Feedback Interface */
 > 		+#define X86_FEATURE_ITD			(14*32+23) /* Intel Thread Director */
 >
 > {{< quote >}} [[RFC PATCH 13/23] x86/cpufeatures: Add the Intel Thread Director feature definitions - Ricardo Neri](https://lore.kernel.org/lkml/20220909231205.14009-14-ricardo.neri-calderon@linux.intel.com/) {{< /quote >}}

## Intel Thread Director {#itd}
パッチは ITD (EHFI) から取得できる Class の概念を導入し、SMT (HTT) における `ASYM_PACKING (asymmetric packing)` と組み合わせることでロードバランサの改善を目的としたものとなる。  
また、Ricardo Neri 氏は、パッチには ITD を使用した Intel ハイブリッドプロセッサの完全な実装 (full implementation) が含まれているとしている。  

パッチでは各 Class の種類について詳細は書かれていないが、Intel が公開している Optimization Reference Manual によれば、以下のような割り当てとなっている。  

| Intel Thread Director | Description |
| :-- | :--: |
| Class 0 | Non-vectorized integer or floating-point code. |
| Class 1 | Integer or floating-point vectorized code,<br>excluding Intel® Deep Learning Boost (Intel® DL Boost) code. |
| Class 2 | Intel DL Boost code. |
| Class 3 | Pause (spin-wait) dominated code. |

Class 0 がベクトル化 (SIMD化) されていない整数演算と浮動小数点演算、Class 1 が VNNI (Vector Neural Network Instructions) 系命令は含まないベクトル化 (SIMD化) された整数演算と浮動小数点演算、Class 2 が VNNI系命令、Class 3 がスピン・ウェイト・ループという割り当てとなる。  
VNNI系命令が Class 1 から分離されているのは、推論処理等を E-Core (Atom, *Gracemont*) で実行することを想定しているのではないかと思われる。  
Hot Chips 33 における *Alder Lake* と Intel Thread Directore の発表では、Class 0 が主要なアプリケーション、Class 1/2 が新しいアプリケーション (Emerging application) とされていた。  

ITD (EHFI) の実装によりタスクごとに Class の情報が追加されることで、例えば同 P-Core 内の 2-Threads に異なる Class のタスク (Class 0 [Int] + Class 1/2 [SIMD]) を割り当てることで P-Core あたりの稼働率とスループットを最大化するといったスケジューリングが可能になると思われる。  

{{< ref >}}
 * Volume 3B: [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html#inpage-nav-3)
 * Optimization Reference Manual: [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html#inpage-nav-5)
 * Alder Lake Architecture: [Title of this template file over multiple lines of text - HC2021.C1.1 Intel Efraim Rotem.pdf](https://hc33.hotchips.org/assets/program/conference/day1/HC2021.C1.1%20Intel%20Efraim%20Rotem.pdf)
 * [Linux Kernel に Intel HFI をサポートするパッチ | Coelacanth's Dream](/posts/2022/01/02/intel-hfi/)
 * [Intel、SDM に Golden Cove、Gracemont アーキテクチャの詳細を追加 | Coelacanth's Dream](/posts/2022/03/04/adl-arch-details/)
 * [Linux Kernel における Alder Lake のスケジューリング改善 | Coelacanth's Dream](/posts/2022/08/28/linux-kernel-alder_lake-sched/)
{{< /ref >}}
