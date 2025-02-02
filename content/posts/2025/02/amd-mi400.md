---
title: "AMD MI400 のチップレット構成?"
date: 2025-02-02T13:06:30+09:00
draft: false
categories: [ "AMD", "GPU", "Hardware" ]
tags: [ "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Shaoyun Liu 氏により、amd-gfx メーリングリストに MES (MicroEngine Scheduler) v12 向けの API ヘッダーを更新するパッチが投稿されたのだが、そこで MI400 のチップレット構成についてのコメントがあった。  

 * [[PATCH] drm/amd/include : Update MES v12 API header according to new MES features](https://lists.freedesktop.org/archives/amd-gfx/2025-February/119450.html)

パッチはおそらくレジスタの書き込む対象となるダイ (チップレット) を指定可能にするためのものと思われる。  
そしてコメントでは MI400 の場合、オプションの指定する値とダイの ID がどのように結び付けられているかを解説している。  
コメントによれば、MI400 は AID (Active Interposer Die) ごとに XCD (Accelerated Compute Die) を 4基搭載し、全体では AID 2基、XCD 8基の構成になるとされている。  
AMD MI300 では AID ごとに XCD 2基、全体では AID 4基、XCD 8基の構成となっており、MI400 では AID が大型化している可能性がある。  
また、AID に搭載される XCD の数が増えたことで、同一の AID に搭載された XCD 間の通信の高速化や、チップレット外に出る通信が減ることによる消費電力の低下といった利点はあるだろう。  
しかし、もしも MI300 と同様に CPU Complex Die (CCD) を XCD 用 AID と別の AID に搭載する形式だとすると、搭載可能な XCD が最大 4基となり、MI300A APU から XCD の数自体は減ることとなる。  

MI400 ではダイの種類に MID (Multimedia IO Die) が追加される。  
名前からしておそらく、MI300 では AID に実装されていたマルチメディアエンジンを別ダイに分割したのだろう。  
他 I/O も MID に移されている可能性はある。  
コメントでは MID を最大 2基としており、AID ごとに MID 1基を搭載する構成と思われる。  

 >         +/*
 >         + * RRMT(Register Remapping Table), allow the firmware to modify the upper address
 >         + * to correctly steer the register transaction to either the local AID/XCD or
 >         + * remote MID on SMN.
 >         + * mode : Mode of operation for RRMT
 >         + *	0=Local XCD
 >         + *	1=Remote/Local AID
 >         + *	2=Remote XCD
 >         + *	3=Remote MID
 >         + * mid_die_id : Physical ID number of the Multimedia IO Die (MID) to be accessed for RRMT.
 >         + *	0=MID0.
 >         + *	1=MID1
 >         + * xcd_die_id :	Virtual ID number of the Accelerated Compute Die (XCD)
 >         + *	to be accessed for RRMT. For MI400, there are 2 Active
 >         + *	Interposer Die (AID) each with 4 XCDs. The number of
 >         + *	available XCDs depends on the Partition Mode programmed
 >         + *	by the Secure Processor
 >         + *	0=XCD0.
 >         + *	1=XCD1.
 >         + *	2=XCD2.
 >         + *	3=XCD3.
 >         + *	4=XCD4.
 >         + *	5=XCD5.
 >         + *	6=XCD6.
 >         + *	7=XCD7.
 >         + *
 >         + */
 >
 > {{< quote >}} [[PATCH] drm/amd/include : Update MES v12 API header according to new MES features](https://lists.freedesktop.org/archives/amd-gfx/2025-February/119450.html) {{< /quote >}}
