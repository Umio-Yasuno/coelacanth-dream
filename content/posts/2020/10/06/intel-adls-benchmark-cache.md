---
title: "Intel Alder Lake-S のベンチマーク結果から読むキャッシュ構成"
date: 2020-10-06T19:36:35+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

[APISAK (@TUM_APISAK) / Twitter](https://twitter.com/TUM_APISAK) 氏が *Alder Lake-S* の SiSoftware の実行結果を発見、それを Twitter にて共有した。  
そして、結果には *Alder Lake-S* のコア、スレッド数、キャッシュ容量が記されており、*Alder Lake-S* のベールがまた少し上げられることとなる。  
{{< link >}} [Details for Computer/Device Intel Alder Lake Client Platform Alder Lake Client System (Intel AlderLake-S ADP-S DRR4 CRB) : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_system.php?q=cea598ab93aa98a193b5d2efc9bb86a0c9f4d2ba87a1d9e4c2a7c2ffcfe99aa79f&l=en) {{< /link >}}

{{< pindex >}}

 * [Alder Lake-S の構成推測](#adl-s-spec-guess)
 * [有効にされている HTT](#enable-htt)
 * [Alder Lake のメモリと GPU](#adl-memory-gpu)

{{< /pindex >}}

## Alder Lake-S の構成推測 {#adl-s-spec-guess}

発見された *Alder Lake-S* の構成は、16-Core/32-Thread、L2キャッシュ 10x1.25MB、L3キャッシュ 30MB となっている。  
16-Core に対して L2キャッシュ 10基という一見奇妙なものだが、これまでの情報から詳細を推測することが可能だ。  

まず、*Alder Lake* は *Core系アーキテクチャ: Golden Cove* と *Atom系アーキテクチャ: Gracemont* を組み合わせたハイブリッド構成であることが、Intel Architecture Day 2020 にて明かされている。[^intel-arch-day-2020]  

[^intel-arch-day-2020]: [Intel Architecture Day 2020 個人的まとめ　―― XeHP は 1-Tile 512EU、XeLPアーキテクチャ詳細 | Coelacanth's Dream](/posts/2020/08/14/intel-architecture-day-2020/#adl)

次に、デスクトップ向けとされる *Alder Lake-S* は {{< comple >}} 今は削除されてしまっているが {{< /comple >}} それぞれ 8-Core ずつ、計 16-Core が最大規模の構成になると思われる。[^adl-core-atom-config]  
今回発見された *Alder Lake-S* はその最大規模のものだろう。  

[^adl-core-atom-config]: [Coreboot へのパッチから Intel Alder Lake-S/P のコア構成が判明か | Coelacanth's Dream](/posts/2020/08/04/intel-adl-core-config/)

10基の L2キャッシュに関してだが、現 *Atom系アーキテクチャ* の [Tremont](/tags/tremont) では、4-Core とそれらで共有する L2キャッシュでクラスタ(またはタイル) を構成している。  
恐らく、この構成が次 *Atom系アーキテクチャ* である *Gracemont* でも引き継がれており、*Golden Cove 8-Core (8x L2) + Gracemont 2-Tile(8-Core, 2x L2)* 、合わせて L2キャッシュ 10基として認識されたものと思われる。  

*Tremont* では共有する L2キャッシュ容量を 1MB から 4.5MB までの範囲で選択可能となっており、現世代のハイブリッドプロセッサ [Lakefield](/tags/lakefield) では 1.5MB としている。  
*Alder Lake-S* の Atom側が L2キャッシュ容量が 1.25MB とすると、*Lakefield* より小さいことになるが、異なるアーキテクチャとトポロジのキャッシュ容量の検出にソフトウェアがまだ対応しておらず、Core側の L2キャッシュを読み取って、それを 10基と表示した可能性もある。  

L3キャッシュ(LLC, Last Level Cache) も、*Golden Cove* のコア、*Gracemont* のタイル数に合わせて増やしていると考えられ、その場合 3MB(per Core or Title) となる。  
*Lakefield* でもこのようになっており、*Sunny Cove* 1コア、*Tremont* 1タイル(4-Core) に対し、LLC 2MB(per Core or Tile) という構成。  

L2 1.25MB、LLC 3MB というのは、*Tiger Lake* に採用されている *Willow Coveアーキテクチャ* と同様の構成であり、*Golden Cove* では新たにキャッシュ構成を変更することはしなかったものと思われる。  
Intel が以前公開した CPU Core Roadmap でも、*Willow Cove* には "Cache redesign" の文言があるが、*Golden Cove* ではキャッシュに関して特に触れられていない。[^intel-cpu-core-roadmap]  

[^intel-cpu-core-roadmap]: [Intel、次世代CPUアーキテクチャ「Sunny Cove」の概要を明らかに ～Willow Cove、Golden Coveと進化予定、Atomのロードマップも更新 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1158093.html)

| Alder Lake-S | Golden Cove<br>(Core) | Gracemont<br>(Atom) |
| :-- | :--: | :--: |
| Core/Thread | 8/16 | 8/16 |
| L2cache | 8x 1.25MB?<br>(per Core) | 2x 1.25MB???<br>(per Tile) |
| LLC | 8x 3MB?<br>(per Core) | 2x 3MB?<br>(per Tile) |

## 有効にされている HTT {#enable-htt}

そして、キャッシュ構成以外に注目されるのがスレッド数であり、*Lakefield* では両アーキテクチャコアで無効化されていた HTT(Hyper Threading Technology) が *Alder Lake-S* では有効化されている。  
*Alder Lake* ではハイブリッドの構成に関して、*Lakefield* よりも拡張されたハードウェアベースのスケジューラーを実装していることが Intel Architecture Day 2020 にて Raja Koduri 氏より語られており、その効果が HTT有効化に繋がっているのかもしれない。  

## Alder Lake のメモリと GPU {#adl-memory-gpu}

今回の情報ではメモリが DDR4 とされているが、*Alder Lake* は DDR4/LPDDR4/LPDDR5/DDR5 に対応するため、特におかしいところはない。[^adl-mc]  
それに現時点で DDR5メモリを検証に使うにしても、それもまたサンプル品となってしまうのではないかと思う。  

 >       +-------------+--------+---------+
 >       | Memory Type | Max Ch | Max DPC |
 >       +-------------+--------+---------+
 >       | DDR4        |      1 |       2 |
 >       | LPDDR4(X)   |      4 |       1 |
 >       | LPDDR5      |      4 |       1 |
 >       | DDR5        |      2 |       1 |
 >       +-------------+--------+---------+
 >
 >    These numbers are for a single memory controller. Since ADL has two memory controllers, there's twice as many channels in total. DPC stays the same.
 >
 >
 > {{< quote>}} <https://review.coreboot.org/c/coreboot/+/45192/7/src/soc/intel/alderlake/meminit.c#109> {{< /quote >}}

[^adl-mc]: <https://review.coreboot.org/c/coreboot/+/45192/7/src/soc/intel/alderlake/meminit.c#109>



それと実行結果で GPU が *AlderLake-S Mobile Graphics* となっているのは、*Tiger Lake-U* と同じ *Gen12LPアーキテクチャ* だからではないかと思われる。  
