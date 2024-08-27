---
title: "AMD CPU も Intel Thread Director に近い機能を実装か"
date: 2024-08-27T22:48:06+09:00
draft: false
categories: [ "CPU", "AMD" ]
tags: [ "CPUID", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Perry Yuan 氏により、Linux Kernel に AMD Hetero Core Hardware Feedback Driver (`amd_hfi`) を追加するパッチが公開されている。  

 * [[PATCH 00/11] Introduce New AMD Heterogeneous Core Driver - Perry Yuan](https://lore.kernel.org/platform-driver-x86/cover.1724748733.git.perry.yuan@amd.com/)

パッチ内では、AMD におけるヘテロジニアスアーキテクチャの目標はバックグラウンドスレッドを Dense (高密度, "c" 系) コアに割り当て、優先度の高いスレッドを Classic (高性能, 無印系) コアに割り当てることで消費電力の観点でメリットを得ることだと説明されている。  
バックグラウンドスレッドを Dense コアに割り当てれば、処理に必要な電力は少なくて済み、その分を Classic コアにまわすことで優先度の高いスレッドをより速く処理することができる。  
また Dense コアを増やすことでマルチスレッド性能も向上させられる。  

Hardware Feedback Interface (HFI) は実行中のスレッドの分類機能もあり、分類とその Class ID、割り当てられるのが望ましいコアの種類、優先度の例が以下のテーブル。  

 >         +Thread Classification Example Table
 >         +^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 >         ++----------+----------------+-------------------------------+---------------------+---------+
 >         +| class ID | Classification | Preferred scheduling behavior | Preemption priority | Counter |
 >         ++----------+----------------+-------------------------------+---------------------+---------+
 >         +| 0        | Default        | Performant                    | Highest             |         |
 >         ++----------+----------------+-------------------------------+---------------------+---------+
 >         +| 1        | Non-scalable   | Efficient                     | Lowest              | PMCx1A1 |
 >         ++----------+----------------+-------------------------------+---------------------+---------+
 >         +| 2        | I/O bound      | Efficient                     | Lowest              | PMCx044 |
 >         ++----------+----------------+-------------------------------+---------------------+---------+
 >         +
 >
 > {{< quote >}} [[PATCH 01/11] Documentation: x86: Add AMD Hardware Feedback Interface documentation - Perry Yuan](https://lore.kernel.org/platform-driver-x86/c4c66679fc6f8432c0402c8def2dc1b09eaa812d.1724748733.git.perry.yuan@amd.com/) {{< /quote >}}

BIOS/UEFI から提供され、共有メモリに配置される CPU コアのランキングデータも `amd_hfi` ドライバーはスケジューリングに活用する。  
ソースコードからドライバーの動作を完全に読み取れてはいないが、自分の理解で書くと、ランキングデータにはすべての CPU コア (スレッド) 分の性能と電力効率の能力を示す値が含まれているのだろう。
ドライバーは CPU コア (スレッド) から実行中のスレッドの分類を読み取り、そしてそのコア (スレッド) が処理に対して最適でない場合、ランキングデータから最適なコア (スレッド) を選択する。  

機能としては概ね Intel HFI, Intel Thread Director (ITD), EHFI (Enhanced HFI) と同じと見ていいだろう。  
ただしスレッドの分類は異なる。  

AMD のヘテロジニアスアーキテクチャは Intel のそれとは異なり、マイクロアーキテクチャと IPC は同一であるため、CPPC (Collaborative Processor Performance Control) に関連する値の設定を調整することで充分適したスケジューリングができると自分は考えていたが、実際は性能と電力効率の最大化にはハードウェアからヒントを用いたスケジューリングが必要だったのだろう。  

 * [amd-pstate ドライバーにおける異種コア構成のトポロジへの対応 | Coelacanth's Dream](/posts/2024/05/22/amd-pstate-hetero-core-topology/)

AMD HFI, Workload Classification の機能フラグビットは `CPUID 0x8000_0021:EAX:Bit22` とされている。  

 >         +	{ X86_FEATURE_WORKLOAD_CLASS,   CPUID_EAX,  22, 0x80000021, 0 },
 >
 > {{< quote >}} [[PATCH 03/11] x86/cpufeatures: add X86_FEATURE_WORKLOAD_CLASS feature bit - Perry Yuan](https://lore.kernel.org/platform-driver-x86/067b306ea3457091d5ac97b0cc5b41fcadac10d2.1724748733.git.perry.yuan@amd.com/) {{< /quote >}}

AMD HFI に関する情報は `CPUID 0x8000_0027` から読み取ることができる。なお `CPUID 0x8000_0027` はまだ AMD が公開しているドキュメントには記載されていない。  
`CPUID 0x8000_0027:EAX` がサポートしているスレッドのクラス (分類) 数を示す。  

 >         +	nr_class_id = cpuid_eax(AMD_HETERO_CPUID_27);
 >         +	if (nr_class_id < 0 || nr_class_id > 255) {
 >         +		dev_warn(dev, "failed to get supported class number from CPUID %d\n",
 >         +				AMD_HETERO_CPUID_27);
 >         +		return -EINVAL;
 >         +	}
 >
 > {{< quote >}} [[PATCH 05/11] platform/x86: hfi: Introduce AMD Hardware Feedback Interface Driver - Perry Yuan](https://lore.kernel.org/platform-driver-x86/8d3d75941a87839ab88dff20e89b73c0f5e6cb2a.1724748733.git.perry.yuan@amd.com/) {{< /quote >}}

また、`CPUID 0x8000_0027:EBX` がランキングテーブルの更新モードに関するフラグ等を示すと思われる。  

 >         +	amd_hfi_data->hfi_meta->dynamic_rank_feature =
 >         +					cpuid_ebx(AMD_HETERO_CPUID_27) & 0xF;
 >
 > {{< quote >}} [[PATCH 05/11] platform/x86: hfi: Introduce AMD Hardware Feedback Interface Driver - Perry Yuan](https://lore.kernel.org/platform-driver-x86/8d3d75941a87839ab88dff20e89b73c0f5e6cb2a.1724748733.git.perry.yuan@amd.com/) {{< /quote >}}

InstaLatx64 氏の Github リポジトリで公開されている Ryzen 7 9700X の CPUID 実行結果一覧を見ると[^cpuid_result]、`CPUID 0x8000_0021:EAX:Bit22` フラグビットが立っており、また `CPUID 0x8000_0027:EAX` が 3 となっているため、AMD HFI は *Zen 5/c* からサポートしていると考えられる。  

[^cpuid_result]: https://github.com/InstLatx64/InstLatx64/blob/4b94fc56e1d959d5124fae901679b434263bdd6e/AuthenticAMD/AuthenticAMD0B40F40_K20_GraniteRidge_03_CPUID.txt#L82

Ryzen AI 300 Series は *Zen 5* と *Zen 5c* のヘテロジニアスアーキテクチャを採っており、`amd_hfi` ドライバーによる最適化が期待される。  
西川善司氏による 4Gamer.net の記事にて、Windows では I/O 処理スレッド等は *Zen 5c* に積極的に割り当てるようになっていることが書かれており、Windows では既に先行して `amd_hfi` ドライバーと同様の機能が実装されていると考えられる。[^4gamer]  

[^4gamer]: [「Ryzen AI 300」とはどんなプロセッサなのか。高効率Zen 5cコアに新世代NPUとPS5を超えるGPUを組み合わせる［西川善司の3DGE］](https://www.4gamer.net/games/804/G080449/20240729031/)

