---
title: "Alder Lake-P のブートログ"
date: 2021-03-11T15:33:38+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Software", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

Intel の ChromeOSエンジニアである [Sathya Prakash M R](https://in.linkedin.com/in/sathyaprakashmr) 氏により、*Alder Lake* の Linux Kernel ブートログ (`dmesg`) がアップロードされている。  
アップロード先の *Sound Open Firmware* はプロセッサやチップセットに内蔵されている音声処理用 DSP のサポートを提供するオープンソースプロジェクトである。  
プロジェクトには Intel も参加しており、その一環でブートログは投稿されたものと思われる。  

 * [KWD not detecting on ADL · Issue #2782 · thesofproject/linux](https://github.com/thesofproject/linux/issues/2782)

## ブートログ {#boot-log}

CPU の Family、Model は `Family: 0x6, Model: 0x9a` となっており、これはマーケットセグメント的にはモバイル向けの `ALDERLAKE_L` に属する。[^adl_l]  
`ALDERLAKE_L` には、*Alder Lake-P* と *Alder Lake-M* の 2種類が確認されているが、ファームウェア名等から、今回のは *Alder Lake-P* と考えられる。  
{{< link >}} [ALDERLAKE_L と Alder Lake-P/M | Coelacanth's Dream](/posts/2021/02/09/alderlake_l/) {{< /link >}}

 >        [    0.339678] smpboot: CPU0: Genuine Intel(R) 0000 0.80GHz (family: 0x6, model: 0x9a, stepping: 0x0)
 >
 > {{< quote >}} [KWD not detecting on ADL · Issue #2782 · thesofproject/linux](https://github.com/thesofproject/linux/issues/2782) {{< /quote >}}

[^adl_l]: [x86/cpu: Add another Alder Lake CPU to the Intel family · torvalds/linux@6e1239c](https://github.com/torvalds/linux/commit/6e1239c13953f3c2a76e70031f74ddca9ae57cd3#diff-7bf85b32beb96091abd89790e701cd01fb13bafbbbca17433ad47830820c1391)

使用しているボードは **Shadowmountain** 。これは **Brya** とは別にサポートが進められている *Alder Lake-P* ボードとなる。  
**Brya** は Chromebookボードにおける *Alder Lake-P* のリファレンスプラットフォームという役割にあるが、**Shadowmountain** は Pre-CEP (Customer Excellence Program?) [^shadowmountain] として開発されている。  

 >        [    0.000000] DMI: Intel shadowmountain/shadowmountain, BIOS Google_shadowmountain4.13-2419-gedf99a7d7ee-dirty 03/02/2021
 >
 > {{< quote >}} [KWD not detecting on ADL · Issue #2782 · thesofproject/linux](https://github.com/thesofproject/linux/issues/2782) {{< /quote >}}

[^shadowmountain]: [mb/intel/shadowmountain: Add Intel Pre-CEP shadowmountain board (I9cb650c8) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/48685)

CPU の機能に目を向けると、コンパイラ等へのパッチから判明していたように、AVX/2 をサポートしていることが分かる。  
AVX-512 命令は非対応とされているが、これは [Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/content/www/us/en/develop/download/intel-architecture-instruction-set-extensions-programming-reference.html) では *「Intel Hybrid Technology は AVX512 をサポートしない」* とあり、今回はそのハイブリッドプロセッサとして動作している状態であるため、無効化されていると考えられる。  
Intel ハイブリッドプロセッサでは、非対称な CPUアーキテクチャを搭載していても両方が対応している命令のみが有効化され、命令の対応レベルでは対称的となる。  

しかし、*Alder Lake* に採用される Atom (Small) コア *Gracemont* は *Atom系アーキテクチャ* としては初めて AVX/2 をサポートしており、大きな変化と言えるだろう。  
同時に新たに追加された AVX-VNNI (Vector Neural Network Instructions) も *Gracemont* ではサポートされている。  

 >        [    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
 >        [    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
 >        [    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
 >        [    0.000000] x86/fpu: Supporting XSAVE feature 0x200: 'Protection Keys User registers'
 >        [    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
 >        [    0.000000] x86/fpu: xstate_offset[9]:  832, xstate_sizes[9]:    8
 >
 > {{< quote >}} [KWD not detecting on ADL · Issue #2782 · thesofproject/linux](https://github.com/thesofproject/linux/issues/2782) {{< /quote >}}

ブートログからは、ハイブリッドプロセッサにおいて重要なスケジュール機能の一端を垣間見ることができる。  

*Alder Lake* ではスケジュール機能の一部に、Energy Model Framework[^em] を活用しているらしい。  
Energy Model Framework は 2018/12/03 に、当時 Arm に所属していた、ソフトウェアエンジニア Quentin Perret 氏によって Linux Kernel に実装された。Arm CPU に限定はされておらず、アーキテクチャを問わずに用いることができる。  

[^em]: [Energy Model of CPUs — The Linux Kernel documentation](https://www.kernel.org/doc/html/v5.4/power/energy-model.html)

CPU の `em_pd (energy model performance domain)` は、CPU[0-7] と CPU[8-15] の 2つに分かれている。  
ここでの CPU数はスレッド数を表しており、今回の *Alder Lake-P* のコア数は明示されていないが、  
2021/02 頃 Linux Kernel に投稿されたパッチとそれに含まれていたドキュメントから、12-Core/16-Thread (Core [Big]: 4-Core/8-Thread + Atom [Small]: 8-Core/8-Thread) の構成ではないかと思われる。  
パッチは、Linux Kernel のパフォーマンス解析ツール (`perf`) に *Alder Lake* のようなハイブリッドプロセッサのサポートを追加するもので、ドキュメント内にて例として示されていた *Alder Lake-S* は、Core [Big] が cpu[0-15]、Atom [Small] が cpu[16-23] に割り振られている。[^hybrid-doc]  

[^hybrid-doc]: [[PATCH v2 27/27] perf Documentation: Document intel-hybrid support - Jin Yao](https://lore.kernel.org/lkml/20210311070742.9318-28-yao.jin@linux.intel.com/)

 >        [    1.723825] intel-hfi: em_pd for cpu0 null, create it. cpumask:0-7, ee: 2048
 >        [    1.725244] intel-hfi: update em_pd CPU1 span:0-7, ee:2048 >> 2048
 >        [    1.726450] intel-hfi: update em_pd CPU2 span:0-7, ee:2048 >> 2048
 >        [    1.727660] intel-hfi: update em_pd CPU3 span:0-7, ee:2048 >> 2048
 >        [    1.728871] intel-hfi: update em_pd CPU4 span:0-7, ee:2048 >> 2048
 >        [    1.730082] intel-hfi: update em_pd CPU5 span:0-7, ee:2048 >> 2048
 >        [    1.731293] intel-hfi: update em_pd CPU6 span:0-7, ee:2048 >> 2048
 >        [    1.732504] intel-hfi: update em_pd CPU7 span:0-7, ee:2048 >> 2048
 >        [    1.733718] intel-hfi: em_pd for cpu8 null, create it. cpumask:8-15, ee: 2048
 >        [    1.735121] intel-hfi: update em_pd CPU9 span:8-15, ee:2048 >> 2048
 >        [    1.736344] intel-hfi: update em_pd CPU10 span:8-15, ee:2048 >> 2048
 >        [    1.737587] intel-hfi: update em_pd CPU11 span:8-15, ee:2048 >> 2048
 >        [    1.738828] intel-hfi: update em_pd CPU12 span:8-15, ee:2048 >> 2048
 >        [    1.740068] intel-hfi: update em_pd CPU13 span:8-15, ee:2048 >> 2048
 >        [    1.741311] intel-hfi: update em_pd CPU14 span:8-15, ee:2048 >> 2048
 >        [    1.742552] intel-hfi: update em_pd CPU15 span:8-15, ee:2048 >> 2048

 > {{< quote >}} [KWD not detecting on ADL · Issue #2782 · thesofproject/linux](https://github.com/thesofproject/linux/issues/2782) {{< /quote >}}

ハードウェア側では、*Alder Lake* からサポートしている `HFI/EFHI (Enhanced/ Hardware Feedback Interface)` が機能している。  
`HFI/EHFI` は OS のソフトウェアスケジューラーを手助けする機能と言え、メモリに配置されるテーブルとそのテーブルのインデックスから、あるソフトウェアスレッドを性能重視の CPU (Core [Big]) と電力効率重視の CPU (Atom [Small]) のどちらに割り当てるかを OS にアドバイスする。  
OSスケジューラーはそれを元に、性能比あるいは電力効率比を計算し、スレッドを割り当てる。OS として性能を重視するか、電力効率を重視するかの設定も影響する。  
また、テーブルは CPU の消費電力等、状況に応じて動的に更新される。[^ehfi]  

[^ehfi]: [Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/content/www/us/en/develop/download/intel-architecture-instruction-set-extensions-programming-reference.html) (Page 191)

以前、Intel Architecture Day 2020 にて Raja Koduri 氏は、「 *Alder Lake* は *Lakefield* よりも拡張されたハードウェアベースのスケジューラーを実装している」と語っていたが、その中身が `HFI/EHFI (Enhanced/ Hardware Feedback Interface)` ではないかと思われる。  

今調べた限りでは、`HFI/EHFI` をサポートするパッチはまだ Linux Kernel に投稿されておらず、Intel 内部でまだ開発中のハードウェアと、それをサポートする Linux Kernel のログを読めるのは中々珍しい機会だと言えるかもしれない。  

`HFI` と `HEFI` の違いについては、`HFI` は物理コアのみをサポートするが、`EHFI` は SMT による論理コアもサポートする、ということと思われる、たぶん。  
恐らく、正式にパッチが投稿された時、パッチのコメントと実装されたコードから、*Alder Lake* のスケジュラーについてより詳細が明らかにされるだろう。  
