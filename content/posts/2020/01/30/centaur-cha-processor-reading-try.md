---
title: "Centaur CHAプロセッサーについて読んでみたかった"
date: 2020-01-30T09:57:53+09:00
draft: true
tags: [ "Centaur", "" ]
keywords: [ "", ]
categories: [ "Hardware", "CPU" ]
noindex: false
---

製造プロセスはTSMC 16nm FinFet Compact(16FFC)。  
16FFCは
ダイサイズは 194mm<sub>2</sub>だが、トランジスタ数は不明。  

2ソケットの構成にも対応する。  

名称は、SoC全体のコードネームが**CHA**、x86 CPU部のマイクロアーキテクチャ名が**CNS**、DLA(deep-learning accelarator)部が**Ncore**とされている。  

### Security (Spectre, Meltdown)
言ってしまえば、不明。  
ただ正式には公表されていないものの、過去のVIAプロセッサはSpectre、Meltdownの影響を受けるとされている。  
[Windows月例更新でVIAプロセッサにもSpectre/Meltdown対策が盛り込まれる - PC Watch](https://pc.watch.impress.co.jp/docs/news/1180455.html)  

資料でもRedditのスレッド（後になってからだが質問をしたユーザーはいた）でも特に述べられていないことから、ハードウェアでの対策は施されていない可能性が高い。  

余談だが、Centaur Technologyが設計した中国市場向けCPU、Zhaoxin（兆芯）は、  
次世代の7-Seriesにて、SpsctreV2とSWAPGSの対策が施されている。  
[Zhaoxin 7-Series x86 CPUs Mitigated For Spectre V2 + SWAPGS - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Zhaoxin-7-Series-Mitigations)  
[[PATCH] x86/speculation/spectre_v2: Exclude Zhaoxin CPUs from SPECTRE_V2 - Linux-Lernel Archive](http://lkml.iu.edu/hypermail/linux/kernel/2001.1/08763.html)  

このパッチを読む限り、x86 CPUでSpectreV2への対策が為されたのはZhaxin 7-Seriesが何気に初となるのだろうか。  
AMDはZen2アーキテクチャでSpectre対策強化のため一部設計を変更したが、あくまで強化であり、ソフトウェア側（OS Kernel、ファームウェア）での対策が不要になる訳ではなかった。  
[AMD、「Spectre」対策で次世代チップ「Zen 2」の設計を変更 - CNET Japan](https://japan.cnet.com/article/35114046/)  
[ASCII.jp：判明した第3世代Ryzenの内部構造を大解説　AMD CPUロードマップ (3/4)](https://ascii.jp/elem/000/001/882/1882171/3/)  
