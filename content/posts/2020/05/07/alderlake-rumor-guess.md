---
title: "Alder Lake の噂と推測 ――Alder Lake は現実に向けたプロセッサか"
date: 2020-05-07T01:52:51+09:00
draft: true
tags: [ "Alder_Lake", "Rumor", "Guess" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU"]
noindex: false
---

まだOSSのコードからは名前しか確認されておらず、素性には謎の多い *Alder Lake* だが、  
{{< link >}}[ChromiumOSへのパッチから Alderlake-S / Alderlake-P を確認 | Coelacanth's Dream](/posts/2020/05/01/vboot-code-add-alderlake/){{< /link >}}
[前回](/posts/2020/05/04/rocketlake-rumor-guess/)のように、噂、未確定情報から個人的な推測を試みた。  
*Rocket Lake* では構成やアーキテクチャについて思考を巡らせてみたが、今回は *Alder Lake-S* が取ると噂されている `big core + small core` が主となる。  

## 噂、未確定情報

 * *Alder Lake-S* には `8 * big + 8 * small` と `8 * big + 0 * small` 2種類の構成がある[^1]
 * ソケットは LGA1700[^1]

[^1]: [Intel Alder Lake-S rumored to feature LGA1700 socket - VideoCardz.com](https://videocardz.com/newz/intel-alder-lake-s-rumored-to-feature-lga1700-socket)

## インデックス

 * [推測](#guess)
     * [Big-Bigger](#big-bigger)
     * [現実的なアプリケーション、現実的なベンチマーク、現実に向けたプロセッサ](#real)

## 推測 {#guess}
### Big-Bigger {#big-bigger}

`big core + small core` という構成は *Lakefield* で採用されている。  

*Lakefield* は高性能な big core *Suny Cove コア* を 1コア、電力効率に優れる *Tremont コア* を 4コアを搭載し、処理によってそれらを使い分けることで、シングルスレッド性能を高く、消費電力を低くしている。  
具体的には、big core である *Suny Cove コア* は性能と応答性を求められる処理を、small core である *Tremont コア* ではバックグラウンドと軽い処理を行なう。  
そしてそれら全てのコアがスレッドを実行可能だ。  
ARMの big.LITTLE技術に近いが、Intel はHotchips31での発表の中で、Big-Bigger という言葉を使っている。  

それぞれアーキテクチャは不明だが、big core、small core と言うからには *Alder Lake* も同様の Big-Bigger 構成となるだろう。  
この構成をデスクトップ向けプロセッサにもたらすことの恩恵としては、ダークシリコン問題の軽減がまず考えられる。  

ダークシリコンとは、消費電力、排熱等の制約からシリコン上の電力を供給できない部分を示す。  
半導体の微細化と駆動電圧の低下が一致しなくなってからダークシリコンは増える一方だ。  
8nmプロセスノードでは、アーキテクチャ、冷却技術、アプリケーションに応じて、ダークシリコンの量が最大50~80%に達するとの予測もある。[^2]  
高性能なコアをひたすらに積んでも全てを最大クロックで動作させることはできず、性能に対するトランジスタ数の効率は良くない。  

[^2]: [Dark silicon - Wikipedia](https://en.wikipedia.org/wiki/Dark_silicon)<br>&emsp;[ISCA11 dark silicon the end of manycore era gpu etc.pdf](http://www.iuma.ulpgc.es/users/nunez/clases-micros-para-com/clases-mpc-slides-links/PH%20COD%20book%20ManyCore%20SMP%20OpenMP/ISCA11%20dark%20silicon%20the%20end%20of%20manycore%20era%20gpu%20etc.pdf)

Intel はダークシリコン問題の軽減のため、プロセッサに搭載するGPUの規模を増やすといった方法を取ってきた。  
新しいBig-Bigger構成では、軽い処理やバックグラウンド処理を small core に任せることで消費電力をより削減する目的もあると思われる。  
また、そのことで big core の性能低下の原因となるノイズを減らす効果もあるはずだ。  

### 現実的なアプリケーション、現実的なベンチマーク、現実に向けたプロセッサ {#real}

Intel は 2019 IFA で、"Real World Performance" という言葉を持ち出し、Cinebenchといったソフトによるテストは大部分が現実に即していないとし、次に一般的によく使われる人気のアプリケーションを示した。  
それらを用いたテストを行なうことで、より現実に近い性能データを得られるということだろう。  
{{< link >}}[Intel Is Suddenly Very Concerned With 'Real-World' Benchmarking - ExtremeTech](https://www.extremetech.com/computing/297864-intel-is-suddenly-very-concerned-with-real-world-benchmarking){{< /link >}}

これは当時、現実に即していればそれは優秀なテストとなるのか、その現実とはノートPCに限った現実であり、デスクトップやワークステーションではまた違った現実があるなどと議論を呼んだ記憶がある。  
性能テスト、ベンチマークは実際の使用感よりも、そのプロセッサの特性を知るために行なう側面もある。  

しかし、Big-Bigger構成を *Lakefield* で採用することが確定しており、*Alder Lake* にも取り入られるかもしれないという前提の中では、上の Intelの主張が少し別の意味を持って見えてくる。  
Intel は *Lakefield* の発表スライドの中で、Webブラウジングを実行中の各コア稼働状況を表したグラフを示していた。[^3]高いシングルスレッド性能を必要とするシーンと、そうでないシーンの切り替わりが煩雑なワークロードで Big-Bigger構成は真価を発揮する。  
要は現実的なアプリケーションにおけるワークロードと考えている。  

実際、普段使用する中でCPU使用率が100%に張り付くことがどれだけあるだろうか。  
自分の用途で言えば、動画のエンコードやソフトウェアのビルドを行なう時くらいで、普段使うことが多いWebブラウジングは通常 5%前後、高くて 20%程だ。  
ゲームはそれよりも高いが、やはり張り付くということは稀だろう。  

*Alder Lake* は `8 * big core` でピークシングルスレッド性能、マルチスレッド性能を確保しつつ、`small core` も搭載することでアイドルではない、通常使用時の微妙に負荷が掛かる状況での消費電力をより削減した、現実に向けたプロセッサなのかもしれない。  

そして、将来的に *Lakefield* 、*Alder Lake* を市場に投入する上で、ピーク性能とその時のピーク消費電力を計測するようなテストではなく、"現実"に即したアプリケーションとそれを用いたテストでなければ Big-Bigger構成を採用したプロセッサをアピールするのは難しい、と Intelは考えた……というのが個人的な推測。  

[^3]: <https://www.hotchips.org/hc31/HC31_2.10_LKF_HC_2019_Final_v7.pdf#page=16>

あくまでも推測。  
それに、 *Alder Lake* は 2021-2022年に登場すると言われているし、こういった推測は絵を描きつつも、そんなのを思い切り打ち破る実物が出てほしいと願うのが常である。

{{< ref >}}

 * [【後藤弘茂のWeekly海外ニュース】モバイルSoCにおけるダークシリコンの呪縛 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/549137.html)
 * [【後藤弘茂のWeekly海外ニュース】Intel「第8世代Core」に見る、微細化準備が整っても、製品を移行させない/させたくない理由 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1076333.html)
 * <https://newsroom.intel.com/wp-content/uploads/sites/11/2019/09/2019-IFA-Intel-RWPE.pdf>
 * [Intel Is Suddenly Very Concerned With 'Real-World' Benchmarking - ExtremeTech](https://www.extremetech.com/computing/297864-intel-is-suddenly-very-concerned-with-real-world-benchmarking)
 * [ASCII.jp：AtomベースのSmall CoreがTremontと判明　インテル CPUロードマップ (1/4)](https://ascii.jp/elem/000/001/981/1981403/)
 * <https://www.hotchips.org/hc31/HC31_2.10_LKF_HC_2019_Final_v7.pdf>

{{< /ref >}}
