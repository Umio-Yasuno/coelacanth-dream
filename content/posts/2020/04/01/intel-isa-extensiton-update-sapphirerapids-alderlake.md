---
title: "Intel、拡張命令リファレンスをアップデート (Sapphire Rapids /Alder Lake /ハイブリッドプロセッサ)"
date: 2020-04-01T22:28:18+09:00
draft: false
tags: [ "Sapphire\ Rapids", "Tiger\ Lake", "Alder Lake" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

[Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/sites/default/files/managed/c5/15/architecture-instruction-set-extensions-programming-reference.pdf)のアップデートが行なわれた。リビジョンは *-O38* となる。  

## トピック

 * [AVX512_BF16命令にSapphire Rapidsが対応](#BF16-SPR)
 * [Alder Lake現る](#alderlake)
 * [非対称ハイブリッドプロセッサへの対応が進められる](#hybrid-processor)

### AVX512_BF16命令にSapphire Rapidsが対応 {#BF16-SPR}
中々姿を見せぬIntelの次世代サーバ向けプロセッサ、*Cooper Lake-SP* と *Ice Lake-SP* の、またさらに次世代となる *Sapphire Rapids* が AVX512_BF16命令 に対応することが明らかになった。[^1] *Sapphire Rapids* は2021年に登場が予定されている。[^2]  
AVX512_BF16には、他に *Cooper Lake* でしか対応していないため、*Sapphire Rapids* が *Cooper Lake* の後継という色が濃くなった形だ。  

  
また、*Tiger Lake* で追加された AVX512_VP2INTERSECT命令 にも対応するとしている。  
*Ice Lake* レベルまでの[^3]その他 AVX512命令に対応するかはわからないが、最近のIntel CPUアーキテクチャ{{< comple >}}Ice Lake、Cooper Lake、Tiger Lake{{< /comple >}}でAVX512命令の対応範囲が異なり、ややこしい事態となっているため、一度 *Sapphire Rapids* で全てに対応してもいいように思う。  

[^1]: <https://software.intel.com/sites/default/files/managed/c5/15/architecture-instruction-set-extensions-programming-reference.pdf#page=16>
[^2]: [ASCII.jp：Ice Lakeは6月から出荷開始 インテル CPUロードマップ (7/7)](https://ascii.jp/elem/000/001/859/1859973/7/#eid1859987)
[^3]: <https://github.com/llvm/llvm-project/blob/master/clang/lib/Basic/Targets/X86.cpp#L166>

### Alder Lake現る {#alderlake}
登場時期は不明ではあるが *Tiger Lake* より後の世代と予想されている *Alder Lake* の名が記載された。[^1]  
しかし、*Tiger Lake* の対応する命令と一致しないため、後継ではなくまた別の用途向けのプロセッサである可能性も十分考えられる。  

### 非対称ハイブリッドプロセッサへの対応が進められる {#hybrid-processor}
*Lakefield* [^4]のような Atom系コアと Core系コアの両方を搭載した非対称ハイブリッドプロセッサの情報を示すEAXレジスタが追加された。[^5]  
コアの種類は現状 Atom系と Core系の2種あるが、新たなEAXレジスタにはそれに加えもう2種分が予約されている。  
これは、Intelが将来的に新たなアーキテクチャを開発、実装するとも考えられるが、あくまで予約されているだけ、ということを強調しておきたい。  

また、これによりスケジューリングといったソフトウェア側の対応が進められるものと考えられる。  

[^4]: [Lakefield: Hybrid CPU with Foveros Technology | Intel Newsroom](https://newsroom.intel.com/press-kits/lakefield/?wapkw=lakefield)
[^5]: <https://software.intel.com/sites/default/files/managed/c5/15/architecture-instruction-set-extensions-programming-reference.pdf#page=32>

<hr>
<span class="reference">参考</span>

 * [Intel GCC Patches + PRM Update Adds SERIALIZE Instruction, Confirm Atom+Core Hybrid CPUs - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Intel-PRM-Hybrid-Sapphire)
 * [Intel Updates ISA Manual: New Instructions for Alder Lake, also BF16 for Sapphire Rapids](https://www.anandtech.com/show/15686/intel-updates-isa-manual-new-instructions-for-alder-lake-also-bf16-for-sapphire-rapids)
