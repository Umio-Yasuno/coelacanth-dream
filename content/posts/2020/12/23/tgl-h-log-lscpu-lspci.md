---
title: "Tiger Lake-H のブートログ、lscpu、lspci ……"
date: 2020-12-23T18:11:19+09:00
draft: false
tags: [ "Tiger_Lake", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

Intel は既にモバイル向けの *Tiger Lake_L (Model: 0x8c)* をリリースし、各社から採用製品が出されている。  
その上位プロセッサ、(バリアント的には) デスクトップ向けにあり、ハイエンドモバイル (H Series) にも採用される *Tiger Lake (Model: 0x8d)* を搭載した Dell PC の Linux Kernel のブートログ、lscpu、lspci の実行結果がアップロードされている。  

 * [[BUG] Dell TGL-H Machine failed to load sof-tgl.ri · Issue #3711 · thesofproject/sof](https://github.com/thesofproject/sof/issues/3711)

*Tiger Lake-H* の各ログ、実行結果がアップロードされたのは [thesofproject/sof: Sound Open Firmware](https://github.com/thesofproject/sof) レポジトリで、Sound Open Firmware は Intelプロセッサやそれとペアになる PCH に内蔵される音声処理用のDSPのサポートを提供するため、Intel がオープンソースで開発しているプロジェクト。バグ報告のため各ファイルがアップロードされた。  

 >        Architecture:                    x86_64
 >        CPU op-mode(s):                  32-bit, 64-bit
 >        Byte Order:                      Little Endian
 >        Address sizes:                   39 bits physical, 48 bits virtual
 >        CPU(s):                          16
 >        On-line CPU(s) list:             0-15
 >        Thread(s) per core:              2
 >        Core(s) per socket:              8
 >        Socket(s):                       1
 >        NUMA node(s):                    1
 >        Vendor ID:                       GenuineIntel
 >        CPU family:                      6
 >        Model:                           141
 >        Model name:                      Genuine Intel(R) CPU 0000 @ 2.60GHz
 >        Stepping:                        0
 >        CPU MHz:                         1602.049
 >        CPU max MHz:                     4800.0000
 >        CPU min MHz:                     800.0000
 >        BogoMIPS:                        6220.80
 >        Virtualization:                  VT-x
 >        L1d cache:                       384 KiB
 >        L1i cache:                       256 KiB
 >        L2 cache:                        10 MiB
 >        L3 cache:                        24 MiB
 >        NUMA node0 CPU(s):               0-15
 >        Vulnerability Itlb multihit:     Not affected
 >        Vulnerability L1tf:              Not affected
 >        Vulnerability Mds:               Not affected
 >        Vulnerability Meltdown:          Not affected
 >        Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
 >        Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
 >        Vulnerability Spectre v2:        Mitigation; Enhanced IBRS, IBPB conditional, RSB filling
 >        Vulnerability Srbds:             Not affected
 >        Vulnerability Tsx async abort:   Not affected
 >        Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb cat_l2 invpcid_single cdp_l2 ssbd ibrs ibpb stibp ibrs_enhanced tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdt_a avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb intel_pt avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves split_lock_detect dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp hwp_pkg_req avx512vbmi umip pku ospke avx512_vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg tme avx512_vpopcntdq rdpid movdiri movdir64b fsrm avx512_vp2intersect md_clear flush_l1d arch_capabilities
 >
 > {{< quote >}} [[BUG] Dell TGL-H Machine failed to load sof-tgl.ri · Issue #3711 · thesofproject/sof](https://github.com/thesofproject/sof/issues/3711) {{< /quote >}}

lscpu の実行結果を見ると、まず `Family: 0x6(6), Model: 0x8d(141)` と、既にリリースされているモバイル向けの *Tiger Lake_L* ではなく、*Tiger Lake (Model: 0x8d)* であることは確かだ。[^intel-fam]  
モデル名からも察せられるように、サンプル段階のシリコンであり、Stepping 0 となっている。  

[^intel-fam]: [linux/intel-family.h at master · torvalds/linux](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h)

規模は 8-Core/16-Thread、総 L1dキャッシュ 384KiB、L1iキャッシュ 256KiB、L2キャッシュ 10MiB、L3キャッシュ 24MiB。  
コア/スレッド数、キャッシュ容量共に、4-Core/8-Thread の *Tiger Lake_L* をそのまま倍にしたものとなっている。  

ただ、サポートする命令に違いがあり、[OpenBenchmarking.org](https://openbenchmarking.org/) にある適当な *Tiger Lake_L* 、**i7-1165G7** の lscpu 実行結果[^ob-i7-1165g7]と比較すると、  
*Tiger Lake-H* は `SMX` と `TME` をサポートしていることがわかる。  
`SMX` は Safer Mode Extension、`TME` は Total Memory Encryption の略で、どちらもセキュリティ機能に関連した命令。  
これらは *Tiger Lake_L* にも実装されており、組み込み向け (E Series) では有効化されている。  
*Tiger Lake_L* のエンジニアサンプリングの結果をまたも [OpenBenchmarking.org](https://openbenchmarking.org/) から引っ張ってくると、製品版の **i7-1165G7** では無効化されていた `SMX` と `TME` が有効化されているため[^ob-tgl-sample]、サンプル特有の違いなのかもしれない。  

[^ob-i7-1165g7]: [tigerlake - lscpu - OpenBenchmarking.org](https://openbenchmarking.org/system/2011033-FI-TIGERLAKE04/tigerlake/lscpu)
[^ob-tgl-sample]: [Intel 0000 - lscpu - OpenBenchmarking.org](https://openbenchmarking.org/system/2006079-HV-1LOG7048351/Intel%200000/lscpu)

iGPU については、lspci の実行結果に `DeviceID: 0x96a0` の記述があった。  
これは *TGL GT1* に割り当てられた DeviceID の 1つであり[^tgl-did]、今回の *Tiger Lake-H* は iGPU にそれを搭載し、DeviceID も共有していることとなる。  

 >        00:02.0 VGA compatible controller [0300]: Intel Corporation Device [8086:9a60] (prog-if 00 [VGA controller])
 >        	DeviceName: Onboard - Video
 >        	Subsystem: Dell Device [1028:2112]
 >
 > {{< quote >}} [[BUG] Dell TGL-H Machine failed to load sof-tgl.ri · Issue #3711 · thesofproject/sof](https://github.com/thesofproject/sof/issues/3711) {{< /quote >}}

[^tgl-did]: [include/pci_ids/iris_pci_ids.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/include/pci_ids/iris_pci_ids.h)

| Intel {{< xe class="lp" >}}/Gen12 | GT0.5 | GT1 | GT2 | GT2 (DG1) |
| :-- | :--: | :--: | :--: | :--: |
| Dual-Sub Slices | 1 | 2 | 6 | 6 |
| &ensp;EUs | 16 | 32 | 96 | 96 |
| &ensp;&ensp;SPs | 128 | 256 | 768 | 768 |
| GPU L3$ Banks | 4 | 4 | 8 | 8 |
| GPU L3$ Size | 1920KB? | 1920KB | 3072KB | 16384KB[^dg1-l3] |
| | RKL / ADL-S | TGL-U / TGL-H<br>/RKL / ADL-S | TGL /ADL-P | DG1/SG1 |

[^dg1-l3]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)

今月の初め、2020/12/04 には Dell がイーサネットコントローラーの電力管理に関するパッチを、リリース予定にある、デスクトップ向け *Tiger Lake* 搭載システムのため投稿している。[^tgl-desktop-eth]
今回の各ファイルをアップロードしたのも Dell に所属するエンジニアによるものだ。*Tiger Lake-H* 搭載製品の開発が、ある程度表に出して良いレベルまで進んでいると推測される。  
*Tiger Lake-H* のベールが取り払われる日はそう遠くないだろう。  


[^tgl-desktop-eth]: [[Intel-wired-lan] [PATCH v3 6/7] e1000e: Add Dell TGL desktop systems into S0ix heuristics](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20201130/022476.html)

{{< ref >}}

 * [Intel Releases New Technology Specification for Memory Encryption](https://software.intel.com/content/www/us/en/develop/blogs/intel-releases-new-technology-specification-for-memory-encryption.html)
 * [Dell Getting Linux Power Management Optimized For Their Latest Systems + Upcoming Tiger Lake Desktop - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Dell-S0ix-i219LM-Fixing)

{{< /ref >}}
