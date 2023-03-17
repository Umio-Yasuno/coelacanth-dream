---
title: "Intel、Thunder Bay VPU をキャンセル"
date: 2023-03-17T19:56:50+09:00
draft: false
categories: [ "Hardware", "Intel", "VPU", ]
# tags: [ "", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Rashmi A 氏により、Linux Kernel から *Thunder Bay VPU* 固有のコードを削除するパッチを投稿された。パッチ内にて *Thunder Bay VPU* がキャンセルされたことが伝えられている。  

 >         Remove Thunder Bay specific code as the product got cancelled and 
 >         there are no end customers or users.
 >
 > {{< quote >}} <https://lore.kernel.org/lkml/20230316120549.21486-1-rashmi.a@intel.com/> {{< /quote >}}

## Thunder Bay {#thb}
*Thunder Bay VPU* は CPU に Arm Cortex-A53 を採用し、複数の VPUスライスを持つ SoC/VPU になることが過去の Linux Kernel へのパッチにて公開されていた。[^thb]  
現行の Intel VPU 製品とされている *Keem Bay APU* も同様に Arm Cortex-A53 を採用しているが、*Thunder Bay VPU* は Arm Cortex-A53 4-Core の CPUクラスタを最大で 4基、VPUスライスは Full バリアントで 4基、Prime (Standard) で 2基持ち、*Keem Bay VPU* よりも大規模な構成を取っている。  

[^thb]: [Intel Movidius VPU に関するメモ ―― Keem Bay, Thunder Bay, Meteor Lake | Coelacanth's Dream](/posts/2022/01/11/intel-kmb-thb/#thb)

*Thunder Bay VPU* の SKU などが Intel から正式に発表されることはなかったが、2021/11/04 付で公開されたニュースリリースでは、中国の Tencent と協力し、推論性能とメディア性能を活かした交通制御システムが実演され、また Xeon プロセッサと PCIe カードに搭載した *Thunder Bay VPU* を組み合わせたプラットフォームを提供する予定であることが発表されている。[^tencent]  
先のパッチでは、*Thunder Bay VPU* を採用する顧客とユーザーがいないことをキャンセルされた理由としているため、そうしたプロジェクトも立ち消えとなってしまったのだろう。  

[^tencent]: [英特尔携手腾讯以全芯、全栈、全方位合作 共推云数智变革](https://www.intel.cn/content/www/cn/zh/newsroom/news/intel-tencent-cooperation.html)

## Keem Bay {kmb}

*Keem Bay VPU* についても、発表自体は 2019/11/15 の Intel AI Summit だったが、Intel 公式サイトに掲載されている *Keem Bay VPU* の SKU は **3700VC** のみ。発売日は Q1'23 となっている。[^kmb]  
恐らくモバイル向け *Raptor Lake* と組み合わせることを想定した SKU と思われる。  

 * [Gen 3 Intel Movidius 3700VC VPU 製品仕様](https://ark.intel.com/content/www/jp/ja/ark/products/230545/gen-3-intel-movidius-3700vc-vpu.html)

**3400VE** と **3700VE** といった SKU の性能を掲載しているページもあるが、*Gen 3 Intel® Movidius™ VPU (coming soon...)* のまま更新されていない。  

 * [Hardware and Software for Edge Inference Solutions](https://www.intel.com/content/www/us/en/developer/articles/technical/how-to-choose-hardware-and-software-for-edge-inference-solutions.html)

[^kmb]: [Intel Movidius VPU に関するメモ ―― Keem Bay, Thunder Bay, Meteor Lake | Coelacanth's Dream](/posts/2022/01/11/intel-kmb-thb/)

*Thunder Bay VPU* はその規模とニュースリリースの内容から、クラウドサーバー向けの製品として開発されていたことがうかがえるが、最終的にはキャンセルされた。  
一応エッジ推論処理向けの *Keem Bay VPU* は続けられており、またモバイル向け *Raptor Lake* と組み合わせた採用が進められていることが以前に語られている。  
*Meteor Lake* では SoC Tile に VPU が統合され、今後はクライアント向けでの活用が加速してことが考えられる。  
すでに Intel の Github レポジトリでは、*Meteor Lake VPU* 向けの UMD (User Mode Driver) が公開されており、Intel VPU の普及と同時に広範な利用が期待される。  

 * [intel/linux-vpu-driver: Intel® Versatile Processing Unit Driver](https://github.com/intel/linux-vpu-driver)

{{< ref >}}
 * [Introduction - 005 - ID:743844 | 13th Generation Intel® Core™ Processors](https://edc.intel.com/content/www/us/en/design/products/platforms/details/raptor-lake-s/13th-generation-core-processors-datasheet-volume-1-of-2/005/introduction/)
 * [Intel Thunder Bay Is Officially Canceled, Linux Driver Code To Be Removed - Phoronix](https://www.phoronix.com/news/Intel-Thunder-Bay-Cancelled)
 * [Intel、次世代のMeteor LakeにVPUを統合予定。第13世代CoreでLE Audio対応も - PC Watch](https://pc.watch.impress.co.jp/docs/news/1439818.html)
 * [Intel Meteor Lake and Arrow Lake - HotChips 34レポート | マイナビニュース](https://news.mynavi.jp/article/20220824-2433247/)
{{< /ref >}}
