---
title: "AMD Barcelo APU ブートログ　―― Ryzen 7 5825U, 100-000000586-40_Y"
date: 2021-12-16T21:16:03+09:00
draft: false
tags: [ "Barcelo", "Cezanne" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

先日取り上げた [Yellow Carp/Rembrandt](/tags/yellow_carp/) のブートログ同様、[Ubuntu in Launchpad](https://launchpad.net/ubuntu) における You-Sheng Yang 氏によるバグ報告の中で *Barcelo APU* の Linux Kernel ブートログが投稿されている。  
{{< link >}} [AMD Yellow Carp/Rembrandt APU ブートログ | Coelacanth's Dream](/posts/2021/12/14/yc-rmb-bootlog/) {{< /link >}}

 * [Bug #1954633 “s2idle suspend failure: amd_pmc AMDI0005:00: SMU r...” : Bugs : linux package : Ubuntu](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1954633)
 * [Bug #1954930 “AMD: Suspend not working when some cores are disab...” : Bugs : linux package : Ubuntu](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1954930)

**AMD Ryzen 7 5825U with Radeon Graphics** 16-Thread、**AMD Eng Sample: 100-000000586-40_Y** 8-Thread の 2種類が確認できる。  
**Ryzen 7 5825U** は、ボードに *Dell Inc. Vostro 3525*、ベースクロックは 2.0 GHz (1999.583 MHz) で動作している。  
**AMD Eng Sample: 100-000000586-40_Y** は、*AMD Celadon-CZN/Celadon-CZN*、ベースクロック 2.6 GHz (2595.124 MHz)。  

どちらも GPU部の DeviceID が `0x15E7` であり、これは *Barcelo* に割り当てられた DeviceID。  
{{< link >}} [新たな Zen 3 + Vega APU 「Barcelo」 | Coelacanth's Dream](/posts/2021/07/17/amd-barcelo-vega-apu/) {{< /link >}}

 > 		åäºŒ 14 21:00:15 WMVB5-DVT2-A2 kernel: [drm] initializing kernel modesetting (RENOIR 0x1002:0x15E7 0x1028:0x0B7C 0xC1).
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/574000802/suspend-success.log) {{< /quote >}}
 >
 > 		[    1.340700] [drm] initializing kernel modesetting (RENOIR 0x1002:0x15E7 0x1002:0x0123 0xC3).
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/574190473/CurrentDmesg.txt) {{< /quote >}}

そのため、確かに *Barcelo APU* だと言えるのだが、CPU の `Family, Model, Stepping` は *Green Sardine/Cezanne APU* と同一となっている。  
また、当てられているマイクロコードのバージョン (`0xa50000c`) も同一のものだった。  

 > 		åäºŒ 14 21:00:15 WMVB5-DVT2-A2 kernel: smpboot: CPU0: AMD Ryzen 7 5825U with Radeon Graphics (family: 0x19, model: 0x50, stepping: 0x0)
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/574000802/suspend-success.log) {{< /quote >}}
 >
 > 		[    0.209819] smpboot: CPU0: AMD Eng Sample: 100-000000586-40_Y (family: 0x19, model: 0x50, stepping: 0x0)
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/574190473/CurrentDmesg.txt) {{< /quote >}}

Coreboot へのパッチから、*Barcelo* が *Green Sardine/Cezanne* と同じ構成、*Raven* に対する *Picasso* 、*Renoir* に対する *Lucienne* のような存在であることは窺い知れていたが、*Picasso* には製造プロセス、*Lucienne* は電力管理機能に改良が含まれており、`Model` は *Raven*、*Renoir* と異なっていた。  
CPU の識別に使われる `Family, Model, Stepping` が変わらないあたり、*Barcelo* には *Green Sardine/Cezanne* から加えられた改良が特に無いということなのだろうか。  
ダイが異なりながら、一部が *Picasso* と同じ `Model` であり、DeviceID も他と被っている (RevisionID は異なる) *Raven2 APU* という前例がいるため、これだけの情報では 100% そうだとは言えないが、そうした可能性は考えられる。  
{{< link >}} [いかに Zen APU は複雑か | Coelacanth's Dream](/posts/2020/02/16/raven-family-complex/) {{< /link >}}
**AMD Eng Sample: 100-000000586-40_Y** はボードに *Celadon-CZN* が使われており、ここでも *Cezanne* と同一となっている。  
*Celadon* はプラットフォーム、開発用のリファレンスボードに対するコードネームであり、モバイル向けリファレンスボード (FP6パッケージ) には *Celadon* か *Majolica*、デスクトップ向け AM4 APU には *Artic* が使われている。  
モバイル向けは Proモデルに *Celadon* 、通常モデルには *Majolica* として使い分けていると思われる。[^majolica]  

[^majolica]: [Lucienne/Cezanne APU を搭載する Chromebookボード　―― Guybrush、Mancomb | Coelacanth's Dream](/posts/2020/11/22/lcn-czn-fp6-chromebook-board/)

*Barcelo APU* は他に、ボードに *Majolica-BRC*、CPU は 8-Core/16-Thread、ベース 1.9GHz で動作する **AMD Eng Sample 100-000000580-40_Y** も確認されている。  

 * [Func Benchmarks [2112063-TJ-FUNC8359488] - OpenBenchmarking.org](https://openbenchmarking.org/result/2112063-TJ-FUNC8359488)

余談だが、*Renoir /Lucienne /Green Sardine (Cezanne) /Barcelo APU* では、AMDGPUドライバーが間違った有効 CU数を検出する。上記ブートログでも、CU 26基か 28基という数が出てきている。  
これは AMDGPUドライバーがそれらの VBIOS、ファームウェアのパースに対応していなかったことによるもので、issue を送ったところ AMD の Alex Deucher 氏より、すぐに修正パッチが投稿された。  

 * [The number of CU on GreenSardine/Cezanne is wrong (#1833) · Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues/1833)
