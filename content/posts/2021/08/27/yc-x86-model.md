---
title: "Yellow Carp は Family 19h Model 40h-4Fh"
date: 2021-08-27T18:48:33+09:00
draft: false
tags: [ "Linux_Kernel", "Yellow_Carp", "Green_Sardine", "Cezanne", "Rembrandt", "Lucienne" ]
# keywords: [ "", ]
categories: [ "AMD", "APU" ]
noindex: false
# summary: ""
---

Linux Kernel におけるハードウェアモニタリングドライバー `hwmon`。それに最近の AMD APU のサポートをいくつか追加するパッチが投稿された。  

 * [[PATCH 0/6] Add k10temp support for more client APUs](https://lore.kernel.org/linux-hwmon/20210826184057.26428-1-mario.limonciello@amd.com/T/#mebcb7f6f73b8d06e0f51867e457ce504bcccd877)
 * [[PATCH v2 0/3] Extend k10temp support for more APUs](https://lore.kernel.org/linux-hwmon/20210827201527.24454-1-mario.limonciello@amd.com/T/#m4ddb9c2a80ac5eb91a0a7e18a46b0d98cf73ed89)

具体的には、*Lucienne (Zen 2, Model: 0x68)* 、*Green Sardine (Cezanne, Zen 3, Model: 0x50)* 、*Yellow Carp (Rembrandt, Zen 3, 0x40)* のサポートを追加するものとなる。  

 > 		 		case 0x0 ... 0x1:	/* Zen3 SP3/TR */
 > 		 		case 0x21:		/* Zen3 Ryzen Desktop */
 > 		 		case 0x50 ... 0x5f:	/* Green Sardine */
 > 		+			data->ccd_offset = 0x154;
 > 		 			k10temp_get_ccd_support(pdev, data, 8);
 > 		 			break;
 > 		 		}
 >
 > {{< quote >}} [[PATCH v2 1/3] hwmon: (k10temp): Rework the temperature offset calculation - Mario Limonciello](https://lore.kernel.org/linux-hwmon/20210827201527.24454-2-mario.limonciello@amd.com/) {{< /quote >}}
 >
 > 		+		case 0x40 ... 0x4f:	/* Yellow Carp */
 > 		+			data->ccd_offset = 0x300;
 > 		+			k10temp_get_ccd_support(pdev, data, 8);
 > 		+			break;
 >
 > {{< quote >}} [[PATCH v2 2/3] hwmon: (k10temp): Add support for yellow carp - Mario Limonciello](https://lore.kernel.org/linux-hwmon/20210827201527.24454-3-mario.limonciello@amd.com/) {{< /quote >}}

今回のパッチから読み取れる点として、*Yellow Carp* は *Zen 3 アーキテクチャ* を採用する AMD CPU/APU に割り当てられているものと同じ `Family: 0x19 (25)` であり、ハードウェア側のモニタリング機能も他 *Zen 3* 系プロセッサと同じ動作となるがデータのオフセットが異なるということ。  
そして、AMD GPUドライバーだけでなく、CPU周りに対するドライバーでも *色+魚* のコードネームが用いられるということだ。  

*Green Sardine* は *Cezanne* 、*Yellow Carp* は *Rembrandt* と別のコードネーム、そちらの方が知られているだろう画家系のコードネームと対応している。  
{{< link >}} [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/) {{< /link >}}
{{< link >}} [VanGogh, Beige Goby/Navi24, Yellow Carp/Rembrandt, Cyan Skilfish の GPUID | Coelacanth's Dream](/posts/2021/07/30/vgh-navi24-rmb-gpuid/) {{< /link >}}
[AMDVLK ドライバー](/tags/gpuopen) や [ROCm](/tags/rocm) では画家系のコードネームが使われているが、Linux Kernel 周りでは *色+魚* で統一していくようだ。  
ある APU/GPU を示すコードネームが複数出現するとなると、ややこしいとも思うが、正直そういった情報を収集し、整理していくのは結構好きだ。  

