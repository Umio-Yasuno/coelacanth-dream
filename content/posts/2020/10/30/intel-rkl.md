---
title: "Intel Rocket Lake-S の概要を発表　―― Cypress Cove の詳細はまだ、AV1 HWエンコードに非対応"
date: 2020-10-30T10:53:31+09:00
draft: false
tags: [ "Rocket_Lake", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

Intel は現地時間 2020/10/29 付で、次世代デスクトップ向けプロセッサ *Rocket Lake-S* の概要を発表した。  

 * [Intel’s 11th Gen Processor (Rocket Lake-S) Architecture Detailed | Intel Newsroom](https://newsroom.intel.com/news/intels-11th-gen-processor-rocket-lake-s-architecture-detailed/)
 * [Intel-Rocket-Lake-S-Architecture.pdf](https://newsroom.intel.com/wp-content/uploads/sites/11/2020/10/Intel-Rocket-Lake-S-Architecture.pdf)

CPUアーキテクチャは *Cypress Cove* と称された *Ice Lake (Client)* の機能を備えたものになり、二桁台 (10%〜) の IPC 向上を果たしている。  
AVX-512 命令を用いた **Intel Deep Leaning Boost / VNNI** に対応し、HEDT向けの *Skylake-SP* を除けば、デスクトップ向けでは初の AVX-512 に対応したプロセッサとなる。  
GPUアーキテクチャは *Tiger Lake* と同じ *Gen12アーキテクチャ* となり、*Gen9アーキテクチャ* と比較して 50% 高速だとしている。  

I/O周りも強化され、PCIe Gen4に対応し、レーン数は GPU (x16) + SSD (x4) 直結の構成が可能となる 20レーンを備える。  
メモリは DDR4-3200 に対応する。  

対応チップセットには 500シリーズが新たに投入される。  

テストに使われた *Rocket Lake-S* は 8-Core/16-Thread、動作クロックは不明で、PL (Power Limit) の仕様は **i9-10900K (Comet Lake-S)** に合わされており、PL1: 125W、PL2: 250W、Tau 56s というもの。  

## 詳細ではなく概要

今回 *Rocket Lake-S* の明かされなかった点は、まず製造プロセスが何nmであるか (14nmか 10nmか)、*Ice Lake (Client)* と同機能としておきながら *Sunny Cove* ではなく新たに *Cypress Cove* と称すのは何故か。  

これまでの情報で、*Rocket Lake-S* のキャッシュ構成が *Ice Lake (Client)* と同じことが判明しているため[^rkl-cache]、実行ユニット周りに *Sunny Cove* から変更が施されているのではないかと思われるが、これにはややこしい話がある。  

*Sunny Cove* は *Ice Lake (Client), Ice Lake (Server)* に採用されているが、両者のコアアーキテクチャは全く同じではなく、AVX-512 命令を処理するユニットに細かな違いが存在する。  
*Ice Lake (Client)* は AVX-512 を、対応した実行ポート 1本で処理するが、*Ice Lake (Server)* はそれに加え、他 2本の実行ポートを束ねて実行することが可能となっている。[^sunnycove-avx512]  
*Lakefield* にも *Sunny Cove* コアは搭載されているが、スケジューリングの問題か、AVX 系命令の対応は無効化されている。  
また、*Ice Lake (Server)* では *Ice Lake (Client)* よりも Re-order Buffer が若干増量されているという話もある。  

[^sunnycove-avx512]: [Intel Rocket Lake は AVX-512 をサポート & 推測 | Coelacanth's Dream](/posts/2020/07/23/intel-rocket_lake-support-avx512/)

そのため、どこまでが *Sunny Cove* で、どこから *Crypress Cove* であるか、というのは曖昧な話である。  
*Ice Lake (Client/Server), Lakefield* は全て Intel 10(+)nm プロセスで製造されているため、  
単に Intel 10nmプロセスで製造されれば *Sunny Cove* で、14nmプロセスならば *Crypress Cove* 、といったマーケティング的な意味合いも考えられる。  
そうなると *Rocket Lake-S* は 14nmで製造されているとなるが、それを明言しなかったあたり、Intel 的にはまだ新しい部分だけを推していきたいのかもしれない。  

[^rkl-cache]: [Intel Rocket Lake のキャッシュ構成は Ice Lake と同様か & 推測 | Coelacanth's Dream](/posts/2020/06/28/intel-rocketlake-cache-guess/)

## AV1 HWエンコードは非対応？

AV1 HWエンコードに関しては、各メディアが資料中から抜粋した画像には 4K60 10b 4:2:0 AV1 でのエンコードに対応するように記述されており[^rkl-av1-enc]、自分も最初資料を見た時はそのようになっていたと記憶している。  
だが後になってから再度資料をダウンロードすると、メディア部の記述が変更されており、デコーダー/エンコーダーに分けられ、エンコーダーに AV1 は無い。  
[intel/media-driver](https://github.com/intel/media-driver) のコードを見ても AV1 Hwエンコードには対応してないように読め[^av1-media-driver]、GPU部が同じ *Tiger Lake* 発表時にも AV1 HWエンコード対応は謳われなかった。  
Hot Chips 32 にて行なわれた **{{< xe class="lp" >}}アーキテクチャ** の発表でも、AV1 HWデコードだけがスライドに記載されている。[^hc32-xe]  

[^rkl-av1-enc]: [Intel、デスクトップ向け第11世代Core「Rocket Lake-S」の詳細を発表 ～CPUはIPCを大幅向上させたCypress Cove、GPUにXe Graphicsを採用 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1286191.html)<br>[ASCII.jp：Rocket Lake-S概要発表！Xe LP＆PCIe 4.0×20、IPCは10％以上GPU性能は50％向上](https://ascii.jp/elem/000/004/032/4032293/)
[^av1-media-driver]: [media-driver/media_libva_caps_g12.cpp at d934d6c48644227ab066d013a105335ad25ce2bc · intel/media-driver](https://github.com/intel/media-driver/blob/d934d6c48644227ab066d013a105335ad25ce2bc/media_driver/linux/gen12/ddi/media_libva_caps_g12.cpp)
[^hc32-xe]: [Hot Chips 2020 Live Blog: Intel's Xe GPU Architecture (5:30pm PT)](https://www.anandtech.com/show/15993/hot-chips-2020-live-blog-intels-xe-gpu-architecture-530pm-pt)

そのため、実際は *Rocket Lake-S* GPU部は AV1 HWエンコードに対応していない、ということに思われるが、Intel は各メディアに通知していないのだろうか？  

