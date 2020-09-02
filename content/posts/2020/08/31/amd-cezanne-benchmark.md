---
title: "AMD Cezanne APU ベンチマーク結果　―― 8-Core/16-Thread, 3.6GHz、構成は Zen 3 + Vega か"
date: 2020-08-31T23:38:04+09:00
draft: false
tags: [ "Cezanne", "Renoir", "Zen_3" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

オープンソースで開発される高度なベンチマークプラットフォーム [OpenBenchmarking.org](https://openbenchmarking.org/) から、AMD の次世代 APU と目されるコードネーム *Cezanne* のベンチマーク結果が見つかった。  

 * [Fdfdffdfd Benchmarks [2008302-NE-FDFDFFDFD18] - OpenBenchmarking.org](https://openbenchmarking.org/result/2008302-NE-FDFDFFDFD18)

 > AMD RENOIR: 
 >
 >    Processor: AMD Eng Sample 100-000000263-30_Y @ 3.60GHz (8 Cores / 16 Threads),  
 >    Motherboard: AMD Artic-CZN (WA20819N BIOS),  
 >    Chipset: AMD Renoir Root Complex,  
 >    Memory: 8GB,  
 >    Disk: 500GB Western Digital WD5000LPVX-7,
 >    Graphics: AMD RENOIR 512MB (1800MHz),
 >    Audio: AMD Device 1637,
 >    Monitor: LG HDR 4K,
 >    Network: Realtek RTL8111/8168/8411
 >
 > {{< quote type="text" >}} [Fdfdffdfd Benchmarks [2008302-NE-FDFDFFDFD18] - OpenBenchmarking.org](https://openbenchmarking.org/result/2008302-NE-FDFDFFDFD18) {{< /quote>}}

<!--

 > Processor Notes: Scaling Governor: acpi-cpufreq ondemand - CPU Microcode: 0xa500009  
 > Security Notes: itlb_multihit: Not affected + l1tf: Not affected + mds: Not affected + meltdown: Not affected + spec_store_bypass: Mitigation of SSB disabled via prctl and seccomp + spectre_v1: Mitigation of usercopy/swapgs barriers and __user pointer sanitization + spectre_v2: Mitigation of Full AMD retpoline IBPB: conditional IBRS_FW STIBP: always-on RSB filling + srbds: Not affected + tsx_async_abort: Not affected  
 >
 > 引用元:  [Fdfdffdfd Benchmarks [2008302-NE-FDFDFFDFD18] - OpenBenchmarking.org](https://openbenchmarking.org/result/2008302-NE-FDFDFFDFD18)

-->

CPU名は `AMD Eng Sample 100-000000263-30_Y` 、マザーボードは `Artic-CZN` 、CPUマイクロコードのバージョンは `0xa500009`。  
CPUクロックは 3.6GHz とされ、8-Core/16-Thread となっている。  
マザーボード名の `Artic` は *Renoir* にも使われていたもので、何らかのプラットフォーム、モバイル向けやデスクトップ向けかを指し示すものと思われるが、(自分の中で)はっきりとした答えは出てない。`CZN` は *Cezanne* の略だろう。  
マイクロコードの表記は、Zen/+/2 系に見られる `0x8` で始まるものと異なるため、それよりも新しい世代の CPU、*Zen 3 アーキテクチャ* の可能性が高い。  

GPU部は *Renoir* と同一とされ、GPUクロックは 1800MHz。  
Chipset と Audio の項目を見るに、GPU 内部の AudioCoProcessor や SoC としての I/O部も *Renoir* と共通ではないかと思われる。  

AMD の次世代 APU には他に [Lucienne](/tags/lucienne) が確認されており、ソフトウェア開発者のコメントから[^lucienne-renoir]、*Renoir* 同様に **Zen 2 + Vega(gfx909)** の構成を取る APU と考えている。  
対し、*Cezanne* は今回確認できた情報から、**Zen 3 + Vega(gfx909?)** の構成であると考えられる。  
ただ、これらがどのタイミングで登場するか、またターゲット帯はそれぞれどうなるか等の謎はある。  
*Renoir* はモバイル(Uシリーズ)、ハイエンドモバイル(Hシリーズ)、デスクトップ向け(Gシリーズ) を単一のダイでカバーしているが、  
*Renoir* の次世代であり、*Renoir* と似た構成を取る *Lucienne* 、 *Cezanne* の両者がどのように棲み分けるかは想像が難しい。  

[^lucienne-renoir]: <https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2246284/5/util/amdfwtool/amdfwtool.c#1087>

| APU | CPU | GPU |
| :-- | :--: | :--: |
| Raven | Zen | Vega<br>(gfx902) |
| Picasso | Zen+ | Vega<br>(gfx902) |
| Raven2<br>(Dali /Pollock) | Zen | Vega<br>(gfx909) |
| Renoir | Zen 2 | Vega<br>(gfx909) |
| Lucienne | Zen 2 | Vega<br>(gfx909) |
| Cezanne | Zen 3? | Vega<br>(gfx909)? |
