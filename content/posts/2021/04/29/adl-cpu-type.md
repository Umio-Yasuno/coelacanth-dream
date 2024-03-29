---
title: "Alder Lake では BIOS から一方の CPUタイプを無効化可能"
date: 2021-04-29T11:31:24+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Intel Hybrid Technology を採用し、2種類のコア *Golden Cove (big/Core)* と *Gracemont (small/Atom)* を搭載する *Alder Lake* だが、ユーザーが選択可能なオプションとして UEFI/BIOS から一方の CPUタイプを丸々無効化する方法が提供されることがパッチ内のコメントにて明かされた。  

 * [[PATCH V6 20/25] perf/x86/intel: Add Alder Lake Hybrid support - kan.liang](https://lore.kernel.org/lkml/1618237865-33448-21-git-send-email-kan.liang@linux.intel.com/)
 * [LKML: kan.liang@linux ...: [PATCH 20/49] perf/x86/intel: Add Alder Lake Hybrid support](https://lkml.org/lkml/2021/2/8/1190)

 > 		Users may disable all CPUs of the same CPU type on the command line or
 > 		in the BIOS. For this case, perf still register a PMU for the CPU type
 > 		but the CPU mask is 0.
 >
 > {{< quote >}} [[PATCH V6 20/25] perf/x86/intel: Add Alder Lake Hybrid support - kan.liang](https://lore.kernel.org/lkml/1618237865-33448-21-git-send-email-kan.liang@linux.intel.com/) {{< /quote >}}

該当のパッチは Linux Kernel の性能解析ツール (`perf`) に、 *Alder Lake* が持つ 2種類の PMU (Performance Monitoring Unit) のサポートを追加するもので、Intel の Linux Kernel開発者である [Kan Liang](https://ca.linkedin.com/in/kan-liang-07601032) 氏より投稿された。同様のパッチは以前より投稿されており、何度か更新されている。  
そして Linux Kernel のメインラインにマージされたバージョン (v6) のパッチに追加されたコメントの一部が上記引用部である。  
コマンドラインからも無効化できることが示唆されているが、カーネルパラメーターのことと思われ、Linux ではそちらの方が UEFI/BIOS より変更が容易かもしれない。  

Intel Hybrid Technology では主に AVX-512命令等、対応の有無が異なる 2種類の CPU を搭載する場合、それらの命令を無効化することにより、命令レベルでは対称的な CPU とする。  
まだ Intel、Intel に所属するソフトウェアエンジニアより明言された訳ではないが、small/Atom 側の CPU を無効化した場合、AVX-512命令の対応が有効化すると考えられる。  
AVX-512命令を活用したいユーザーにとって、一方の CPUタイプを無効化する手段が提供されることは喜ばしいことだろう。  
逆に big/Core側を無効化して、8コアという多コアな Atomプロセッサとして使うことも可能と思われるが、シングルスレッドのピーク性能は下がり、有効化される命令もそうないはずであり、あまりメリットは無いように思われる。  
それでもユーザーが選択可能ということに意味があり、*Alder Lake* はイジりがいのあるプロセッサとなるかもしれない。  

{{< ins >}}

Intel Architecture Day 2021, Hot Chip 33 にてさらなる *Alder Lake* の詳細が明かされたが、Atom (Small) 側を無効化しても AVX-512 等は有効化されず、また物理的に AVX-512 ユニットを *Alder Lake* の Core (Big) は搭載していないという話も出ている。  

 * [Alder Lake DeepDive - HotChipsで垣間見たIntel「Alder Lake」の細部 | マイナビニュース](https://news.mynavi.jp/article/20210825-1955663/)
 * [Instruction Sets: Alder Lake Dumps AVX-512 in a BIG Way - Intel Architecture Day 2021: Alder Lake, Golden Cove, and Gracemont Detailed](https://www.anandtech.com/show/16881/a-deep-dive-into-intels-alder-lake-microarchitectures/5)

{{< /ins >}}

それと、Arm の big.LITTLE 技術と混同されやすいが、*Alder Lake* の CPUタイプはソースコード中では big (Core - Golden Cove) と small (Atom - Gracemont) として扱われている。 ( *Lakefield* では Big-Bigger なんて言い方が使われてたりもしたが[^lkf-hc31])  

[^lkf-hc31]: [Lakefield: Hybrid Cores in a Three Dimensional Package](https://old.hotchips.org/hc31/HC31_2.10_LKF_HC_2019_Final_v7.pdf)

 > 		+static int adl_hw_config(struct perf_event *event)
 > 		+{
 > 		+	struct x86_hybrid_pmu *pmu = hybrid_pmu(event->pmu);
 > 		+
 > 		+	if (pmu->cpu_type == hybrid_big)
 > 		+		return hsw_hw_config(event);
 > 		+	else if (pmu->cpu_type == hybrid_small)
 > 		+		return intel_pmu_hw_config(event);
 > 		+
 > 		+	WARN_ON(1);
 > 		+	return -EOPNOTSUPP;
 > 		+}
 >
 > {{< quote >}} [[PATCH V6 20/25] perf/x86/intel: Add Alder Lake Hybrid support - kan.liang](https://lore.kernel.org/lkml/1618237865-33448-21-git-send-email-kan.liang@linux.intel.com/) {{< /quote >}}
