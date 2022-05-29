---
title: "Ampere-1 のマイクロアーキテクチャを見る"
date: 2022-05-27T22:21:08+09:00
draft: false
categories: [ "Ampere", "CPU" ]
tags: [ "Arm", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

クラウド、エッジーサーバー向けに性能、電力効率、コストに優れたプロセッサを開発、提供する Ampere は、先日公開した「Ampere Strategy & Product Roadmap Update, 2022」にて、独自設計のコアを採用する *AmpereOne* を発表した。  

 * [Ampere Strategy & Product Roadmap Update, 2022 - YouTube](https://www.youtube.com/watch?v=rxPt7bpXGSk)
 * [Ampere Roadmap Has Four Future Arm Server Chips](https://www.nextplatform.com/2022/05/27/ampere-roadmap-has-four-future-arm-server-chips/)
 * [AmpereOne Announced As Ampere's In-House AArch64 Cloud Native Processor Design - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=AmpereOne)

Ampere は最大 80コアの *Altra* と最大 128コアの *Altra Max* を既に提供しており、どちらもコアは *Arm Neoverse-N1* ベースであり、DDR4-3200 8ch、PCIe Gen4 128-Lanes をサポートしている。製造プロセスは TSMC 7nm。  
今回発表された *AmpereOne* は DDR5、PCIe Gen5 をサポートし、TSMC 5nmプロセスで製造される。  

*AmpereOne* が採用しているであろう *Ampere-1* マイクロアーキテクチャの最適化情報は既に LLVM、GCC に追加されており、そこから読み取れるマイクロアーキテクチャを *Arm Neoverse-N1* と比較してみる。  

## Neoverse-N1 {#n1}
先に *Neoverse-N1* アーキテクチャに触れると、ISA は Armv8.2、デコード幅は 4命令、L1命令キャッシュサイズは 64KB。  
リネームユニットはデコードされた命令 (micro-ops, uops, Mop) を最大 4命令受け取り、re-order buffer (ROB) は 128エントリ持つ。  
バックエンド部には、Integer ALU 3ポート、ASIMD (FP, SIMD) 2ポート、AGU (Adress Generation Unit, Load/Store) 2ポート、Branchユニット 1ポートを持ち、計 8ポートを備える。パイプラインは 11段。  
L1データキャッシュサイズは 64KB、コアごとに L2キャッシュを持ち、512 KB と 1MB の構成が選択可能。*Ampere Altra/Altra Max* は 1MB の構成を取っている。  

 * [Neoverse N1 – Arm®](https://www.arm.com/ja/products/silicon-ip-cpu/neoverse/neoverse-n1)
 * [Neoverse N1 - Microarchitectures - ARM - WikiChip](https://en.wikichip.org/wiki/arm_holdings/microarchitectures/neoverse_n1)
 * [An interactive guide to the CPU roadmap - 20210818_Hotchips_NeoverseN2.pdf](https://hc33.hotchips.org/assets/program/conference/day1/20210818_Hotchips_NeoverseN2.pdf)

## Ampere-1 {#ampere-1}

 * [[PATCH v1] aarch64: enable Ampere-1 CPU](https://gcc.gnu.org/pipermail/gcc-patches/2021-November/583030.html)
 * [[AArch64] Support for Ampere1 core · llvm/llvm-project@64816e6](https://github.com/llvm/llvm-project/commit/64816e68f4419a9e14c23be8aa96fa412bed7e12)
    * [llvm-project/AArch64SchedAmpere1.td at 64816e68f4419a9e14c23be8aa96fa412bed7e12 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/64816e68f4419a9e14c23be8aa96fa412bed7e12/llvm/lib/Target/AArch64/AArch64SchedAmpere1.td)
    * [llvm-project/AArch64.td at 731f0e27ec110443aa5faaceda4b20adafbddcc7 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/731f0e27ec110443aa5faaceda4b20adafbddcc7/llvm/lib/Target/AArch64/AArch64.td)

*Ampere-1* アーキテクチャの ISA は Armv8.6-A、デコード幅は 4命令。このデコード幅について、GCC へのパッチ内で Philipp Tomsich 氏は、最新のマイクロアーキテクチャと同様に、最大ディスパッチレートとスケジューラに発行される micro-ops の最大レートの中間、妥協点だと語っている。  
また SVE (Scalable Vector Extension) はサポートしない。しかし ISA は Armv8.6-A であり、Bfloat16系命令や Int8 行列演算命令は ASIMD (FP, SIMD) ユニットに実装されている。  

 > 		The Ampere-1 implements the ARMv8.6 architecture in A64 mode and is
 > 		modelled as a 4-wide issue (as with all modern micro-architectures,
 > 		the chosen issue rate is a compromise between the maximum dispatch
 > 		rate and the maximum rate of uops issued to the scheduler).
 >
 > {{< quote >}} [[PATCH v1] aarch64: enable Ampere-1 CPU](https://gcc.gnu.org/pipermail/gcc-patches/2021-November/583030.html) {{< /quote >}}

リネームユニットへの発行レート (issue rate) も *Neoverse-N1* と同じ 4命令とされる。  
ROB は 172エントリとなり、*Neoverse-N1* と比較して x1.5 の規模となる。  

 > 		// The Ampere-1 core is an out-of-order micro-architecture.  The front
 > 		// end has branch prediction, with a 10-cycle recovery time from a
 > 		// mispredicted branch.  Instructions coming out of the front end are
 > 		// decoded into internal micro-ops (uops).
 > 		
 > 		def Ampere1Model : SchedMachineModel {
 > 		  let IssueWidth            =   4;  // 4-way decode and dispatch
 > 		  let MicroOpBufferSize     = 174;  // micro-op re-order buffer size
 > 		  let LoadLatency           =   4;  // Optimistic load latency
 > 		  let MispredictPenalty     =  10;  // Branch mispredict penalty
 > 		  let LoopMicroOpBufferSize =  32;  // Instruction queue size
 > 		  let CompleteModel = 1;
 > 		
 > 		  list<Predicate> UnsupportedFeatures = !listconcat(SVEUnsupported.F,
 > 		                                                    SMEUnsupported.F);
 > 		}
 >
 > {{< quote >}} [llvm-project/AArch64SchedAmpere1.td at 64816e68f4419a9e14c23be8aa96fa412bed7e12 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/64816e68f4419a9e14c23be8aa96fa412bed7e12/llvm/lib/Target/AArch64/AArch64SchedAmpere1.td#L14-L29) {{< /quote >}}

バックエンド部には Integer ALU 4ポート、AGU (Load/Store) 2ポート、ASIMD (SIMD, FP, Cypto) 2ポートを備える。  
計 8ポートと、*Neoverse-N1* と同じポート数であり、AGU と ASIMD は基本的に同じ構成と思われるが、Integer ALU の構成は異なっているように見える。  

 > 		//===----------------------------------------------------------------------===//
 > 		// Define each kind of processor resource and number available on Ampere-1.
 > 		// Ampere-1 has 12 pipelines that 8 independent scheduler (4 integer, 2 FP,
 > 		// and 2 memory) issue into.  The integer and FP schedulers can each issue
 > 		// one uop per cycle, while the memory schedulers can each issue one load
 > 		// and one store address calculation per cycle.
 > 		
 > 		def Ampere1UnitA  : ProcResource<2>;  // integer single-cycle, branch, and flags r/w
 > 		def Ampere1UnitB  : ProcResource<2>;  // integer single-cycle, and complex shifts
 > 		def Ampere1UnitBS : ProcResource<1>;  // integer multi-cycle
 > 		def Ampere1UnitL  : ProcResource<2>;  // load
 > 		def Ampere1UnitS  : ProcResource<2>;  // store address calculation
 > 		def Ampere1UnitX  : ProcResource<1>;  // FP and vector operations, and flag write
 > 		def Ampere1UnitY  : ProcResource<1>;  // FP and vector operations, and crypto
 > 		def Ampere1UnitZ  : ProcResource<1>;  // FP store data and FP-to-integer moves
 >
 > {{< quote >}} [llvm-project/AArch64SchedAmpere1.td at 64816e68f4419a9e14c23be8aa96fa412bed7e12 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/64816e68f4419a9e14c23be8aa96fa412bed7e12/llvm/lib/Target/AArch64/AArch64SchedAmpere1.td#L33-L47) {{< /quote >}}
 
基本的な算術演算命令は最大 4ポート (2x UnitA, 2x UnitB) で実行可能であり、Branch命令は最大 2ポート (2x UnitA)、除算命令のような複数サイクルを必要とする複雑な命令は 1ポート (1x UnitBS {UnitB?}) で実行される。  

 > 		// For basic arithmetic, we have more flexibility for short shifts (LSL shift <= 4),
 > 		// which are a single uop, and for extended registers, which have full flexibility
 > 		// across Unit A or B for both uops.
 > 		def Ampere1Write_Arith : SchedWriteVariant<[
 > 		                                SchedVar<RegExtendedPred, [Ampere1Write_2cyc_2AB]>,
 > 		                                SchedVar<AmpereCheapLSL,  [Ampere1Write_1cyc_1AB]>,
 > 		                                SchedVar<NoSchedPred,     [Ampere1Write_2cyc_1B_1AB]>]>;
 > 		
 > 		def Ampere1Write_ArithFlagsetting : SchedWriteVariant<[
 > 		                                SchedVar<RegExtendedPred, [Ampere1Write_2cyc_1AB_1A]>,
 > 		                                SchedVar<AmpereCheapLSL,  [Ampere1Write_1cyc_1A]>,
 > 		                                SchedVar<NoSchedPred,     [Ampere1Write_2cyc_1B_1A]>]>;
 >
 > {{< quote >}} [llvm-project/AArch64SchedAmpere1.td at 64816e68f4419a9e14c23be8aa96fa412bed7e12 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/64816e68f4419a9e14c23be8aa96fa412bed7e12/llvm/lib/Target/AArch64/AArch64SchedAmpere1.td#L567-L578) {{< /quote >}}


GCC 内の最適化情報では、*Ampere-1* の L2キャッシュサイズは 2048KB となっている。  
これがコアごとのサイズであれば、*Neoverse-N1* ベースの *Altra/Altra Max* から倍のサイズとなる。  

 > 		+static const cpu_prefetch_tune ampere1_prefetch_tune =
 > 		+{
 > 		+  0,			/* num_slots  */
 > 		+  64,			/* l1_cache_size  */
 > 		+  64,			/* l1_cache_line_size  */
 > 		+  2048,			/* l2_cache_size  */
 > 		+  true,			/* prefetch_dynamic_strides */
 > 		+  -1,			/* minimum_stride */
 > 		+  -1			/* default_opt_level  */
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH v1] aarch64: enable Ampere-1 CPU](https://gcc.gnu.org/pipermail/gcc-patches/2021-November/583030.html) {{< /quote >}}

| Arm            | Neoverse-N1 | Ampere-1 |
| :------------- | :---------: | :------: |
| Decode width   | 4           | 4        |
| ROB            | 128         | 174      |
| Pipeline Depth | 10          | 12       |
| ALU            |3 (+1 Branch)| 4        |
| AGU            | 2           | 2        |
| ASIMD (SIMD,FP)| 2           | 2        |
| Cache Line size| 64          | 64       |
| L2 cache       | 512/1024 KB | 2048 KB  |

*Ampere-1* アーキテクチャは全体的に見て ROB や L2キャッシュサイズの増加といった実行効率の向上を目的としたと思われる点が目立ち、デコード幅やバックエンド部は *Neoverse-N1* と比較して増やしていない。また SVE の実装もされていない。  
*Ampere Altra/Altra Max* 同様に電力効率やソケットあたりのコア数を重視した結果ではないかと思われる。  

{{< ref >}}
 * [Press Release Details](https://amperecomputing.com/press/2020-03-03/ampere-altra---industrys-first-80-core-server-processor-unveiled.html)
 * [BFloat16 extensions for Armv8-A - AI and ML blog - Arm Community blogs - Arm Community](https://community.arm.com/arm-community-blogs/b/ai-and-ml-blog/posts/bfloat16-processing-for-neural-networks-on-armv8_2d00_a)
 * [Arm A profile architecture update 2019 - Architectures and Processors blog - Arm Community blogs - Arm Community](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-architecture-developments-armv8-6-a)
{{< /ref >}}
