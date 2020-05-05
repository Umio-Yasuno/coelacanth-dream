---
title: "Rocket Lakeの噂、そこからの推測"
date: 2020-05-04T21:19:30+09:00
draft: false
tags: [ "Rocket_Lake", "Guess", "Rumor" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU", "GPU" ]
noindex: false
---

先日、Intel は *Rocket Lake* に関連するパッチを Linux Kernel へ投稿した。  

{{< link >}}[Intel、Rocket Lake 一連の Linux Kernelパッチを投稿 | Coelacanth's Dream](/posts/2020/05/01/intel-send-kernel-patch-rocketlake/){{< /link >}}

それよりもずっと前から *Rocket Lake* の噂、未確定情報は流れており、いつその名前が出てきたかは把握していないが、Wikichipのページの履歴を見ると、2019/06/04 に作られたことになっている。  
しかし噂は噂で、11ヶ月近く経ってもまだはっきりしていない部分は多い。  
今回は、普段は取り上げない噂、未確定情報をガッツリ取り入れて個人的な、ひどく個人的な *推測* を行なってみた。  

## 確定情報 {#fact}

 * GPUの世代は[Gen12](/tags/gen12)[^1]
 * PCHは *Comet Lake PCH* 、*Tiger Lake LP PCH* とされている[^2]
 * それまでのプラットフォームとは異なるメモリ特性[^3]

[^1]: [[Intel-gfx] [PATCH 00/23] Introduce Rocket Lake](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238498.html)
[^2]: [[Intel-gfx] [PATCH 05/23] drm/i915/rkl: Add PCH support](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238515.html)
[^3]: [[Intel-gfx] [PATCH 06/23] drm/i915/rkl: Update memory bandwidth parameters](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238505.html)

## 噂、未確定情報 {#rumor}

 * MCM(Multi-chip Module)構成[^4]
 * CPUアーキテクチャは *Skylake* 、もしくはそれ以降の *Suny Cove (Ice Lakeのベース)* 、 *Willow Cove (Tiger Lakeのベース)* [^4]
 * CPUダイは14nmプロセスで製造される[^4]
 * CPUコア数は最大8-Core[^8]
 * GPU EU数は最大32EU[^8]
 * PCIe Gen4.0のサポート[^4]
 * CPUから使用可能なPCIeレーンがx4分追加される[^4]

[^4]: [Exclusive: Intel Rocket Lake-S features PCI-Express 4.0, Xe Graphics - VideoCardz.com](https://videocardz.com/newz/exclusive-intel-rocket-lake-s-features-pci-express-4-0-xe-graphics)
[^8]: [Intel "Rocket Lake-S" Desktop Processor Comes in Core Counts Up to 8, Gen12 iGPU Included | TechPowerUp](https://www.techpowerup.com/261611/intel-rocket-lake-s-desktop-processor-comes-in-core-counts-up-to-8-gen12-igpu-included)<br>&emsp;<https://twitter.com/momomo_us/status/1200042853587537921>

## インデックス

 * [推測](#guess)
    * [PCIe Gen4.0 とそのレーン数](#pcie)
    * [Rocket Lake の GPU は推されるか？](#rkl-gpu)
    * [CPUアーキテクチャは Skylake 以降](#cpu-arch)
    * [MCM](#mcm)
        * [パターン1: CPU+I/O(14nmm) + GPU+Display(10nm)](#pattern1)
        * [パターン2: CPU(10nm) + GPU+I/O(14nm)](#pattern2)


## 推測 {#guess}
### PCIe Gen4.0 とそのレーン数 {#pcie}
既に発表されている Z490マザーボードの中に、デスクトップ向け *Comet Lake* では使用不可能な M.2スロット (PCIe x4 CPU側) を備えるもの、[^5]  
PCIe Gen4.0の対応を謳うもの[^6]があるのは確かで、確度は高いと言える。  

[^5]: [Z490 AORUS PRO AX (rev. 1.0) | Motherboard - GIGABYTE Global](https://www.gigabyte.com/Motherboard/Z490-AORUS-PRO-AX-rev-10/support)<br>&emsp;<https://download.gigabyte.com/FileList/Manual/mb_manual_z490-aorus-pro-ax_1001_e.pdf>
[^6]: [ASRock > Z490 PG Velocita](https://www.asrock.com/mb/Intel/Z490%20PG%20Velocita/index.asp)

マザーボード関連で言えば、特にCPU、GPU共にアーキテクチャが変わらない前世代 *Coffee Lake* 向けのマザーボードではサポートされていなかった DisplayPort 1.4 のサポートが、Z490マザーボードでは追加されていたりする。  
それでも対応解像度は 4096x2304@60Hz と変わっておらず、これも *Rocket Lake* へ先駆けた対応ではないかと思う。  
DisplayPort 1.4への対応は、GPUが[Gen11](/tags/gen11)アーキテクチャである *Ice Lake-U/Y* で既に行なわれており、[^7]Gen12 の *Tiger Lake* 、 *Rocket Lake* でも対応するのは順当であるはずだ。  

[^7]: [Intelligent Performance 10th Gen Intel® Core™ Processors Brief](https://www.intel.com/content/www/us/en/products/docs/processors/core/10th-gen-core-mobile-processors-brief.html?wapkw=ice%20lake%20brief)

### Rocket Lake の GPU は推されるか？ {#rkl-gpu}
Gen12アーキテクチャとなる *Rocket Lake* のGPUだが、積極的に推す気はないのではないかと思う。  

デスクトップ向けのプラットフォームでは、モバイル向けのように LPDDR4/xメモリを採用することができず、メモリ帯域がボトルネックとなりやすい統合GPUの弱点はそのままだ。  
これを解消しようとすると *Broadwell* や *Coffee Lake-U* のように eDRAMをパッケージ内に搭載するといった手法が求められる。  
しかし、*Rocket Lake* のGPU規模は32EUという話だ。  
これらを結びつけると、*Rocket Lake* でGPU性能を高め、アピールしようとする意図は小さいのではないかと考えられる。  
メディア機能の強化や最新世代である Gen12アーキテクチャの普及といった意図の方が大きそうだ。  

また、上で DisplayPort 1.4 への対応は *Rocket Lake* に向けたものとしたが、対して HDMI は 1.4(4096x2160@30Hz)のままだ。*Ice Lake-U/Y* は HDMI 2.0b に対応している。[^7]\(HDMI 2.0に対応したのは *Gemini Lake* から\)[^9]  

[^9]: [Intel Launches New Pentium Silver and Celeron Atom Processors: Gemini Lake is Here](https://www.anandtech.com/show/12146/intel-launches-gemini-lake-pentium-silver-and-celeron-socs-new-cpu-media-features)

よって、*Rocket Lake* 新機能の見所は `CPU > GPU >= I/O` くらいに思っている。  

### CPUアーキテクチャは Skylake 以降 {#cpu-arch}
CPUアーキテクチャはさすがに *Skylake* 以降になるはずだ。  
だって単純に考えて、10-Core *Comet Lake* が出た後にアーキテクチャは特に変わらない 8-Core CPU、GPUもそこまで規模は大きくないプロセッサを出すだろうか？  

そうなると気になるのはプロセッサの構成だ。  

### MCM構成 {#mcm}
#### パターン1: CPU+I/O(14nmm) + GPU+Display(10nm) {#pattern1}
CPUアーキテクチャは *Suny Cove* か *Willow Cove* で、それを14nmプロセスで製造するという話だ。  
この場合、基本的なI/O(メモリコントローラー、LP/DDR4 PHY、PCIe等)はCPUダイ側に実装されると考えられる。  
I/Oのパッド部は微細化の恩恵を受けづらく、最先端プロセスでI/Oを多く含んだダイを製造しようとすると、ダイサイズが肥大化し、コストが高く付いてしまう。  

Intelも、違うプロセスで製造したダイを1つのパッケージに収めた非対称プロセッサの構想を示した際、CPU/GPUダイは 10nmプロセス、I/Oは成熟した 14nmプロセスとしていた。[^10]  
非対称ハイブリッドプロセッサ *Lakefield* も、I/Oの多くは Base die と呼ぶ 22nmで製造されるダイにまとめられている。[^11]  

[^10]: [【後藤弘茂のWeekly海外ニュース】IntelがHBM2とAMD GPUダイを統合した「Kaby Lake-G」を年内に投入 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1054618.html)
[^11]: <https://www.hotchips.org/hc31/HC31_2.10_LKF_HC_2019_Final_v7.pdf>

GPUは別ダイに存在するメモリコントローラを経由してメモリにアクセスすることとなるが、これが「これまでとは異なるメモリ特性」[^3]の理由と考えられなくもない。  
GPU性能に悪影響を与えるかもしれないが、上述したようにデスクトップ向け *Rocket Lake* においてGPUは推されない、重要でないとされているように自分は考えている。  
モバイル向け10世代プロセッサで、*Comet Lake* はCPUの絶対性能、*Ice Lake* はAI処理性能とGPU性能と棲み分けしたのが、その次世代で、  
前者の位置に *Rocket Lake* 、後者に *Tiger Lake* が棲むこととなるだろう。  

CPUアーキテクチャが *Skylake* からそれ以降となることで、コア部が占める面積は大きくなるが、現行の Intelプロセッサを見てもGPU部はそれなりに面積を取っており、GPUを別ダイとすることで、CPUダイ自体のサイズは現行プロセッサとそう変わらないのではないかと思う。  

成熟した14nmプロセスではあるが、キャッシュ増量等の理由でこれまでの *Skylake* 系プロセッサ程のクロックは難しいと考えられる。  
また、発熱を分散させる意味もあったGPUが別ダイとなったことで、冷却が難しくなるといった問題が出てくる可能性もある。  

14nmプロセス、新アーキテクチャ、キャッシュ構成、熱密度、これらが組み合わさり、最終的にどの程度のクロックになるかは気になるところだ。  
Intelは *Suny Cove (Ice Lake)* でIPCを平均18%向上させたとあるため、4.5GHzは達成して欲しいのが個人的な心情。  
(SkyalkeのIPCを1、Rocket Lakeに達成して欲しいクロックを x として、1 * 5.3[GHz] = 5.3, 1.18 * x = 5.3, x = 4.5)  

#### パターン2: CPU(10nm) + GPU+I/O(14nm) {#pattern2}
言ってしまえば、AMDが取ったチップレット構成に近い。  

サイズの大きいCPUダイを最新プロセスで製造し、規模の小さいGPU、それとI/Oを成熟したプロセスで製造する。  
この場合、CPUダイを小さくし、消費電力も低くすることができるが、クロックと熱密度の問題は14nmプロセスで製造したときよりも顕在化してしまう。  
デスクトップ向けというのを考えればパターン1の方が調和しているように思える。

{{< ref >}}

 * [Intel、「Comet Lake」ベースの8製品を第10世代Coreプロセッサに追加投入 | マイナビニュース](https://news.mynavi.jp/article/20190821-881022/)
 * [北森瓦版 - 一部のマザーボードには“Rocket Lake-S”用のM.2スロットがある模様](https://northwood.blog.fc2.com/blog-entry-10255.html)
    * [Intel Rocket Lake: Das verraten Z490-Mainboards über Intels nächste CPU-Generation](https://www.pcgameshardware.de/CPU-CPU-154106/Specials/rocket-lake-vorhersage-pcie-ddr-1349116/)
    * [Z490 AORUS PRO AX (rev. 1.0) | Motherboard - GIGABYTE Global](https://www.gigabyte.com/Motherboard/Z490-AORUS-PRO-AX-rev-10/support)<br>&emsp;<https://download.gigabyte.com/FileList/Manual/mb_manual_z490-aorus-pro-ax_1001_e.pdf>
    * [ASRock > Z490 PG Velocita](https://www.asrock.com/mb/Intel/Z490%20PG%20Velocita/index.asp)
 * [Rocket Lake - Microarchitectures - Intel - WikiChip](https://en.wikichip.org/wiki/intel/microarchitectures/rocket_lake)
 * [Exclusive: Intel Rocket Lake-S features PCI-Express 4.0, Xe Graphics - VideoCardz.com](https://videocardz.com/newz/exclusive-intel-rocket-lake-s-features-pci-express-4-0-xe-graphics)
 * [Intel "Rocket Lake-S" Desktop Processor Comes in Core Counts Up to 8, Gen12 iGPU Included | TechPowerUp](https://www.techpowerup.com/261611/intel-rocket-lake-s-desktop-processor-comes-in-core-counts-up-to-8-gen12-igpu-included)
     * <https://twitter.com/momomo_us/status/1200042853587537921>
