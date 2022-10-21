---
title: "コードネームあれこれ ―― Phoenix/2, Plum Bonito, Wheat Nas, Hotpink Bonefish, Intel PTP"
date: 2022-10-01T06:22:40+09:00
draft: true
categories: [ "Hardware", "AMD", "Intel", "CPU", "GPU", "CPU" ]
tags: [ "Phoenix", "GFX11", "Codename" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

OSS 内のコードやコメント内で公開されている AMD APU/GPU、Intel CPU のコードネームが少し貯まってきたからメモ書き。  
だから面白そうな話もない。  

## Phoenix/Phoenix2 {#phx}
*AMD Pink Sardine APU* の USB4 サポートに関連して、You-Sheng Yang 氏が以下のようなコメントをしている。  

 >         [Other Info]
 >         
 >         While this is for AMD Phoenix/Phoenix2 platforms, only oem-6.0 and newer
 >         are nominated for fix.
 >
 > {{< quote >}} [[PATCH 0/2][SRU][OEM-6.0/Unstable] Fix USB4 PCIe hotplug on AMD Pink Sardine](https://lists.ubuntu.com/archives/kernel-team/2022-September/133485.html) {{< /quote >}}

*Pink Sardine* の名が初めて出てきた時は typo が含まれており、その後の更新された v2 パッチでは修正されたため不確かな部分があったが、やはり *Pink Sardine* は *Phoniex (Point) APU* の別のコードネームなのだろう。[^first-ps]  
また *Phoenix2 platform* にも触れられているが、まだ名前が出てきただけであり、AMD が公式に発表しているロードマップにもそれらしいものは確認できない。  
*Raven* に対する *Raven2 (Dali, Pollock)* のように規模を縮小した APU なのか、それとも *Rembrandt/Yellow Carp* に FP7 と FP7r2 の 2種類のパッケージがあるように、同じシリコンで違うパッケージであることを示すものなのか。  

[^first-ps]: [AMD Pink Sardine プラットフォームをサポートするパッチが Linux Kernel に投稿される | Coelacanth's Dream](/posts/2022/08/13/amd-pink_sardine/)

## Plum Bonito, Wheat Nas, Hotpink Bonefish {#gfx11}
ROCm v5.3.0 に向けて各種開発者向けツール、ROCm ソフトウェアのレポジトリが一気に更新され、*RDNA 3/GFX11* に関するコードが公開されている。  

今回公開されたコミットの中に *RDNA 3/GFX11* のコードネームらしきものがあった。  

 >         +
 >         +FILTER[plum_bonito]=\
 >         +"$BLACKLIST_ALL_ASICS:"\
 >         +"$BLACKLIST_GFX10_NV2X"
 >         +
 >         +FILTER[wheat_nas]=\
 >         +"$BLACKLIST_ALL_ASICS:"\
 >         +"$BLACKLIST_GFX10_NV2X"
 >         +
 >         +FILTER[hotpink_bonefish]=\
 >         +"$BLACKLIST_ALL_ASICS:"\
 >         +"$BLACKLIST_GFX10_NV2X"
 >         
 > {{< quote >}} [kfdtest: Add gfx11 to kfdtest.exclude · RadeonOpenCompute/ROCT-Thunk-Interface@86a11e5](https://github.com/RadeonOpenCompute/ROCT-Thunk-Interface/commit/86a11e53ddcbe67d9c1efab2a724605f1e0a3db1) {{< /quote >}}

上記引用部は後のコミットにて、*Plum Bonito* は *Navi31/gfx1100* に、*Hotpink Bonefish* は *Navi33/gfx1102* に置き換えられている。  

 >         -FILTER[plum_bonito]=\
 >         +FILTER[gfx1100]=\
 >          "$BLACKLIST_ALL_ASICS:"\
 >          "$BLACKLIST_GFX10_NV2X:"\
 >          "$TEMPORARY_BLACKLIST_GFX11"
 >          
 >         +# TODO: Update to gfx version when kernel moves to staging-drm-next
 >          FILTER[wheat_nas]=\
 >          "$BLACKLIST_ALL_ASICS:"\
 >          "$BLACKLIST_GFX10_NV2X:"\
 >          "$TEMPORARY_BLACKLIST_GFX11"
 >          
 >         -FILTER[hotpink_bonefish]=\
 >         +FILTER[gfx1102]=\
 >          "$BLACKLIST_ALL_ASICS:"\
 >          "$BLACKLIST_GFX10_NV2X:"\
 >          "$TEMPORARY_BLACKLIST_GFX11"
 >
 > {{< quote >}} [kfdtest: Update gpuName logic for ip discovery · RadeonOpenCompute/ROCT-Thunk-Interface@378fef1](https://github.com/RadeonOpenCompute/ROCT-Thunk-Interface/commit/378fef192d3b58e24c9148eaea06e7546825b829) {{< /quote >}}

Linux Kernel における AMDGPU ドライバーでは IPバージョンベースのサポートに移行し、コードネームの代わりに *GC (Graphics, Compute) IP* 部のバージョンを使った名前が使われるようになったが、AMD 内部では今も 色+魚 のコードネームを採用しているのだろう。  
しかし Mesa3D でも、おそらく分かりやすさを重視して、色+魚のコードネームから *Navi2x* や *Rembrandt* といった名前や、GFX ID (gfxXXXX) を基にした名前を用いるようになっているため、*RDNA 3/GFX11* 世代で色+魚のコードネームが使われる機会は少ないと見られる。  

## Intel PTP {#ptp}
Intel の Sasha Neftin 氏より、intel-wired-lan メーリングリストに次世代 Intel クライアントプラットフォームの LOM (LAN-On-Motherboard)、Ethernet コントローラのサポートを追加するパッチが投稿された。  
過去には同様のパッチの中で、*Meteor Lake* や *Lunar Lake* といったコードネームが公開されている。[^wired-lan]  
さすがにコードネームがそのまま公開されることはなくなったが、コードネームの略称は DeviceID の定数名に使われ続けている。  

[^wired-lan]: [Intel Meteor Lake の次世代は「Lunar Lake」、Ethernetコントローラーの初期サポートパッチが投稿される | Coelacanth's Dream](/posts/2021/03/07/intel-lunar_lake/)

 * [[Intel-wired-lan] [PATCH v1 1/1] e1000e: Add support for the next LOM generation - Sasha Neftin](https://lore.kernel.org/intel-wired-lan/20220929080859.2912497-1-sasha.neftin@intel.com/)

 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_ARL_I219_LM24), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_ARL_I219_V24), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_PTP_I219_LM25), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_PTP_I219_V25), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_PTP_I219_LM26), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_PTP_I219_V26), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_PTP_I219_LM27), board_pch_mtp },
 >         +	{ PCI_VDEVICE(INTEL, E1000_DEV_ID_PCH_PTP_I219_V27), board_pch_mtp },
 >
 > {{< quote >}} [[Intel-wired-lan] [PATCH v1 1/1] e1000e: Add support for the next LOM generation - Sasha Neftin](https://lore.kernel.org/intel-wired-lan/20220929080859.2912497-1-sasha.neftin@intel.com/) {{< /quote >}}

*ARL* は Intel Investor Meeting 2022 で発表されたクライアント向け CPU ロードマップにある *Arrow Lake* の略だと思われる。[^intel-investor]  
しかし *PTP* に関しては当てはまりそうなものがない。最後の P はチップセットに対して使われる Point かもしれないが、それでも *PT* は謎だ。  

[^intel-investor]: [Intel Technology Roadmaps and Milestones](https://www.intel.com/content/www/us/en/newsroom/news/intel-technology-roadmaps-milestones.html)

{{< ref >}}
 * [[PATCH 0/2] Fix some hotplug event issues - Mario Limonciello](https://lore.kernel.org/linux-usb/20220921145434.21659-1-mario.limonciello@amd.com/)
 * [[PATCH v2 00/13] Add Pink Sardine platform ASoC driver - Syed Saba Kareem](https://lore.kernel.org/alsa-devel/20220827165657.2343818-1-Syed.SabaKareem@amd.com/)
 * [Announcing New Model Numbers for 2023+ Mobile Proc... - AMD Community](https://community.amd.com/t5/corporate/announcing-new-model-numbers-for-2023-mobile-processors/ba-p/543985)
{{< /ref >}}
