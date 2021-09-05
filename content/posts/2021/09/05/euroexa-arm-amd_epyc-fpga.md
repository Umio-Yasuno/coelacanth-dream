---
title: "Arm Cortex & AMD EPYC、FPGA によるシステム構築を計画する EuroEXAプロジェクト"
date: 2021-09-05T10:44:52+09:00
draft: false
tags: [ "HPC", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

エクサスケールの性能を持つ HPC を実現する上での壁、課題を解決することを目的とする [EuroEXA](https://euroexa.eu) プロジェクトは現在、Arm Cortex と AMD EPYC、アクセラレーターには FPGA を用いるシステムを計画し、開発を進めている。  
2021/08/31 付で公開された「European HPC Handbook 2021」(Page 32) にて明かされた。  

 * [EU EuroEXA Project (@euroexa) / Twitter](https://twitter.com/euroexa)
    * <https://twitter.com/euroexa/status/1432715138914729989>
 * [The new edition of our Handbook of European HPC projects is out! | etp4hpc](https://www.etp4hpc.eu/news/268-the-new-edition-of-our-handbook-of-european-h.html)

EuroEXAプロジェクトは 2022/23年にエクサスケールのシステムを調達、展開するため、システムの開発を進めている。  

エクサスケールの壁として EuroEXA は以下の項目を挙げている。  

 * Energy Efficiency (電力効率)
 * Resilience (障害からの復旧性、復旧速度)
 * Performance and Scalability (性能と拡張性)
 * Programmability (プログラマビリティ、広範なオープンソースアプリケーションをサポート)
 * Practicality (実用性、創薬開発等)

EuroEXAプロジェクトではアクセラレーターとして FPGA を採用する予定にある。  
その利点としてプロジェクトは、創薬におけるバーチャルスクリーニング (Virtual screening) で FPGA は CPU か GPU を用いる場合と比べて 10倍速く、その上消費電力も少ないとしている。  

 * [Application Development — EUROEXA](https://euroexa.eu/application-development)

## AMD EPYC + Xilinx FPGA

今回自分が最も興味を惹かれたのは AMD EPYC と FPGA が組み合わされるという点で、テストベッドの構成から、FPGA には Xilinx のものが採用されると見られる。[^euroexa-testbed]  
そして AMD は 2020/10/27付で Xilinx の買収を発表、2021/04/07 には株式投票で承認されたことを発表している。  

 * [AMDが米Xilinxを350億ドルで買収。データセンター領域の強化見込む - PC Watch](https://pc.watch.impress.co.jp/docs/news/1285651.html)
 * [AMDのXilinx買収が株主投票で承認 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1317257.html)

[^euroexa-testbed]: [Testbeds — EUROEXA](https://euroexa.eu/testbeds)

AMD による Xilinx の買収には様々な反応があったようだが[^amd-xilinx]、EuroEXAプロジェクトでその最初の具体的な成果が公開されるかもしれない。  

また、EuroEXA は当初、Arm CPU を搭載する Xilinx FPGA でシステムを開発していたようだが、そこに AMD EPYC を組み合わせる構成に変更されている。  

 * [EU、ARM+FPGAのエクサスケール計画に2,000万ユーロの資金提供へ | HPCwire Japan](https://www.hpcwire.jp/archives/13106)
 * [EU Projects Unite on Heterogeneous ARM-based Exascale Prototype](https://www.hpcwire.com/2016/02/24/eu-projects-unite-exascale-prototype/)

[^amd-xilinx]: [AMD、Xilinxの買収で世界第4位のファブレスICメーカーへ - TrendForce分析 | TECH+](https://news.mynavi.jp/article/20201102-1445676/), [AMDによるXilinxの買収について思うこと - 吉川明日論の半導体放談(161) | TECH+](https://news.mynavi.jp/article/semicon-161/)

テストベッドの最終段階では *1x AMD EPYC; 1x Xilinx FPGA Accelerator; 1x Xilinx Storage/Network Processor/FPGA* で構成されたノードでクラスタを構築しており、AMD EPYC を採用する方向で固まっていると思われる。  
昨今の Arm IP を巡る情勢、AMD の Xilinx 買収などを受けての変更なのだろうか？  

最後に、自分が所属する Mastodon サーバーがバレて欲しくないため、名前を出せないのが申し訳ないが、  
自分に EuroEXAプロジェクトと European HPC Handbook 2021 の存在を知る切っ掛けを与えてくださった方に感謝を。  

{{< ref >}}
 * [EUROEXA](https://euroexa.eu/)
    * [Testbeds — EUROEXA](https://euroexa.eu/testbeds)
    * [Application Development — EUROEXA](https://euroexa.eu/application-development)
 * [AMD Powering the Exascale Era with El Capitan | AMD](https://www.amd.com/en/products/exascale-era)
 * [AMD to Acquire Xilinx | AMD](https://www.amd.com/ja/corporate/xilinx-acquisition)
 * [AMD to Acquire Xilinx, Creating the Industry’s High Performance Computing Leader :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/977/amd-to-acquire-xilinx-creating-the-industrys-high)
 * [AMD and Xilinx Stockholders Overwhelmingly Approve AMD’s Acquisition of Xilinx](https://www.amd.com/en/press-releases/2021-04-07-amd-and-xilinx-stockholders-overwhelmingly-approve-amd-s-acquisition)
 * [AMD、Xilinxの買収で世界第4位のファブレスICメーカーへ - TrendForce分析 | TECH+](https://news.mynavi.jp/article/20201102-1445676/)
 * [AMDによるXilinxの買収について思うこと - 吉川明日論の半導体放談(161) | TECH+](https://news.mynavi.jp/article/semicon-161/)
{{< /ref >}}

