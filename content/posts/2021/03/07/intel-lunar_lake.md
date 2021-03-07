---
title: "Intel Meteor Lake の次世代は Lunar Lake、Ethernetコントローラーの初期サポートパッチが投稿される"
date: 2021-03-07T08:47:46+09:00
draft: false
tags: [ "Meteor_Lake", "Lunar_Lake", "Alder_Lake" ]
# keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU" ]
noindex: false
# summary: ""
toc: false
---

Intel Ethernetデバイス関連の新たなパッチにて、将来のクライアント向け Intel CPU のコードネームが *Lunar Lake* であることが明かされた。  
[Intel-wired-lan](https://lists.osuosl.org/pipermail/intel-wired-lan/) メーリングリストは、Intel Ethernetデバイス関連のパッチが投稿、公開される場である。  

パッチでは、*Lunar Lake* 世代の PCH に統合されている Ethernetコントローラーの名前、DeviceID を追加する初期サポートが行われている。*Lunar* は *LN* と略されるらしい。  

 * [[Intel-wired-lan] [PATCH v1 1/1] e1000e: Add support for Lunar Lake](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20210301/023498.html)

 >         Add devices IDs for the next LOM generations that will be
 >        available on the next Intel Client platform (Lunar Lake)
 >        This patch provides the initial support for these devices
 >
 > {{< quote >}} [[Intel-wired-lan] [PATCH v1 1/1] e1000e: Add support for Lunar Lake](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20210301/023498.html) {{< /quote >}}

2020/08 には *Meteor Lake* の名が明かされ、同様の初期サポートが行われていた。  

 >        Add devices ID's for the next LOM generations that will be
 >        available on the next Intel Client platform (Meteor Lake)
 >        This patch provides the initial support for these devices
 >
 > {{< quote >}} [[Intel-wired-lan] [PATCH v1 1/1] e1000e: Add support for Meteor Lake](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20200810/021025.html) {{< /quote >}}

パッチのコメント自体は *Alder Lake* [^adl]、*Tiger Lake* [^tgl] でも同様で、テンプレートだろう。  
*Alder Lake* のパッチは 2020/01/16 に、*Tiger Lake* は 2019/10/16 に投稿されており、大体 7ヶ月〜9ヶ月ごとに新たな Intel CPU コードネームが公開されている。  

[^adl]: [[Intel-wired-lan] [PATCH v1] e1000e: Add support for Alder Lake](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20200113/018580.html)
[^tgl]: [[Intel-wired-lan] [PATCH v1] e1000e: Add support for Tiger Lake](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20191014/017859.html)

コードネームばかりが先行しているが、次世代クライアント向け CPU が *Alder Lake* 、その次世代が *Meteor Lake* 、さらにその次世代が *Lunar Lake* となるのだろう。  
それらで 各種 I/O、GPU、Coreboot でのサポート等が公開されているのは *Alder Lake* までで、*Meteor Lake* はまだ Ethernetデバイスの初期サポートのみが行われている状況にある。  

*Alder Lake* の情報公開は、自分の過去記事を参照するに、2020/04/01 に公開された 「Intel® Architecture Instruction Set Extensions Programming Reference」(Rev: -O38) にその名が記載され、それから ChromiumOS関連レポジトリや GCC、Linux Kernel に本格的なサポートに向けたパッチが投稿され始めている。  
そして、2020/08 に開催された Intel Architecture Day 2020 にて、コア構成が *Golden Cove (Core)* と *Gracemont (Atom)* のハイブリッドになることが発表された。  

*Alder Lake* の時のペースから言えば、*Meteor Lake* もそろそろ他にパッチが投稿され、詳細が徐々に公開され始める頃ではないかと思われるが、結局の所、公開するタイミングは Intel 次第である。それを待つしかない。  

