---
title: "AMD の USB4 ホストルーターをサポートするパッチが投稿される"
date: 2021-08-12T01:48:07+09:00
draft: false
tags: [ "Linux_Kernel", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD" ]
noindex: false
# summary: ""
---

Linux-USB メーリングリストに AMD の USB4 ホストルーター (ホストコントローラー) のサポートを追加するパッチが投稿された。  
USB4 は Thunderbolt 3 のプロトコルをベースに実装されており、これまでコントローラーを開発し、CPU に統合してきたのは Intel が主だったため、AMD の CPU/SoC に USB4 が将来的に統合されることは USB4 のシステムとしてもユーザーとしても大きなトピックだと言える。  

 * [[PATCH 0/4] Add support for AMD USB4 and bug fixes](https://lore.kernel.org/linux-usb/YQgK9fQoI35P0yLf@lahna/T/#md7c47c816215c73b2d7e08e58b9532d5c5c4c9d0)

パッチでは AMD USB4 HIA (Host Interface Adapter?) の DeviceID (PCI ID) を追加し、いくつかのバグを修正をするものとなっている。  
USB4 については今回のパッチを機に調べたことしか知らないが、ホストルーターという表現が使われているのは、USB4 では PCIeトンネリングや DisplayPortトンネリングもサポートしているからだろうか？  


 > 		+	/* AMD USB4 host */
 > 		+	{ PCI_VDEVICE(AMD, PCI_DEVICE_ID_AMD_USB4_HIA0) },
 > 		+	{ PCI_VDEVICE(AMD, PCI_DEVICE_ID_AMD_USB4_HIA1) },
 > 		+
 >
 > {{< quote >}} [[PATCH 0/4] Add support for AMD USB4 and bug fixes](https://lore.kernel.org/linux-usb/YQgK9fQoI35P0yLf@lahna/T/#m6374365dcf16ecc112db3a29e4e8d64cc19abba5) {{< /quote >}}

 > 		+#define PCI_DEVICE_ID_AMD_USB4_HIA0	0x162e
 > 		+#define PCI_DEVICE_ID_AMD_USB4_HIA1	0x162f
 >
 > {{< quote >}} [[PATCH 0/4] Add support for AMD USB4 and bug fixes](https://lore.kernel.org/linux-usb/YQgK9fQoI35P0yLf@lahna/T/#m6374365dcf16ecc112db3a29e4e8d64cc19abba5) {{< /quote >}}

また、今回の AMD USB4 のサポートを追加するパッチには、、Intel のソフトウェアエンジニアであり、Linux Kernel における Thunderboltドライバーのメンテナを担当する **Mika Westerberg** 氏も好意的なコメントを寄せている。氏は他にも Intel の各種ハードウェアのドライバーを開発している。  

 > 		Hi Sanjay,
 > 		
 > 		On Mon, Aug 02, 2021 at 07:58:16AM -0500, Sanjay R Mehta wrote:
 > 		> From: Sanjay R Mehta <sanju.mehta@amd.com>
 > 		> 
 > 		> This series adds support for AMD USB4 host router and
 > 		> some general USB4 bug fixes.
 > 		
 > 		Nice to see AMD support being added! :) I have few comments on the
 > 		series. I'll comment on the separate patches.
 > 		
 > 		In general looks already good.
 >
 > {{< quote >}} [[PATCH 0/4] Add support for AMD USB4 and bug fixes](https://lore.kernel.org/linux-usb/YQgK9fQoI35P0yLf@lahna/T/#m39a3f8b4c04569832cef86d14d72a2872a02158c) {{< /quote >}}


{{< ref >}}
 * [USB4 and Thunderbolt — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/thunderbolt.html)
 * [D1T1-3 - USB4 System Overview.pdf](https://www.usb.org/sites/default/files/D1T1-3%20-%20USB4%20System%20Overview.pdf)
{{< /ref >}}
