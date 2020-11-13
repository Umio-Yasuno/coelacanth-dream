---
title: "Ryzen 7 5700U は Zen 2 + Vega、Lucienne か　―― Cezanne との棲み分けは"
date: 2020-11-12T22:57:47+09:00
draft: false
tags: [ "Lucienne", "Cezanne" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

AMD の次世代モバイル向けプロセッサの 1つ、**Ryzen 7 5700U** の Geekbench 実行結果が現れた。  
このことを発見、共有したのは Twitter で活動する [APISAK (@TUM_APISAK)](https://twitter.com/TUM_APISAK) 氏である。  

 * [HP HP Pavilion Laptop 15-eh1xxx - Geekbench Browser](https://browser.geekbench.com/v5/compute/1804236)

それと、まだ確証は出てきていないが、この記事では **Ryzen 7 5700U** のベースとなる APU を便宜的に [Lucienne APU](/tags/lucienne) とする。  
*Lucienne* は *Renoir* と同じ **Zen 2 + Vega** の構成を取る。  

{{< pindex >}}

 * [Zen 2アーキテクチャ、x86 Model は 68h (104)](#fam17h-model68h)
 * [Lucienne と Cezanne の棲み分けを考える](#lcn-czn)
   * [プロセス](#process)
   * [Zen 2 と Zen 3](#zen_2-zen_3)

{{< /pindex >}}

## Zen 2アーキテクチャ、x86 Model は 68h (104) {#fam17h-model68h}

**Ryzen 7 5700U** の CPUファミリーは `17h (23)` であり、L1命令キャッシュ 32KB、8-Core/16-Thread に対し、L3キャッシュ 4MB\*2 という点から *Zen 2アーキテクチャ* だと考えられる。  

x86 Model は `68h (104)` とされ、以前 「PPR for Family 17h Model 60h, Revision A1 Processor」 が公開された際は `Family 17h, Model 67h` が *Lucienne* ではないかと考えたが、実際はそこからずれた `Family 17h, Model 68h` となる。  
{{< link >}} [AMD、「PPR for Family 17h Model 60h, Revision A1 Processor」を公開 | Coelacanth's Dream](/posts/2020/07/09/amd-ppr-fam17h-model60h/) {{< /link >}}
その資料では、`Family 17h, Model 60h-67h` プロセッサは TDP 4.5〜55W の範囲に対応するとしていたが、*Renoir* が現在カバーしているのは TDP 10〜55W であるため、他に TDP 4.5〜10W をカバーする *Zen 2アーキテクチャ* を採用した APU {{< comple >}} 恐らくは `Family 17h, Model 67h` ？ {{< /comple >}} がいるのではないかと思われる。  

| Family17h Processor | CPU Arch | x86\_Model |
| :-- | :--: | :--: |
| Summit Ridge /Naples | Zen | 1h |
| Pinaccle Ridge /Colfax | Zen+ | 8h |
| Raven Ridge | Zen | 11h |
| Picasso | Zen+ | 18h |
| Raven2 /Dali | Zen | 18h |
| Dali (for Chromebook, some)<br>/Pollock | Zen | 20h |
| Matisse | Zen 2 | 71h |
| Castle Peak /Rome | Zen 2 | 31h |
| Renoir | Zen 2 | 60h
| Lucienne? | Zen 2 | 68h? |


## Lucienne と Cezanne の棲み分けを考える {#lcn-czn}

*Lucienne* は、*Raven* に対する *Picasso* のように、*Renoir* の後継となる APU だが、  
{{< link >}} [AMD Renoir の兄弟？ 現れるは Lucienne | Coelacanth's Dream](/posts/2020/06/20/amd-lucianne-apu/) {{< /link >}}
他にも、*Lucienne* と同時期に登場するとされている APU に、**Zen 3 + Vega** の構成を取る [Cezanne](/tags/cezanne) が存在する。  

一度、同時期に CPUアーキテクチャの世代が異なる APU が登場することの意味を考え、文字に出力してみる。  

### プロセス {#process}

下は、今回発見された **Ryzen 7 5700U** と同じ CPUコア/スレッド数、GPU CU数となる **Ryzen 7 4800U (Renoir)** の仕様を比較した表。  
CPU ブーストクロック、GPU クロック共に、**Ryzen 7 5700U** が 100〜200MHz 高い。  

| | Ryzen 7 4800U [^rn-4800u] | Ryzen 7 5700U |
| :-- | :--: | :--: |
| CPU Arch | Zen 2 | Zen 2 |
| &emsp;&emsp;Core/Thread | 8/16 | 8/16 |
| &emsp;&emsp;Base Clock | 1.8 GHz | 1.8 GHz |
| &emsp;&emsp;Boost Clock | 4.2 GHz | 4.34 GHz<br>(4.3-4.4) |
| GPU Arch | gfx90c | gfx90c? |
| &emsp;&emsp;CU | 8 | 8 |
| &emsp;&emsp;Clock | 1.75 GHz | 1.9 GHz? |

[^rn-4800u]: [AMD Ryzen™ 7 4800U | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-4800u#product-specs)

*Lucienne* 、*Cezanne* の製造プロセスは当然まだ明らかにされていないが、*Renoir* や **Ryzen 5000シリーズ** の TSMC 7nmプロセス (N7) か、TSMC 第二世代7nmプロセス (N7P) 等が候補として考えられる。  
N7P は N7 と完全なデザイン互換製を持ち、N7 と比較して、同じ消費電力なら 7% 高い性能を、同じ性能なら 10% 低い消費電力を実現させられるとしている。[^n7p]  
7% の性能向上というのは、**Ryzen 7 4800U** と **Ryzen 7 5700U** のクロック差に近くはある。  

[^n7p]: [TSMC Talks 7nm, 5nm, Yield, And Next-Gen 5G And HPC Packaging – WikiChip Fuse](https://fuse.wikichip.org/news/2567/tsmc-talks-7nm-5nm-yield-and-next-gen-5g-and-hpc-packaging/)

モバイル向けプロセッサを多く確保するため、例えば *Lucienne* は N7P、*Cezanne* は N7 という風にラインを分けるということも思い付くが、どのように棲み分けるかには繋がらない。  
アーキテクチャ差も埋められはしないだろう。  

### Zen 2 と Zen 3 {#zen_2-zen_3}

AMD の発表では、**Ryzen 5000シリーズ** は前世代の **Ryzen 3000シリーズ** と比較して、24% 高いとしている。[^zen_3]  
だが、絶対的な消費電力は、費やすトランジスタ数を増やし、クロックも上げた分、増えているものと考えられる。  
レビュー記事を読んでも、**Ryzen 7 5800X** の実効消費電力は、同じ 8-Core/16-Thread である **Ryzen 7 3800XT** と比較して約 20% 高いものとなっている。[^r50-review]これは APU である *Cezanne* でも同様だろう。  

[^zen_3]: [AMD "Zen 3" Core Architecture | AMD](https://www.amd.com/en/technologies/zen-core-3)
[^r50-review]: [Ryzen 5000シリーズを試す - 性能編「Ryzen 9 5900X」と「Ryzen 7 5800X」 (5) | マイナビニュース](https://news.mynavi.jp/article/20201105-1457526/5)

*Zen 3アーキテクチャ* の電力効率が高いとしても、それはデスクトップ向けの高い TDP である場合で、同時に絶対的な消費電力の低さが求められるモバイル向けでは *Zen 2アーキテクチャ* のが電力効率では逆転する可能性も考えられる。  

*Lucienne* と *Cezanne* でデフォルトTDPの値は変わらないかもしれないが、cTDPの範囲、ブースト機能の設定、求められる冷却能力は変わってくるのではないだろうか。  

また、*Lucienne* は L3キャッシュ 4MB\*2 の構成だが、*Cezanne* は先日公開された 「Software Optimization Guide for the AMD Family 19h Processors」 から、L3キャッシュ 16MB\*1 の構成になると思われる。  
{{< link >}} [AMD、Zen 3 アーキテクチャの詳細を公開 | Coelacanth's Dream](/posts/2020/11/07/amd-zen_3-arch-detail/) {{< /link >}}
*Cezanne* の総L3キャッシュ容量は *Lucienne* の倍となり、CPUコアの規模も大きくなるため、順当に *Cezanne* のダイサイズは大きく、製造コストは高いものとなるだろう。  

{{< comple >}} それ程までに規模に差がある訳ではないが {{< /comple >}} *Zen 2アーキテクチャ* と *Zen 3アーキテクチャ* は、Intel の *Core系アーキテクチャ* と *Atom系アーキテクチャ* の関係に似たものを覚える。  

まとめると、同じ TDP であっても、*Cezanne* は高性能だが採用製品の価格は高く、  
*Lucienne* は *Cezanne* より性能は劣るが、実効消費電力は低く {{< comple >}} バッテリー持続時間は長く {{< /comple >}}、コストパフォーマンスに優れる、といった棲み分けになるのではないかと考える。  


