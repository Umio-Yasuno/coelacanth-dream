---
title: "AMD Smart Shift への対応を進める Linux AMDGPU ドライバー"
date: 2021-06-07T23:08:33+09:00
draft: false
tags: [ "Linux_Kernel", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "APU", "GPU" ]
noindex: false
# summary: ""
---

**AMD Smart Shift** は **Ryzen 4000 U/H-Series** と **Radeon RX 5000M Series** の発表に合わせて登場した機能であり、ワークロードに応じて APU/dGPU への電力の割り振りを動的に変更させることで、限られた電力枠内で性能を最大限引き出すというもの。  
 
 * [SmartShiftテクノロジー | AMD](https://www.amd.com/ja/technologies/smartshift)

その **Smart Shift** を Linux AMDGPUドライバーにおいて本格的にサポートするためのパッチが、Linux Kernel (amd-gfx) メーリングリストに投稿、公開され始めた。  


{{< pindex >}}
 * [Device/Driver State](#dev_drv)
 * [APU/dGPU の Power Limit](#power_limit)
 * [Smart Shift のバイアスが設定可能に](#bias)
{{< /pindex >}}

## Device/Driver State {#dev_drv}

まず、**Smart Shift** に必要な、dGPU のデバイスステートを管理するパッチから読んでいく。  
以下のパッチは AMD のソフトウェアエンジニア **Sathishkumar S** 氏より作成、投稿されている。  

 * [[PATCH] drm/amdgpu: support atcs method powershift (v4)](https://lists.freedesktop.org/archives/amd-gfx/2021-May/064441.html)
 * [[PATCH] drm/amdgpu: enable smart shift on dGPU (v5)](https://lists.freedesktop.org/archives/amd-gfx/2021-May/064536.html)

パッチ内に出てくる PX, HG といった単語はそれぞれ Power eXpress, Hybrid Graphics の略とされるが[^px-hg]、  
その他の AT, CS については、何の略かはっきりと示してくれるようなドキュメント、パッチを見付けられなかった。CS はまだ Clock State か Chipset Specific というような推測はできるが、AT はさっぱり。  

[^px-hg]: [Recap: AMD’s PowerXpress, aka Dynamic Switchable Graphics, aka Enduro - AMD’s Enduro Switchable Graphics Levels Up](https://www.anandtech.com/show/6243/amds-enduro-switchable-graphics-levels-up/2)

パッチでは APU/System BIOS に dGPU のデバイスステートを通知する `POWER_SHIFT_CONTROL` 機能を追加しており、ステートにはデバイス側は D0 と D3_hot、ドライバー側には LOAD/OPR (Operational) と UNLOAD/NOT_OPR が用意されている。  
D0 ステートは通常の稼働状態を示し、D3 ステートは最も低い消費電力となるステートであり、デバイスは稼働せず待機状態に入る。D3_hot はそのサブステート。  
D3 ステートのサブステートには D3_cold もあり、D3_hot は D0 から直接移行することができるが、D3_cold は D3_hot からしか移行できない。  
また D3_cold は消費電力が D3_hot よりも低くなるが、D0 への復帰に時間が掛かる。  

dGPU が待機状態である D3_hot ステートに入ること自体は、**Smart Shift** 発表時に行われたテクニカルジャーナリスト [西川善司](https://twitter.com/zenjinishikawa) 氏の取材によれば、iGPU と単体 GPU (dGPU) を同時に稼働することは無く、常に一方のみが稼働する状態で **Smart Shift** は機能する、と説明されている。  
グラフィクス処理の負荷が低い時は APU 内の GPU のみが動作し、dGPU は待機状態に入ることで最低限の電力のみを消費する。  

 > ――グラフィックス処理の負荷に応じて，CPUとGPUの間で電力を動的に振り分ける「AMD SmartShift」において，iGPUと単体GPUを同時に稼動させることはできるのか。
 > 
 > Stankard氏：  
 > 　iGPUと単体GPUを同時に動かすことはできない。iGPUと単体GPUのどちらか一方が動いているとき，稼動していないGPUはほぼ完全に動作を停止する。
 >
 > {{< quote >}} [西川善司の3DGE:Ryzen 4000とRadeon RX 5600 XTの気になるところをAMDにアレコレ聞いてみた](https://www.4gamer.net/games/446/G044684/20200115091/) {{< /quote >}}

## APU/dGPU の Power Limit {#power_limit}

**Smart Shift** は APU と dGPU を合計した消費電力枠の中で、割り振りをワークロードに最適化することで性能を引き出す。そのためにはドライバーが APU/dGPU の消費電力枠、Power Limit を管理する必要がある。  
そこで従来は dGPU の Power Limit を VBIOS から取得していたが、APU のも取得するようにするパッチが投稿された。  
パッチは **Darren Powell** 氏より作成、投稿されている。  

 * [[PATCH 2/6] amdgpu/pm: clean up smu_get_power_limit function signature](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064917.html)
 * [[PATCH 3/6] amdgpu/pm: modify Powerplay API get_power_limit to use new pp_power enums](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064918.html)
 * [[PATCH 4/6] amdgpu/pm: modify and add smu_get_power_limit to Powerplay API](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064920.html)
 * [[PATCH 6/6] amdgpu/pm: add kernel documentation for smu_get_power_limit](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064921.html)

CPU と GPU の両方を搭載する APU 全体の Power Limit は PPT (Package Power Tracking) で管理されており、それを SMU (System Management Unit) から取得する。  
PPT には Sustained と Fast、2種類の Power Limit Type があり、それらはブーストクロックの継続時間に関係する。  
前者は長い時間 (~5000ms) 動作するブーストクロックとその電力枠となり、基本 TDP として表記されている値が電力枠に使われる。Fast は短い時間 (~10ms) 動作し、電力を基準としている。  
Intel CPU における PL1 が Sustained、PL2 が Fast に大体相当する。  
Slow と呼ばれる Power Limit Type もあるが、これは Fast が時間か電力の制限に達し、Sustained に移行する際に使われる電力枠で、Sustained と Fast の中間に近い値が設定される。  
{{< link >}} [AMD VanGogh APU では PPT もドライバーから調節可能に | Coelacanth's Dream](/posts/2021/02/03/vgh-ppt/) {{< /link >}}

## Smart Shift のバイアスが設定可能に {#bias}

Linux AMDGPUドライバーへの **Smart Shift** の実装では、APU/dGPU のどちらに電力を優先的に割り振るかが設定可能になる。  

 * [[PATCH] drm/amdgpu: attr to control SS2.0 bias level (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064577.html)

-100..100 の範囲でバイアスが設定可能になっており、-100 ではAPU に、100 では dGPU に最大限優先的に割り振るようになる。デフォルトは 0 でバイアスは掛かっていない状態。  

 > 		+/**
 > 		+ * DOC: smartshift_bias
 > 		+ *
 > 		+ * The amdgpu driver provides a sysfs API for reporting the
 > 		+ * smartshift(SS2.0) bias level. The value ranges from -100 to 100
 > 		+ * and the default is 0. -100 sets maximum preference to APU
 > 		+ * and 100 sets max perference to dGPU.
 > 		+ */
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: attr to control SS2.0 bias level (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064577.html) {{< /quote >}}

ただあくまでも設定できるのはバイアスであり、それで変更されるのは負荷に応じて割り振られる量の度合いと考えられる。CPU に負荷がほとんど無いグラフィクス処理の場合に、dGPU へ割り振られる電力量は、バイアスを -100 に設定しても変わらないだろう。  
CPU/GPU への負荷量が全体としては低いが、負荷に波があり iGPU と dGPU の切り替えが多く発生するようなブラウジング、あるいは複数のアプリケーションを立ち上げている場合等、どちらかが電力枠の制限に達することのない程度のワークロードには有効的に働く設定と思われる。そうした場合には性能、消費電力をバイアスの設定によって実質的にコントロール可能となる。  
だがバイアスの設定が *どれだけ* 有効的に働くは微妙で、BIOS とドライバーの実装に任せた方が良い場合がほとんどかもしれない。  

また、APU/dGPU がもう一方の電力枠をどれだけ用いているかを sysfs から読み取れるようにするパッチも投稿されている。  

 * [[PATCH] drm/amd/pm: sysfs attrs to read ss powershare (v6)](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064575.html)

 > 		+/**
 > 		+ * DOC: smartshift_apu_power
 > 		+ *
 > 		+ * The amdgpu driver provides a sysfs API for reporting APU power
 > 		+ * share if it supports smartshift. The value is expressed as
 > 		+ * the proportion of stapm limit where stapm limit is the total APU
 > 		+ * power limit. The result is in percentage. If APU power is 130% of
 > 		+ * STAPM, then APU is using 30% of the dGPU's headroom.
 > 		+ */
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: sysfs attrs to read ss powershare (v6)](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064575.html) {{< /quote >}}
 >
 > 		+/**
 > 		+ * DOC: smartshift_dgpu_power
 > 		+ *
 > 		+ * The amdgpu driver provides a sysfs API for reporting the dGPU power
 > 		+ * share if the device is in HG and supports smartshift. The value
 > 		+ * is expressed as the proportion of stapm limit where stapm limit
 > 		+ * is the total APU power limit. The value is in percentage. If dGPU
 > 		+ * power is 20% higher than STAPM power(120%), it's using 20% of the
 > 		+ * APU's power headroom.
 > 		+ */
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: sysfs attrs to read ss powershare (v6)](https://lists.freedesktop.org/archives/amd-gfx/2021-June/064575.html) {{< /quote >}}

ドキュメントとしても扱われるコメント部を読むに、APU/dGPU が協調して動作する点に変わりはないが、  
ワークロードに適した電力割り振りを APU/dGPU に行う、というよりはワークロードに応じて余裕のある APU/dGPU がその分を一方が使えるよう分け与える、といった表現のがしっくりくるかもしれない。  

{{< ref >}}
 * [西川善司の3DGE:Ryzen 4000とRadeon RX 5600 XTの気になるところをAMDにアレコレ聞いてみた](htps://www.4gamer.net/games/446/G044684/20200115091/)
 * [Device Power States - Windows drivers | Microsoft Docs](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/device-power-states)
 * [Device Low-Power States - Windows drivers | Microsoft Docs](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/device-sleeping-states)
 * [AR# 3552: LogiCORE PCI - Power Management: Description of Function Power states](https://japan.xilinx.com/support/answers/3552.html)
{{< /ref >}}


