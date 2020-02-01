---
title: "Centaur CHAプロセッサーについて読んでみたかった"
# date: 2020-01-30T09:57:53+09:00
date:	2020-02-01T19:39:13+09:00 
draft: true
tags: [ "Centaur", "" ]
keywords: [ "", ]
categories: [ "Hardware", "CPU" ]
noindex: false
---

製造プロセスはTSMC 16nm FinFet Compact(16FFC)。  
ハイパフォーマンス向けの16FF+を低コストにしたのが16FFC(16FF Compact)であり、メインストリーム向けや低消費電力(Ultra Low Power)向けに位置付けされている。  

 > 参考:  
 [10nmに見切りをつけ低コストの12FFCに注力　TSMC 半導体ロードマップ - ASCII.jp](https://ascii.jp/elem/000/001/516/1516220/2/)  
 [16/12nm Technology - Taiwan Semiconductor Manufacturing Company Limited](https://www.tsmc.com/english/dedicatedFoundry/technology/16nm.htm)  
 [TSMC Symposium: New 16FFC and 28HPC+ Processes Target “Mainstream” Designers and Internet of Things (IoT) - Industry Insights - Cadence Blogs - Cadence Community](https://community.cadence.com/cadence_blogs_8/b/ii/posts/tsmc-symposium-new-16ffc-and-28hpc-processes-target-mainstream-designers-and-internet-of-things-iot)  

<!--
ダイサイズは 194mm<sup>2</sup>だが、トランジスタ数は不明。
2ソケットの構成にも対応

-->

名称は、SoC全体のコードネームが**CHA**、x86 CPU部のマイクロアーキテクチャ名が**CNS**、DLA(deep-learning accelarator)部が**Ncore**とされている。  

##### I/O

#### CNS (x86 CPU)
<!--
CPUの最高クロックを制限することで物理設計の最適化に費やす時間を削減した

CHAのTDPを公表していないが、Xeon Silver 4208(Cascade Lake, 8-Core/16-Thread,Base 2.1GHz Boost 3.2GHz, 85W)より低い消費電力とピーク周波数を期待しているそう

IPC: {
	Haswell <= CNS <= Skylake
	?
}

-->

#### Ncore
学習（トレーニング）は行わず、推論専用のアクセラレータ。  
Ncore部のダイサイズはTSMC 16FFCプロセスで34.4mm<sup>2</sup>、  
x86 CPU 8-Coreクラスターのおおよそ半分のサイズであり、Ncoreの約2/3は16MB SRAMが占める。  

ダイショットを見ると、中央部分のComputer unitが16スライスに分割されていることがわかり、これによって設計をシンプルにしているとのこと。  
緑色の部分はData Unit内の密集した金属配線を示し、この配線は主にデータの再配置するため。  
そして16スライスの中央にある横長の部分に、Instruction UnitとRing Interfaceが含まれる。  

#### Security (Spectre, Meltdown)
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
