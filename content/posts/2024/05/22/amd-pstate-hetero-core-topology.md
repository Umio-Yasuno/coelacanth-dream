---
title: "amd-pstate ドライバーにおける異種コア構成のトポロジへの対応"
date: 2024-05-22T22:17:28+09:00
draft: false
categories: [ "AMD", "CPU", "APU", "Hardware" ]
tags: [ "Zen_4", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Perry Yuan 氏により、最新の AMD APU/CPU 向けの CPU 周波数ドライバー `amd-pstate` に異種コア構成のトポロジへの対応を追加するパッチが投稿された。  

 * [[PATCH v2 09/10] cpufreq: amd-pstate: implement heterogeneous core topology for highest performance initialization - Perry Yuan](https://lore.kernel.org/linux-pm/c632dbb4ee183e185d1fa408c7b5511765d1c711.1715356532.git.perry.yuan@amd.com/)

パッチでは、まず `CPUID` 命令から取得できる `HeterogeneousCoreTopology (0x8000_0026:EAX:Bit30)` を見て、それがサポートされている場合は `CoreType (0x8000_0026:EBX:Bit31-28)` を取得する。  
`CoreType` は以下のように定義されている。  

 >         +/* defined by CPUID_Fn80000026_EBX BIT [31:28] */
 >         +enum amd_core_type {
 >         +	CPU_CORE_TYPE_NO_HETERO_SUP = -1,
 >         +	CPU_CORE_TYPE_PERFORMANCE = 0,
 >         +	CPU_CORE_TYPE_EFFICIENCY = 1,
 >         +	CPU_CORE_TYPE_UNDEFINED = 2,
 >         +};
 >         +
 >
 > {{< quote >}} [[PATCH v2 09/10] cpufreq: amd-pstate: implement heterogeneous core topology for highest performance initialization - Perry Yuan](https://lore.kernel.org/linux-pm/c632dbb4ee183e185d1fa408c7b5511765d1c711.1715356532.git.perry.yuan@amd.com/) {{< /quote >}}

`CPU_CORE_TYPE_PERFORMANCE` の場合は `highest_perf` に 196、`CPU_CORE_TYPE_EFFICIENCY` の場合は 132、それ以外か `HeterogeneousCoreTopology` をサポートしていない場合は 166 がセットされるようになっている。  

 >         +	switch (core_type) {
 >         +	case CPU_CORE_TYPE_NO_HETERO_SUP:
 >         +		highest_perf = CPPC_HIGHEST_PERF_DEFAULT;
 >         +		break;
 >         +	case CPU_CORE_TYPE_PERFORMANCE:
 >         +		highest_perf = CPPC_HIGHEST_PERF_PERFORMANCE;
 >         +		break;
 >         +	case CPU_CORE_TYPE_EFFICIENCY:
 >         +		highest_perf = CPPC_HIGHEST_PERF_EFFICIENT;
 >         +		break;
 >         +	default:
 >         +		highest_perf = CPPC_HIGHEST_PERF_DEFAULT;
 >         +		WARN_ONCE(true, "WARNING: Undefined core type found");
 >         +		break;
 >         +	}
 >
 > {{< quote >}} [[PATCH v2 09/10] cpufreq: amd-pstate: implement heterogeneous core topology for highest performance initialization - Perry Yuan](https://lore.kernel.org/linux-pm/c632dbb4ee183e185d1fa408c7b5511765d1c711.1715356532.git.perry.yuan@amd.com/) {{< /quote >}}

Intel CPU ではハイブリッド構成、異種コア構成におけるスケジューリングにおいて `HFI (Hardware Feedback Interface)` や `ITD (Intel Thread Director)` といったハードウェア側からのヒントを用いることを可能としているが、言い換えれば複雑でもある。  
一方、AMD CPU は現状 *Zen 4* に対する *Zen 4c* のような、サポートする拡張命令も IPC も同じで、(システムから見れば) 動作クロックのみが異なるコア実装を採っているため、パッチのような CPPC に関連する値の調節だけで対応が済むのだと思われる。  

それと *Zen 4* と *Zen 4c* で構成された AMD APU として *Phoenix2* が存在するが、`HeterogeneousCoreTopology` と `CoreType` をサポートしていない可能性がある。  
*Phoenix2* の `CPUID` 実行結果が InstLatx64 氏のリポジトリにて公開されているが、どちらも 0 となっており、`HeterogeneousCoreTopology` はサポートせず、すべての `CoreType` は `Performance Core` と読み取れる。  

 * <https://github.com/InstLatx64/InstLatx64/blob/master/AuthenticAMD/AuthenticAMD0A70F80_K19_Phoenix2_01_CPUID.txt>

そのため、このパッチは *Phoenix2* に対しては想定と異なる動作をする可能性はあるが、元から Preferred Core の指定で対応していたか、最新の BIOS/UEFI では異なる `CPUID` 実行結果となる可能性も考えられる。  
どちらにせよ自分は実機を持っていないため検証できない。  

{{< ref >}}
 * <https://www.kernel.org/doc/html/latest/admin-guide/pm/amd-pstate.html>
{{< /ref >}}
