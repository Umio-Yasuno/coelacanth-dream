---
title: "Intel、Lakefield の GPU Programmer's Reference Manuals を公開"
date: 2021-06-19T23:40:30+09:00
draft: false
tags: [ "Lakefield", "Gen11" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

Intel は GPUのオープンソースドライバーに関係したリソースを公開する [Intel® Graphics for Linux\* | 01.org](https://01.org/linuxgraphics) にて、 *「 Intel® UHD Graphics Open Source Programmer's Reference Manual for the 2020 Intel Core™ Processors with Intel Hybrid Technology based on the "Lakefield" Platform 」* を新たに公開した。  

 * [2020 Intel(r) Processors with Hybrid Technology based on the Lakefield platform | 01.org](https://01.org/node/37196)

*Lakefield* は CPU に *Sunny Cove アーキテクチャ (Core/Big)* と *Tremont アーキテクチャ (Atom/Small)* を両採用したハイブリットコア構成かつ、I/O部をまとめた Baseダイ (14nm) と CPU/GPU やメモリコントローラーを搭載した Computeダイ (10nm) を 3D積層技術 *Foveros* を用いる新技術を盛り込んだ省電力な SoC。  
2020/06/10 に 2種類の *Lakefield* SKU が正式発表されている。  
{{< link >}} [Intel、Lakefiled を正式発表 & AVX命令には非対応？ | Coelacanth's Dream](/posts/2020/06/13/intel-lakefiled-without-avx/) {{< /link >}}


## Gen11 GPU

 * [Volume 4: Configurations - intel-gfx-prm-osrc-lkf-vol04-configurations.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-lkf-vol04-configurations.pdf)

*Lakefield* は GPU に、 [Ice Lake (client)](/tags/ice_lake) や [Elkhart/Jasper Lake](/tags/jasper_lake) 同様 [Gen11 アーキテクチャ](/tags/gen11) を採用している。  
GPU の規模は最大 64EU (Subslice 8基 x EU 8基) と、*Ice Lake (client)* と同規模だが、GPU L3キャッシュは *Lakefiled* が 2560KB、*Ice Lake (client)* が 3072KB と異なり、  
またメディアサンプラーが *Lakefield* は搭載していないため、メディアエンジンを用いたデノイズ、デインターレースには対応していないと思われる。 *Elkhart/Jasper Lake* もそれら機能には制限があり、メディアサンプラーの有無は *Lakefield* と *Ellhart/Jasper Lake* で共通しているのかもしれない。  

*Lakefield* の GPU のメモリアドレス範囲は *Ice Lake (client)* より若干狭いが、これは *Lakefield* が LPDDR4 64-bit をメモリインターフェイスに対応し、最大容量は 8GB までと、*Ice Lake (client)* より小さいことが関係しているのだろう。  


### Display Engine 11.5

 * [Volume 12: Display Engine - intel-gfx-prm-osrc-lkf-vol12-displayengine.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-lkf-vol12-displayengine.pdf)

GPU部の世代は *Gen11 アーキテクチャ* であり他と同世代だったが、ディスプレイ部、Display Core/Engine は Gen11.5 とわずかに進んだ世代となっている。  
ただ資料や機能を見る限りでは、どこか機能を強化、改良したというより、画面出力数に対応する Display Pipe が Gen11 では 3 Pipes だったのを Gen11.5 では 4 Pipes に増やし、内部ディスプレイ向けの出力を増やしたことが大きな違いであるように思う。  

また、*Lakefield* では I/O部の多くが 14nmプロセスで製造される Baseダイに集積しているが、GPU や Display Core/Engine は 10nmプロセスの Computeダイに集積している関係で、内部ディスプレイへの映像出力は Computeダイから行われるが、外部ディスプレイはダイを経由し、Baseダイの USB Type-C から出力する方式となっているらしい。  
仕様上では 5K (5120x3200) や 8K (7680x4320) 60Hz の解像度に対応しながら、 *Lakefield* の対応最大解像度が 4K (4096x2304) に制限しているのはこの辺りの方式が影響しているのだろうか？  

*Lakefield* GPU の PRM をこのタイミングが公開した理由については、SKU の正式発表からちょうど約 1年後ということから元々非公開期限を決めていたのかもしれない。  
*DG1/Iris Xe Max* の PRM は 2021/02 に公開されているため、*Lakefield* がここまで遅れた理由は謎だが。  
{{< link >}} [Intel、DG1/Iris Xe Max PRM を公開 | Coelacanth's Dream](/posts/2021/02/20/intel-dg1-prm/) {{< /link >}}
ついでに言えば、ark.intel.com で公開されている *Lakefield* SKU の仕様ページには、まだ Technical Documentation にリンクされた資料が無い。[^i5-l16g7]
*Lakefield* は Microsoft が開発する 2in1デバイス Surface Neo に搭載される予定だったが、Surface Neo の発売は当初 2020年末に予定していたのを 2021年以降に延期[^surface-neo]、さらには Surface Neo に採用する予定だった Windows 10X OS も開発中止が正式に発表される等、逆風が吹き続いている。[^windows10x]  
もしかしたら結果的に実験的なプロセッサになってしまったことが資料があまり公開されない理由なのだろうか。  
キャンセルされた [Cannon Lake](/tags/cannon_lake) も公開されている資料は少なく、単に必要性という点で資料を公開するかしないかを決めている、ということも考えられるが。  

[^windows10x]: [大型アップデート「Windows 10 21H1」提供開始。Windows 10Xは開発中止 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1325442.html)
[^surface-neo]: [Microsoft、2画面のSurface Neo発売を2021年以降に延期 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1250856.html)
[^i5-l16g7]: [Intel® Core™ i5-L16G7 Processor (4M Cache, up to 3.0GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/202777/intel-core-i5-l16g7-processor-4m-cache-up-to-3-0ghz.html#tab-blade-1-0)
