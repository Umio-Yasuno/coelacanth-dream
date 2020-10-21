---
title: "またも OpenBenchmarking.org にて AMD Cezanne APU が見つかる　―― 4-Core/8-Thread, 3.5GHz"
date: 2020-10-21T19:05:47+09:00
draft: false
tags: [ "Cezanne" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

オープンソースで開発される高度なベンチマークプラットフォーム [OpenBenchmarking.org](https://openbenchmarking.org) から、またも AMD の次期 APU *Cezanne* の結果が見つかった。  

 * [FishyCat Benchmarks [2010216-FI-FISHYCAT424] - OpenBenchmarking.org](https://openbenchmarking.org/result/2010216-FI-FISHYCAT424)

 >   Processor: AMD Eng Sample 100-000000262-30_Y @ 3.50GHz (4 Cores / 8 Threads),  
 >   Motherboard: AMD Artic-CZN (TA21100B BIOS),  
 >   Chipset: AMD Renoir Root Complex,  
 >   Memory: 1 x 16384 MB DDR4-2400MT/s 18ASF2G72AZ-2G3B1,  
 >   Disk: 120GB SanDisk SSD PLUS,  
 >   Graphics: llvmpipe,  
 >   Audio: NVIDIA Device 10f8,  
 >   Network: Realtek RTL8111/8168/8411  
 >
 > {{< quote >}} [FishyCat Benchmarks [2010216-FI-FISHYCAT424] - OpenBenchmarking.org](https://openbenchmarking.org/result/2010216-FI-FISHYCAT424) {{< /quote >}}

[前回](/posts/2020/08/31/amd-cezanne-benchmark/) 見つかった *Cezanne* は 8-Core/16-Thread, 3.6GHz というものだったが、今回は 4-Core/8-Thread, 3.5GHz となっている。  
CPUマイクロコードは `0xa50000b` 。前回は `0xa500009` だった。CPUマイクロコードのパッチレベル自体は前回よりも進んでいるものと考えられる。  
マザーボード名の `Artic` は、デスクトップ向け、またはデスクトップのプロ向けプラットフォーム (PRO565) を指すと思われる。[^artic-pro565]  
前回は GPU、Audio に {{< comple >}} Renoir APU と同じとされる {{< /comple >}} 内部GPU が認識されていたが、今回のシステムは NVIDIA TU104 をベースとする dGPU を搭載しているらしく、そちらが優先して認識されている。  

[^artic-pro565]: [What's PROMONTORY 1.05V current requirement... | Community](https://community.amd.com/thread/243920)

ベンチマークには Blender が実行されているが、*1 x 16384 MB* とのことから、メモリはシングルチャネルと考えられるため、スコアは特に気にしない。  
ただ DDR4メモリということと、`Artic` という名から AM4 プラットフォームで検証されていると思われる。   

気になるのは CPUクロックであり、前回の *Cezanne (ES) 8-Core/16-Thread* (3.6GHz) は、デスクトップ向け *Renoir* と比較すると同コア同スレッド数の **Ryzen 7 4700G, PRO 4750G** (3.6GHz) と同じクロックだが[^rn-4700g-4750g]、  
今回の *Cezanne (ES) 4-Core/8-Thread* (3.5GHz) は **Ryzen 3 4300G, PRO 4350G** のクロック (3.8GHz) とは一致せず、デスクトップ向けでも TDP 35W の SKU **Ryzen 3 4300GE, PRO 4350GE** (3.5GHz) と一致する。[^rn-4300ge-4350ge]  
このことから、今回の *Cezanne (ES) 4-Core/8-Thread* は TDP 35W の SKU であることが考えられる。  

[^rn-4700g-4750g]: [AMD Ryzen™ 7 4700G | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-4700g#product-specs) <br> [AMD Ryzen™ 7 PRO 4750G | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-pro-4750g#product-specs)
[^rn-4300ge-4350ge]: [AMD Ryzen™ 3 4300GE | AMD](https://www.amd.com/en/products/apu/amd-ryzen-3-4300ge#product-specs) <br> [AMD Ryzen™ 3 PRO 4350GE | AMD](https://www.amd.com/en/products/apu/amd-ryzen-3-pro-4350ge#product-specs)


