---
title: "PCIeカード型の Radeon Pro W5700X 32GB?"
date: 2020-09-15T09:36:45+09:00
draft: false
tags: [ "Navi10", "gfx1010" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

ベンチマークプラットフォーム [OpenBenchmarking.org](https://openbenchmarking.org/) に以下のような、*Navi10* をベースとしたVRAM 32GBを搭載するカードを組み込んだシステムの結果が見つかった。  

 > Processor: AMD A10 PRO-7800B R7 12 Compute Cores 4C+8G @ 3.50GHz (2 / 4 Threads),  
   Motherboard: LENOVO Bantry CRB (M0LKT12AUS BIOS),  
   Chipset: AMD 15h, Memory: 2 x 4096 MB DDR3-1600MT/s,  
   Disk: 1000GB Seagate ST1000LM048-2E71,  
   **Graphics: AMD Navi 10 32GB (300/875MHz),**  
   Audio: Realtek ALC662 rev3,  
   Monitor: LG HDR 4K,  
   Network: Realtek RTL8111/8168/8411
 >
 > {{< quote >}} [Unigine-tropics730 Benchmarks - OpenBenchmarking.org](https://openbenchmarking.org/result/2007314-NI-UNIGINETR01) {{< /quote >}}

Intel Core i7 950 でテストした結果も見つかっている。[^ob-w5700x-32gb]  

[^ob-w5700x-32gb]: [Gputest_s2 Benchmarks - OpenBenchmarking.org](https://openbenchmarking.org/result/2005246-NI-GPUTESTS233)

このカードの正体は分かっており、現地時間 2019/12/10 付けで Apple Mac向けのモデルとして発表された **Radeon Pro W5700X** と判明している。  
Phoronix Test Suite には、ベンチマークを実行するシステムの情報を収集する機能があり、そのシステムの環境によっては Linux Kernel が出力するログも収集されている。Kconfig の `SECURITY_DMESG_RESTRICT` は有効にしておこう。  
そして、ログの中に AMD GPU の DeviceID、RevisionID も含まれていたため、そこで **Radeon Pro W5700X (0x7310:0x00)** と判断できる。  

VRAM 32GB という巨大なメモリサイズに目が行くが、現時点では Mac Pro の CTO で 32GBモデルを選ぶことは出来ず、16GBのみであるものの、**Radeon Pro W5700X** のプレスリリースにて最大 32GBにも対応するとあるため[^radeon-pro-w5700x]、その点では特別おかしいカードという訳ではない。  

[^radeon-pro-w5700x]: [AMD Radeon™ Professional GPUs Offer Breakthrough Graphics Performance for Apple Mac Pro | AMD](https://www.amd.com/en/press-releases/2019-12-10-amd-radeon-professional-gpus-offer-breakthrough-graphics-performance-for)

気になるのは、ベンチマークを実行したシステムが Apple Mac ではなく、ふつうのシステムであるという点だ。  
上述したように、**Radeon Pro W5700X** は Apple Mac向けのモデルとして発表され、多くのボードが対応する PCIeカード型ではなく、Apple Mac Pro (2019) のロジックボードが対応する MPXモジュール型のみがリリースされている。  
今回のふつうのシステムで実行された結果は、PCIeカード型の **Radeon Pro W5700X 32GB** が存在し、今後正式にリリースされる予定にある *かも* しれないことを示す。PCIeカード型が存在しても、市場に出ないことは十分あり得る。  

ちなみに *Navi10* のメモリバス幅 256-bit で VRAM 32GB を実現するとなると、GDDR6 16Gb(2GB) チップを 16枚それぞれ 16-bit 接続 (`16-bit * 16 * 16Gb(2GB)`) という構成になる。  
**Radeon Pro W5700X (16GB) MPX Module** は $1,000 となっており[^w5700x-mpx]、32GBモデルが出たらそれが *Navi10* ベースで最も高価なカードとなるのだろうか？  

[^w5700x-mpx]: [Radeon Pro W5700X MPX Module - Apple](https://www.apple.com/shop/product/MW662AM/A/radeon-pro-w5700x-mpx-module)
