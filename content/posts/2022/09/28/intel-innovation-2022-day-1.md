---
title: "Intel Innovation 2022 Day 1 個人的メモ"
date: 2022-09-28T18:51:30+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Alder_Lake", "Raptor_Lake", ]
noindex: true
# summary: ""
# keywords: [ "", ]
# author: ""
---

## Raptor Lake {#raptor_lake}
第 13世代 Intel プロセッサとして、コードネーム *Raptor Lake* が正式発表された。  
*Raptor Lake* は *Alder Lake* に採用されている *Golden Cove (Core, P-Core) + Gracemont (Atom, E-Core)* からマイクロアーキテクチャ自体は変更されていないと思われるが、P-Core は *Raptor Cove* となった。  
*Raptor Cove* では Intel 7 プロセスの改良、動作クロックの向上、P-Core の L2キャッシュの増量 (1.25 MB -\> 2.0 MB) と "L2P" と呼ぶダイナミックプリフェッチアルゴリズムの導入によりコアあたりの性能を向上させている。  

 * [13th Gen Intel Core Desktop Media Presentation - 13th-Gen-Intel-Core-Desktop-Media-presentation.pdf](https://download.intel.com/newsroom/2022/2022innovation/13th-Gen-Intel-Core-Desktop-Media-presentation.pdf)

プロセッサ全体としては、E-Core Module/Cluster を *Alder Lake* の最大 2基 (8 E-Core) から 4基 (16 E-Core) に増やしている。E-Core Module/Cluster 自体は *Alder Lake* と同じとされる。  
E-Core Module/Cluster を増やしたことで LLC (Last Level Cache) の容量も増えた。  
内部ファブリックの高速化、メモリレイテンシの削減も合わさって、マルチスレッド性能は 41% 向上したと主張している。(Core i9-13900K vs Core i9-12900K)

Intel Thread Director (ITD)、EHFI (Enhanced Hardware Feedback Interface) も *Alder Lake* から改良されている。  
PC Watch の 笠原 一輝 氏の記事によると、機械学習を利用してスレッドの分類がより細かく改善され、また *Alder Lake* から P-Core ごとに ITD 用のマイクロコントローラを搭載している。[^pc-watch]  
ただ Intel が公開しているメディア向けプレゼンテーションを見るに、スレッドの種類 (Class) 数は *Alder Lake* は変わらず 4種類となっている。また Class の判別に機械学習を用いているのは *Alder Lake* からであり、Hot Chips 33 でそのことが発表されている。[^hc33]  
要はファームウェアの改良によってスレッドの種類の判別がより正確になった、ということと思われる。  

[^hc33]: [HC2021.C1.1 Intel Efraim Rotem.pdf](https://hc33.hotchips.org/assets/program/conference/day1/HC2021.C1.1%20Intel%20Efraim%20Rotem.pdf)
[^pc-watch]: [【笠原一輝のユビキタス情報局】Alder Lake/Raptor Lakeの「高性能の秘密」はPコアに内蔵されたマイクロコントローラにあった！ - PC Watch](https://pc.watch.impress.co.jp/docs/column/ubiq/1442699.html)

P-Core ごとに ITD 用のマイクロコントローラを搭載していることについては、ITD/EHFI では各コア Class ごとの性能と電力効率を表すレベルが提供されるため、実際には E-Core にも ITD 用のマイクロコントローラが搭載されているのではないかと思われる。  
あるいは P-Core が E-Core の状況も監視しているか、またはコア全体を監視するマイクロコントローラが存在する可能性もあるが。  

{{< ref >}}
 * [Linux Kernel に Intel Thread Director を実装するパッチ | Coelacanth's Dream](/posts/2022/09/10/linux-kernel-intel-itd/)
{{< /ref >}}
