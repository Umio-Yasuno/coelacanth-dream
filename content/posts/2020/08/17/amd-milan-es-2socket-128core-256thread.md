---
title: "2ソケット 128コア/256スレッドの AMD Milan ES ?"
date: 2020-08-17T21:15:27+09:00
draft: false
tags: [ "Zen_3", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

[tp-qemu](https://github.com/autotest/tp-qemu) へのプルリクエストの中に、AMD の次世代 EPYC *Milan* エンジニアサンプリングの CPU 情報らしきものがあった。[tp-qemu](https://github.com/autotest/tp-qemu) はプロセッサエミュレーターである [QEMU](https://www.qemu.org/) のテストツール。  

 >       Architecture:        x86_64
 >       CPU op-mode(s):      32-bit, 64-bit
 >       Byte Order:          Little Endian
 >       CPU(s):              256
 >       On-line CPU(s) list: 0-255
 >       Thread(s) per core:  2
 >       Core(s) per socket:  64
 >       Socket(s):           2
 >       NUMA node(s):        2
 >       Vendor ID:           AuthenticAMD
 >       CPU family:          25
 >       Model:               0
 >       Model name:          AMD Eng Sample: 100-000000114-07_22/15_N
 >       Stepping:            0
 >
 > 引用元: <https://github.com/autotest/tp-qemu/pull/2337#issuecomment-671139596>

`CPU Family` は `25(0x19)` となっており、*Zen 3* に予約されている値と一致する。[^zen_3-0x19]  
CPUの仕様は 2ソケットで、ソケットあたり 64コア、コアあたりのスレッド数は 2スレッド、計 128コア/256スレッドとなっている。  

[^zen_3-0x19]: [linux/k10temp.rst at 47acac8cae28b36668bf89400c56b7fdebca3e75 · torvalds/linux](https://github.com/torvalds/linux/blob/47acac8cae28b36668bf89400c56b7fdebca3e75/Documentation/hwmon/k10temp.rst)

`Model name` の *AMD Eng Sample: 100-000000114-07_22/15_N* は 1ヶ月近く前に話題になったものと同じであり、それも `QEMU` 関連であったから同じ筋の情報元なのだろうか。  

依然としてキャッシュ構成等は不明で、語れることはないのだが、個人的に `CPU Family: 25(0x19)` が見られたことが嬉しかった。  
