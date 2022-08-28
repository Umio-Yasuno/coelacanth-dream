---
title: "Linux Kernel における Alder Lake のスケジューリング改善"
date: 2022-08-28T11:45:21+09:00
draft: false
categories: [ "Software", "Intel", "CPU" ]
tags: [ "Alder_Lake", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Linux Kernel では現状、Windows11 の Intel Thread Director のような Intel EHFI (Enhanced Hardware Feedback Interface) を活用したハイブリッドアーキテクチャ向けのスケジューリングは実装されていない。[^intel-hfi]  
しかし、*Alder Lake* 等のハイブリッドアーキテクチャに特化はしていないが、*Alder Lake* が性能向上の恩恵を受けられるスケジューリングの改善は行われている。  

[^intel-hfi]: [Linux Kernel に Intel HFI をサポートするパッチ | Coelacanth's Dream](/posts/2022/01/02/intel-hfi/)

## P-Core, E-Core, P-Core (SMT) {#pse}
スケジューリングの改善は主に P-Core, P-Core (SMT), E-Core の各優先度の修正に向いている。  

Intel のソフトウェアエンジニアである Ricardo Neri 氏、Len Brown 氏、Peter Zijlstra 氏らによって作成されたパッチが 2021/04/05 に、メインラインに取り込まれた v5 リビジョンは 2021/09/11 に投稿されている。  

 * [[PATCH v5 0/6] sched/fair: Fix load balancing of SMT siblings with ASYM_PACKING - Ricardo Neri](https://lore.kernel.org/all/20210911011819.12184-1-ricardo.neri-calderon@linux.intel.com/)

*Alder Lake* のような P-Core, P-Core (SMT), E-Core のような 3種類のトポロジが存在する CPU では、最初に P-Core、次に E-Core、それから P-Core (SMT) の順にトポロジを選択、タスクを割り当てることがほとんどの場合で有益だとパッチ内でコメントされている。  

 > 		=== Problem statement ===
 > 		++ Load balancing ++
 > 		
 > 		When using asymmetric packing, there exists CPU topologies with three
 > 		priority levels in which only a subset of the physical cores support SMT.
 > 		An instance of such topology is Intel Alderlake, a hybrid processor with
 > 		a mixture of Intel Core (with support for SMT) and Intel Atom CPUs.
 > 		
 > 		On Alderlake, it is almost always beneficial to spread work by picking
 > 		first the Core CPUs, then the Atoms and at last the SMT siblings.
 >
 > {{< quote >}} [[PATCH v5 0/6] sched/fair: Fix load balancing of SMT siblings with ASYM_PACKING](https://lore.kernel.org/all/20210911011819.12184-7-ricardo.neri-calderon@linux.intel.com/T/#u) {{< /quote >}}

*Alder Lake* では P-Core (*Golden Cove*) は SMT (Hyper-Threading) をサポートし 1-Core/2-Thread の構成だが、E-Core (*Gracemont*) は SMT をサポートせず 1-Core/1-Thread の構成となっている。  
パッチ以前では、すでに P-Core に負荷が掛かっている状態であっても、E-Core ではなく P-Core のもう片方のスレッド、P-Core (SMT) をロードバランサーが選択してしまう状態だった。  
自分の推測と想像が入るが、Intel CPU では CPU (APIC ID) の並びが各 P-Core とその P-Core (SMT)、その後に E-Core が続くという順になっているため、ロードバランサーがそのような動作になっていたのではないかと思われる。  
一連のパッチでは、同 P-Core 内の両スレッドの状態をチェックし、それを基に割り当て先を決定するように動作を修正している。  
パッチによって、P-Core (SMT) より E-Core が優先度の低い設定であっても、P-Core がビジー状態にある場合は E-Core が使われるようになる。  

自分は *Alder Lake* 環境を持っていないため検証できていないが、gihyo.jp で公開されている柴田充也 (しばたみつや) 氏の記事によれば、パッチ内でコメントされていたような P-Core, E-Core, P-Core (SMT) の順でタスクが割り当てられている。[^gihyo]  

[^gihyo]: [第721回　新型ベアボーンキットであるDeskMeetと最新CPUであるAlder Lakeで夢のパワフルUbuntuライフ | gihyo.jp](https://gihyo.jp/admin/serial/01/ubuntu-recipe/0721)

その後、2022/08/25 に補完的なパッチが Ricardo Neri 氏、Len Brown 氏によって投稿されている。  

 * [[PATCH 0/4] sched/fair: Avoid unnecessary migrations within SMT domains - Ricardo Neri](https://lore.kernel.org/lkml/20220825225529.26465-1-ricardo.neri-calderon@linux.intel.com/)

一部の Intel CPU は、パッケージ内のいくつかのコアが通常の Boost Clock を超えて動作する Intel Turbo Boost Max Technology 3.0 (ITMT) をサポートしている。  
Linux Kernel のスケジューラーには、ITMT が有効化されているコアの優先度を高くすることで性能を向上させる機能が実装されている。  
だが、そうした ITMT が有効化されたコアの SMT側に低い優先度を割り当てるようにしていたため、状況によっては同コア内で不要なタスク移行が発生していたとパッチ内で Ricardo Neri 氏はコメントしている。  
パッチ (2022/08/25) では、不要なタスク移行を防ぐための修正と、先のパッチ (2021/09/11) で同コア内の両スレッドの状態を確認するようになったことで不要になった部分の削除が行われている。  

{{< ref >}}
 * [[PATCH 0/5] Make Cluster Scheduling Configurable](https://lore.kernel.org/all/20211204091402.GM16608@worktop.programming.kicks-ass.net/T/#u)
 * [Intel® Turbo Boost Max Technology 3.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-max-technology.html)
{{< /ref >}}
