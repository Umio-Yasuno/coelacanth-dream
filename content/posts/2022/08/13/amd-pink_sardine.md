---
title: "AMD Pink Sardine プラットフォームをサポートするパッチが Linux Kernel に投稿される"
date: 2022-08-13T08:38:14+09:00
draft: false
categories: [ "Hardware", "APU", "AMD" ]
tags: [ "Linux_Kernel", "Codename", "Phoenix" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Syed Saba kareem 氏より、Linux Kernel ASoC ドライバーに *Pink Sardine* プラットフォームと その ACP (Audio CoProcessor) のサポートを追加するパッチが alsa-devel メーリングリストに投稿されている。  
*Pink Sardine* は ACP v6.2 を採用し、これは先にパッチが投稿された *AMD RPL* プラットフォームと同じ IPバージョンとなる。  
また *Pink Sardine* ACP の DeviceID は `0x15E2` となっており、これは *Raven, Renoir, VanGogh, Yellow Carp (Rembrandt), RPL* ACP と同じ DeviceID。AMD ACP は *Raven* から DeviceID を `0x15E2` で統一し、PCI RevisionID で区別する方法を取っている。  

 * [[PATCH 00/13] Add Pink Sardine platform ASoC driver - Syed Saba kareem](https://lore.kernel.org/alsa-devel/20220812120731.788052-1-Syed.SabaKareem@amd.com/)

## Pink Sardine {#ps}
Linux Kernel では以前、`k10temp` ドライバーと `AMD PMC (Power Management Controller)` ドライバーに *Family 19h Model 70h-7Fh* のサポートを追加するパッチが投稿されていた。  
そこでは *Family 19h Model 70h-7Fh* に `AMD_CPU_ID_PS` が関連付けられており、`PS` が何の略かは明かされていなかったが、今回投稿された *Pink Sardine* がそこに当てはまると思われる。  
{{< link >}} [Linux Kernel で Family 19h Model 60h (CB), Family 19h Model 70h (PS) のサポートが進む | Coelacanth's Dream](/posts/2022/07/10/fam19h-60h-70h/) {{< /link >}}

今回のパッチには `AMD PHEONIX PDM Driver` の記述があり、`PHEONIX` は *Phoenix* の typo ではないかと思われる。  

 > 		+MODULE_DESCRIPTION("AMD PHEONIX PDM Driver");
 > 		+MODULE_LICENSE("GPL v2");
 > 		+MODULE_ALIAS("platform:" DRV_NAME);
 >
 > {{< quote >}} [[PATCH 05/13] ASoC: amd: add acp6.2 pdm platform driver - Syed Saba kareem](https://lore.kernel.org/alsa-devel/20220812120731.788052-6-Syed.SabaKareem@amd.com/) {{< /quote >}}

*Phoenix* は *AMD RDNA 3/GFX11 APU*、*GC 11.0.1 (gfx1103)* に付けられたコードネーム。  
プラットフォームと APU に対するコードネームという違いはあるのかもしれないが、実質 *Pink Sardine = Phoenix APU* だと考えられる。  
*Green Sardine* と *Cezanne APU*、*Yellow Carp* と *Rembrandt APU* と同じ関係性だろう。  

AMDGPUドライバーでは IPバージョンベースのサポートに移行したことで新たな APU/GPU においてコードネームはほとんど使われなくなり、Mesa3D も *色 + 魚* のコードネームから *Navi21* のようなコードネームか GPUID を用いるようになったが、その他 Linux Kernel 内のドライバーでは引き続き *色 + 魚* を用いるようだ。  
*AMD RPL* という例外がすでに存在してはいるが、*Family 19h Model 60h-6Fh* に関連付けられた `CB (AMD_CPU_ID_CB)` の略はまだ不明であり、*RPL* とは別のコードネームがあると思われる。  

ACP をサポートするドライバーには SOF (Sound Open Firmware) プロジェクトによるものもあるが、SOF では *Yellow Carp* のような *色 + 魚* のコードネームではなく、*Rembrandt* を用いている。  
また、最近では *Sabrina* のコードネームで Coreboot のサポートが進められていた APU が、実際は *Mendocino APU* であり、コードネームも *Mendocino* にすべて置き換えられるということがあった。[^sbr-mdn]  
そうした件から見るに、正式発表を前に内部的なコードネームを公開できない事情があるのだろう。  

[^sof]: [Add Support for Rembrandt platform by reddysujith · Pull Request #3749 · thesofproject/linux](https://github.com/thesofproject/linux/pull/3749)
[^sbr-mdn]: [AMD Sabrina SoC の名が Mendocino SoC に置き換えられる | Coelacanth's Dream](/posts/2022/07/15/amd-sabrina-mendocino-soc/)
