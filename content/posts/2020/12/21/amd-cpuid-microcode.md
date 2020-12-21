---
title: "CPUID とマイクロコードバージョンから AMD CPU の Family と Model を知りたい"
date: 2020-12-21T13:49:27+09:00
draft: false
tags: [ "Cezanne", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
toc: false
---


以前 Intel CPU の CPUID から `Family, Model, Stepping` を読み取る方法に触れたが、AMD CPU はまだ自分自身理解していない部分があったため、整理も兼ねて書く。  
{{< link >}} [Alder Lake-P を搭載する Chromebookボード 「Brya」、そして Alder Lake-M | Coelacanth's Dream](/posts/2020/12/02/adl-p-chromebook-board-adl-m/) {{< /link >}}
そして、過去の主張を訂正するための記事。  

## CPUID 

CPUID とひと括りにしているが、CPUID にも色々ある。そもそも CPUID は CPU の各種情報を出力する命令である。  
CPUID命令は EAXレジスタに特定の値をセットしてから実行することで、ある情報が取得、出力される。  
そうして得られる情報の中で、`Family, Model, Stepping` の識別情報が、ベンチマーク結果から得られる CPU情報と照らし合わせたりと活用しやすいから注目されやすい、収集されやすいという話。  

CPUID から `Family, Model, Stepping` を読み取る方法に関しては当然 AMD からドキュメントが出されているし[^amd-cpuid-doc]、Coreboot のコードにも分かりやすく記述されている。[^coreboot-cpuid]  
そのため CPUID から読み取ることは難しくない。  
例えば *Zen+ CPU / Pinacle Ridge* の CPUID は `0x00800f82` だが、その文字列を `Family, Model, Stepping` に対応させると以下の表のようになる。  

[^amd-cpuid-doc]: [CPUID Specification - AMD](https://developer.amd.com/resources/25481-2/)
[^coreboot-cpuid]: [coreboot/Cpuid.h at master · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/master/src/vendorcode/intel/edk2/edk2-stable202005/MdePkg/Include/Register/Amd/Cpuid.h)


| CPUID | (Reserved) | Ext.Family | Ext.Model | (Reserved) | Base.Family | Base.Model | Stepping |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| Pinaccle Ridge | 0x0 | 0x08 | 0x0 | 0x0 | 0xF | 0x8 | 0x2 |

それをさらによく見る形式に変換すると、`Family: 0x17 (23), Model: 0x8 (8), Stepping: 0x2` のようになる。  

## Microcode

CPUID については、あまり見掛ける機会は無いが、マイクロコードはそれなりにある。  
というのもベンチマーキングプラットフォームである [Phoronix Test Suite](https://www.phoronix-test-suite.com/) は、マイクロコードのバージョンを取得し、CPU情報の 1つとして記録するからだ。  
今回調べるきっかけとなったのも、[Cezanne APU](/tags/cezanne) の Model について、Coreboot には `Model: 0x51 (81)` であるように記述されているが、Geekbench の結果等では `Model: 0x50 (80)` となっていたことだ。この 1 違いにもっとまともな答えを見出したかった。[^czn-model]  

[^czn-model]: [Coreboot に Renoir と Lucienne/Cezanne らしき DeviceID が追加される | Coelacanth's Dream](/posts/2020/11/19/rn-lcn-czn-coreboot-did/) [Cezanne APU、Ryzen 7 5800U が Geekbench に現る | Coelacanth's Dream](/posts/2020/11/26/r7-5800u-czn/)

以下はとりあえず [OpenBenchmarking.org](https://openbenchmarking.org/) から集めた Zen系 CPU のマイクロコードバージョンだが、`Family: 0x17(23)` である *Zen+/2* CPU は先頭が同じ `0x8` になっていることから、マイクロコードバージョンは `Family, Model, Stepping` に対応していると推測できる。  

| Microcode Patch Level | |
| :-- | :--: |
| Zen+ Ryzen | 0x800820d |
| Zen+ APU[^3400g] | 0x8108109 |
| Zen2 EPYC[^77f2] | 0x830101c |
| Zen2 Ryzen[^3800xt] | 0x8701021 |
| Zen3 APU[^czn-sample] | 0xa500009 |

[^3400g]: [Opus 3400g Benchmarks - OpenBenchmarking.org](https://openbenchmarking.org/result/2012204-HA-OPUS3400G69)
[^77f2]: [Extra Tests Performance - OpenBenchmarking.org](https://openbenchmarking.org/result/2008249-NE-EXTRATEST01)
[^3800xt]: [3800xt Monday Benchmarks - OpenBenchmarking.org](https://openbenchmarking.org/result/2012146-HA-3800XTMON29)
[^czn-sample]: [AMD Cezanne APU ベンチマーク結果　―― 8-Core/16-Thread, 3.6GHz、構成は Zen 3 + Vega か | Coelacanth's Dream](/posts/2020/08/31/amd-cezanne-benchmark/)

そして、それぞれを比較、CPUID の各値と照らし合わせた結果、以下のように対応していると考えられる。  
Base と Ext を合わせることで、Family と Model が求められる。  

| Microcode | Ext.Family | Ext.Model | (Reserved?) | Base.Model | Stepping | Version? |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: |
| Zen+ Ryzen | 0x8 | 0x0 | 0x0 | 0x8 | 0x2 | 0x0d |
| Zen+ APU | 0x8 | 0x1 | 0x0 | 0x8 | 0x1 | 0x09 |
| Zen2 EPYC | 0x8 | 0x3 | 0x0 | 0x1 | 0x1 | 0x1c |
| Zen2 Ryzen | 0x8 | 0x7 | 0x0 | 0x1 | 0x0 | 0x21 |
| Zen3 APU | 0xA | 0x5 | 0x0 | 0x0 | 0x0 | 0x09 |

ここでも *Cezanne* は `Model: 0x50 (80)` となった。  
訂正といのはこのことで、*Cezanne* は `Model: 0x51 (81)` ではなく、`Model: 0x50 (80)` 可能性のが高く、Coreboot のは誤字かもしれない。{{< comple >}} あるいは *Cezanne* とは別の APU に向けたものである可能性もあるが、それは GPU部に *Cezanne* と同じ DeviceIDを用い 、Model は 1 違いというややこしいものになる。{{< /comple >}} [^czn-vbios]  

`Stepping 0` であるから `Model: 0x51 (81)` となっていないのではないか、とも書いたが、*Zen 2 EPYC (Rome)* も *Zen 2 Ryzen (Matisse)* も `Stepping 0` であるため根拠にはなり得ず、思い切り外した推測であった。  
マイクロコードバージョンの読み方を知っていたら、少しはまともな見方が出来ただろう、という後悔。  

[^czn-vbios]: [Cezanne の Coreboot 対応が進行中　―― MI200, Mero, Rembrandt のネームプレート | Coelacanth's Dream](/posts/2020/12/16/czn-coreboot-mi200-mero-rmb/)
