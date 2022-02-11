---
title: "Linux Kernel に Intel HFI をサポートするパッチ"
date: 2022-01-02T22:40:32+09:00
draft: false
tags: [ "Lakefield", "Alder_Lake", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Software", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Linux Kernel において Intel HFI (Hardware Feedback Interface) をサポートするパッチが、Intel の Linuxソフトウェアエンジニア [Ricardo Neri](https://www.linkedin.com/in/ricardo-neri-3347265) 氏によって投稿されている。  

 * [[PATCH v2 0/7] Thermal: Introduce the Hardware Feedback Interface for thermal and performance management](https://lore.kernel.org/linux-pm/20211220151438.1196-1-ricardo.neri-calderon@linux.intel.com/T/)

{{< pindex >}}
 * [Intel HFI](#hfi)
 * [Intel "E"HFI](#ehfi)
{{< /pindex >}}

## Intel HFI {#hfi}
Intel HFI は、ハードウェア (CPU) が OS に対して、ワークロードに応じた最適なスケジューリングを実行するためのガイダンス、ヒントを示す機能。  
パッチと Intel のドキュメントにはどの CPU でサポートされているかは記されていないが、CPUID (Level: 0x6:0x0, Bit19) を見るに、ハイブリッドアーキテクチャを採用する *Lakefield (Sunny Cove + Tremont)* [^lkf-cpuid] と *Alder Lake (Golden Cove + Gracemont)* [^adl-cpuid] でサポートされている。  

HFI ではスケジューラに対するヒントとして、相対的なレベルを示す値 (8-bit, 0-255) が性能と電力効率のそれぞれに用意され、加えて 6-Byte の予約部を持つ構造体を提供する。その構造体は論理プロセッサ数 (スレッド数) 分が用意される。  
性能と電力効率のレベルは高いほど優れている。レベルは設定された CPU温度の上限に達した時や、温度と消費電力に関する設定が変更された時などに更新される。  
0 の場合は性能、電力効率が著しく低下していることを示し、ドキュメントではその CPU にタスクをスケジューリングしないことを推奨している。  
パッチの実装では、0 の場合、ユーザースペースはその論理プロセッサをオフラインかアイドル状態に移行させられるようになっている。  
また、性能のレベルが低下傾向にあると気付いた場合には、0 になることを避けるため、電力制限を事前に調整することもあるとしている。  

[^lkf-cpuid]: [InstLatx64/GenuineIntel00806A1_Lakefield_CPUID.txt at 382580dc9f5fbed23cb1945c6b4a259a49cab484 · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/382580dc9f5fbed23cb1945c6b4a259a49cab484/GenuineIntel/GenuineIntel00806A1_Lakefield_CPUID.txt)
[^adl-cpuid]: [InstLatx64/GenuineIntel0090672_AlderLake_CPUID01.txt at 229ebf299f6cda5f85e1934a391e34cbc80da00d · InstLatx64/InstLatx64](https://github.com/InstLatx64/InstLatx64/blob/229ebf299f6cda5f85e1934a391e34cbc80da00d/GenuineIntel/GenuineIntel0090672_AlderLake_CPUID01.txt)

## Intel \"E\"HFI {#ehfi}
*Lakefield* ではサポートせず、*Alder Lake* ではサポートしている機能として、Intel HFI より進んだ機能となる Intel EHFI (Enhanced Hardware Feedback Interface) がある。  

EHFI ではソフトウェアスレッドに固有のインデックス (ClassID) が割り当てられ、スケジューラはこれに基づいて、ソフトウェアスレッドをどの CPUコアに割り当てるかを決定できる。  
論理プロセッサには EHFI 関連の履歴が蓄積されるようになっており、これは ClassID の割り当て等に用いているのではないかと思われる。履歴をリセットする命令として `HRESET` 命令も *Alder Lake* ではサポートされている。  

マーケティング的には *Intel Thread Director* と呼ばれ、Windows 11 でサポートされているそれは Intel EHFI を活用したものとなっている。  
今の所 Linux Kernel には HFI へのインターフェイスを実装するパッチが投稿、公開されている段階であり、*Lakefield* も含めた Intel ハイブリッドアーキテクチャへの最適化は進んでいると言えるが、*Intel Thread Director* に相当する機能はまだ実装されていない。  
Linux におけるハードウェアの検証や OSS関連のニュースを発信している [Phoronix](https://www.phoronix.com/scan.php?page=home) は、Windows 11 と各種 Linuxディストリで多種多様のベンチマークを実行し、その結果を掲載している。[^phoronix-adl]  
一部のアプリケーションにおいて Windows 11 の方が優れた性能を示しており、Linuxディストリでは *Golden Cove (Performance/Big), Gracemont (Efficient/Small)* へのコア割り当てが適切に行えていないことが窺える。  

[^phoronix-adl]: [Windows 11 Better Than Linux Right Now For Intel Alder Lake Performance - Phoronix](https://www.phoronix.com/scan.php?page=article&item=alderlake-windows-linux&num=1)


{{< ref >}}
 * [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
 * [](https://hc33.hotchips.org/)
* [Windows 11 Better Than Linux Right Now For Intel Alder Lake Performance - Phoronix](https://www.phoronix.com/scan.php?page=article&item=alderlake-windows-linux&num=1)
{{< /ref >}}
