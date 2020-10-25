---
title: "気になってた Jasper Lake のキャッシュ構成  ―― L3キャッシュを搭載か"
date: 2020-10-24T22:47:40+09:00
draft: false
tags: [ "Jasper_Lake", "Tremont", "Gen11" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

以前 [Elkhart Lake](/tags/elkhart_lake) が発表された際、[Tremontアーキテクチャ](/tags/tremont) を採用するプロセッサの中では、まだ唯一キャッシュ構成が不明だった *Jasper Lake* だが、  
{{< link >}} [Intel Elkhart Lake 発表、その詳細 | Coelacanth's Dream](/posts/2020/09/24/intel-atom-ehl-tgl/) {{< /link >}}
Geekbench に *Jasper Lake* による実行結果が現れたことでキャッシュ構成が判明した。  
だがそれは *Tremontファミリー* の中では、少し異質なものであった。  

## Jasper Lake {#jsl}
まず、*Tremontアーキテクチャ* では 4コアとそれらで共有する L2キャッシュでクラスタ(タイルとも呼ばれる) を構成する。L2キャッシュ容量はプロセッサごとに異なり、1MB から 4.5MB の範囲で選択可能となっている。[^intel-arch-manual]  
マイクロサーバー向けの *Snow Ridge* は最大の 4.5MB、ハイブリッドプロセッサ *Lakefield* は 1.5MB、組み込み向けの *Elkhart Lake* も 1.5MB を搭載している。  
*Lakefield* は *Tremont* 4コアに加え、bigコアである *Sunny Cove* 1コアを搭載し、それと共有、L3キャッシュに位置する LLC (Last Level Cache) を 4MB 持つ。  

[^intel-arch-manual]: [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html) (Page 146)

### Jasper Lake は L3キャッシュを搭載か {#jsl-l3}

以下が、*Jasper Lake* である **Intel Celeron N4500** 、**Intel Celeron N5100** のGeekbench 5 実行結果となる。  

 * Celeron N4500 - <https://browser.geekbench.com/v5/cpu/4239654>
 * Celeron N5100 - <https://browser.geekbench.com/v5/cpu/4277736>

CPU の x86\_Model が 156(0x9C) となっており、これを Linux Kernel に含まれる Intel CPUファミリー一覧と照らし合わせると *Tremont_L / Jasper Lake* と一致する。[^intel-family]
ベンチマーク結果にある、モデル名 *Google dedede* というのは、以前紹介したように、*Jasper Lake* を搭載する Chromebook のメインボード(リファレンスボード) のコードネームであり、そこでも *Jasper Lake* だと分かる。マザーボードの *drawcia* はその派生ボードの名。  
{{< link >}} [Intel 10nm プロセッサを搭載した Chromebook ボード | Coelacanth's Dream](/posts/2020/05/13/intel-10nm-processor-chromebook/) {{< /link >}}

[^intel-family]: [linux/intel-family.h at master · torvalds/linux](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h)

そして、キャッシュ構成を見てみると、{{< comple >}} *Lakefield* のようなハイブリッドプロセッサでないのに {{< /comple >}} そこには `L3 Cache 4.00MB x 1` とある。  
上述したように、この結果はほぼ確実に *Jasper Lake* である。  

| Intel Tremont | Snow Ridge | Lakefield | Elkhart Lake | Jasper Lake |
| :-- | :--: | :--: | :--: | :--: |
| L2cache (per Tile) | 4.5 MB | 1.5 MB | 1.5 MB | 1.5MB |
| L3cache/LLC (per Tile) | N/A | 2MB | N/A | 4MB |

ここで疑問に思うのは、4コアで共有する L2キャッシュ容量は *Lakefield* 、*Elkhart Lake* と同じ 1.5MB なのに、L3キャッシュ 4MB を搭載していることだ。L2キャッシュを 1.5MB より多くには設定しなかった。`L2 Cache 1.50 MB x 1` とあることから、Tremont コアそれぞれが L2キャッシュを別に持ってはいない。  
そのことから、ただ CPU性能の向上を目的としている訳ではないと思われる。  

*Jasper Lake* で L3キャッシュを搭載した理由としては、1つに GPU性能の向上が考えられる。  
Intel プロセッサにおける L3キャッシュ/LLC には GPU からもアクセスが可能となっており、一部をキャッシュとして利用できる。  
*Jasper Lake* はモバイル向けプロセッサであり、CPU性能だけでなく、最低限の GPU性能が求められるシーンが少なくはないだろう。  
L2キャッシュを多く設定するのではなく、L3キャッシュを搭載することで CPU、GPU 両方の性能向上が見込まれる。  

*Jasper Lake* の GPU の仕様は *Elkhart Lake* とほとんど変わらず、EU数は最大 32基 (256SP) となる。  
*Jasper Lake* のターゲットとなる TDP(PL1)帯は 6W あたりからとされ、**AMD 3015e (Pollock) / 3020e (Dali)** が競合となるのではないかと思う。[^jsl-pl1]  
大きく異なるアーキテクチャとなる省電力プロセッサ同士の比較は結構気になる所だ。  
*Jasper Lake* は LPDDR4メモリ対応、Intel 10nmプロセスという点で有利に思われる。  

[^jsl-pl1]: <https://review.coreboot.org/c/coreboot/+/41668/6/src/mainboard/google/dedede/variants/baseboard/devicetree.cb>

また、L3キャッシュの存在以外に、ハイパースレッティングが無効化されていることが判明している。  
*Tremontファミリー* でハイパースレッティングが有効化されているモデルは無く、自分はこれまで、モバイル向けの *Jasper Lake* では有効化されるのだろうと考えていた。  
[Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html) でも触れられていなかったが、*Tremontアーキテクチャ* にハイパースレッティングは実装されていないのだろうか？  


