---
title: "Intel PCH Matome (Coreboot編)"
date: 2019-12-04T00:19:21+09:00
draft: false
tags: [ "PCH" ]
keywords: [ "Intel", "PCH" ]
categories: [ "Hardware", "Chipset", "Intel" ]
---

[Coreboot](https://github.com/coreboot/coreboot)で公開されている情報を元に、一部PCHのコードネームをマーケティングにおける名前と照らし合わせてみた。  
括弧内がアルファベット順となっていないが、コンシューマー向けのスペック順とした方がわかりやすいと思い、そうした。  
コンシューマー向けにおいて基本、スペック順にH、B、Zが使われ、  
Qは業務、組み込み向け、  
Cはワークステーション向けとなる。  
Mが付くのはモバイル向け。  
CMP2ではWが追加されているが、組み込み向けらしい、という話が出ている。  

<https://github.com/coreboot/coreboot/blob/master/src/include/device/pci_ids.h#L2682>  

| Intel PCH | a.k.a. | Marketing Name |
| :--- | :---: | :---: |
| **SPT** | SPT_H |  H110, (B/Q)150, (H/HM/Z/Q/QM)170, (HM/QM)175, C232, (C/CM)236, CM238 |
| **KBP** | KBP_H | (B/Q)250, (H/Z/Q)270 |
| **CNP** | CNP_H | H310, B360, (H/HM/Q/QM)370, Z390, C242, (C/CM)246 |
| **CMP2** | CMP_H | (H/HM/Q)470, (W/QM)480, (Z/WM)490 |

Z370がないが、中身としてはZ270なので省かれたと思われる。  
[第8世代Coreプロセッサの本当のコードネームはどれ? - PC Watch](https://pc.watch.impress.co.jp/docs/column/ubiq/1076326.html)

 > ただし、このZ370は、実際には現在Kaby Lake-S用として提供されているIntel 200シリーズチップセットのダイを応用したものとなる。


#### きっかけ
<https://twitter.com/momomo_us/status/1201847124280762368>  
Corebootへのコミットで公式にComet LakeのSKUが明かされた、というツイート。  
恥ずかしながら、それまでCorebootがハードウェア情報の鉱脈であることを知らなかった。  
GPUに続いて、Intelのオープンな姿勢には驚嘆の念を抱く。  

#### おまけ
Tiger LakeにGT3の名が。  
<https://github.com/coreboot/coreboot/blob/master/src/include/device/pci_ids.h#L3352>  
他のファイルではただのGT3ではなく、eDRAM付きであることを表す3eとなっている。  
[tigerlake/bootblock/report_platform.c](https://github.com/coreboot/coreboot/blob/master/src/soc/intel/tigerlake/bootblock/report_platform.c#L48)  

[media_sysinfo_g12.cpp - intel/media-driver](https://github.com/intel/media-driver/blob/master/media_driver/linux/gen12/ddi/media_sysinfo_g12.cpp)やMesaではGT2(96EU)までしか確認できないが、それを上回るSKUを予定してるのだろうか。  

