---
title: "AMD の次世代モバイルプラットフォームでは人感センサーをサポート"
date: 2021-06-27T12:02:47+09:00
draft: false
tags: [ "Yellow_Carp", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

AMD APU において *System Management Unit (SMU)* 内部で実行される電力管理機能に対するドライバー *AMD Power Management Controller (amd-pmc)* に、AMD の次世代モバイルプラットフォームに搭載される人感センサー (Human Presence Detection / HPD) のサポートを追加するパッチが投稿された。  
次世代プラットフォームでは Ambient Light Sensor (ALS) も *Renoir* や *Cezanne* といった APU/SoC から拡張されるとし、そのサポートも同時に追加されている。  

 * [[PATCH 0/3] Add SFH sensor support for newer AMD platforms](https://lore.kernel.org/linux-input/20210618081838.4156571-4-Basavaraj.Natikar@amd.com/T/)

 > 		Add Human Presence Detection (HPD) sensors support
 > 		on AMD next generation HPD supported platforms.
 >
 > {{< quote >}} [[PATCH 0/3] Add SFH sensor support for newer AMD platforms](https://lore.kernel.org/linux-input/20210618081838.4156571-4-Basavaraj.Natikar@amd.com/T/#u) {{< /quote >}}

{{< pindex >}}
 * [Yellow Carp が次世代モバイルプラットフォーム?](#yc)
 * [人感センサーの用途](#sensor)
{{< /pindex >}}

## Yellow Carp が次世代モバイルプラットフォーム? {#yc}

次世代プラットフォームとなる APU/SoC については、 *amd-pmc* ドライバーへの別パッチにて **YC** という APU/SoC に搭載される PMC の PCI ID, APIC ID が追加されている。
そして **YC** は [Yellow Carp](/tags/yellow_carp) APU の略称として AMD GPU ドライバーでは用いられており、このことから *Yellow Carp* が次世代モバイルプラットフォームの APU/SoC だと考えられる。  
{{< link >}} [Yellow Carp は VanGogh に続く RDNA 2 APU に | Coelacanth's Dream](/posts/2021/05/19/radeonsi-beige_goby-yellow_carp/) {{< /link >}}

 * [[PATCH v2 0/7] updates to amd-pmc driver](https://lore.kernel.org/platform-driver-x86/20210623200149.2518885-7-Shyam-sundar.S-k@amd.com/T/#m781f645e0c9351ec75f5969d07d30fb843c08dbd)

 > 		--- a/drivers/platform/x86/amd-pmc.c
 > 		+++ b/drivers/platform/x86/amd-pmc.c
 > 		@@ -68,6 +68,7 @@
 > 		 #define AMD_CPU_ID_RN			0x1630
 > 		 #define AMD_CPU_ID_PCO			AMD_CPU_ID_RV
 > 		 #define AMD_CPU_ID_CZN			AMD_CPU_ID_RN
 > 		+#define AMD_CPU_ID_YC			0x14B5
 > 		 
 > 		 #define PMC_MSG_DELAY_MIN_US		100
 > 		 #define RESPONSE_REGISTER_LOOP_MAX	200
 > 		@@ -309,6 +310,7 @@ static int amd_pmc_get_os_hint(struct amd_pmc_dev *dev)
 > 		 	case AMD_CPU_ID_PCO:
 > 		 		return MSG_OS_HINT_PCO;
 > 		 	case AMD_CPU_ID_RN:
 > 		+	case AMD_CPU_ID_YC:
 > 		 		return MSG_OS_HINT_RN;
 > 		 	}
 > 		 	return -EINVAL;
 > 		@@ -354,6 +356,7 @@ static const struct dev_pm_ops amd_pmc_pm_ops = {
 > 		 };
 > 		 
 > 		 static const struct pci_device_id pmc_pci_ids[] = {
 > 		+	{ PCI_DEVICE(PCI_VENDOR_ID_AMD, AMD_CPU_ID_YC) },
 > 		 	{ PCI_DEVICE(PCI_VENDOR_ID_AMD, AMD_CPU_ID_CZN) },
 > 		 	{ PCI_DEVICE(PCI_VENDOR_ID_AMD, AMD_CPU_ID_RN) },
 > 		 	{ PCI_DEVICE(PCI_VENDOR_ID_AMD, AMD_CPU_ID_PCO) },
 > 		@@ -444,6 +447,7 @@ static int amd_pmc_remove(struct platform_device *pdev)
 > 		 static const struct acpi_device_id amd_pmc_acpi_ids[] = {
 > 		 	{"AMDI0005", 0},
 > 		 	{"AMDI0006", 0},
 > 		+	{"AMDI0007", 0},
 > 		 	{"AMD0004", 0},
 > 		 	{ }
 > 		 };
 >
 > {{< quote >}} [[PATCH v2 0/7] updates to amd-pmc driver](https://lore.kernel.org/platform-driver-x86/20210623200149.2518885-7-Shyam-sundar.S-k@amd.com/T/#m781f645e0c9351ec75f5969d07d30fb843c08dbd) {{< /quote >}}

 > 		    case FAMILY_YC:
 > 		      identify_chip(YELLOW_CARP);
 > 		      break;
 >
 > {{< quote >}} [amd: add Yellow Carp support (c54bb135) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/c54bb135aad025cad747dedb75541474505c28ae#c3cf206d71203e77a4252c3915daf913c9251dc3) {{< /quote >}}

*Yellow Carp* は GPU部に *RDNA 2 アーキテクチャ* を採用する APU で、ディスプレイエンジンにはクロック調節機能を前世代からさらに追加した DCN 3.1 を採用している。  
{{< link >}} [RDNA 2 APU 「Yellow Carp」 をサポートするパッチが Linux Kernel に投稿される　―― DCN3.1 / VanGogh より大きいキャッシュ | Coelacanth's Dream](/posts/2021/06/03/yellow_carp-apu-linux-kernel/) {{< /link >}}
*RDNA 2 アーキテクチャ* を採用する APU にはもう一つ [VanGogh](/tags/vangogh) APU がいるが、*VanGogh* は CPU に *Zen 2 アーキテクチャ* 4-Core/8-Thread を搭載する構成であり、8-Core/16-Thread の現モバイルプラットフォーム *Lucienne (Zen 2)* / *Cezanne (Green Sardine, Zen 3)* の後継になるとは考えにくい。  

## 人感センサーの用途 {#sensor}
AMD の次世代モバイルプラットフォームに搭載される人感センサーだが、実は近い技術とチップを Intel が 2020/12 に発表している。  

 * [Video: CES 2021: Intel Visual Sensing Technology (Demo) | Intel Newsroom](https://newsroom.intel.com/video-archive/video-ces-2021-intel-visual-sensing-technology-demo/)
 * [Building the Industry's Best PC Experiences](https://www.intel.com/content/www/us/en/newsroom/opinion/building-industrys-best-pc-experiences.html)

*Intel Visual Sensing Controller (Code Name: Clover Falls)* は AI機能を活用して PCプラットフォームをよりセキュアかつ高機能にするものと説明されており、Intel は例として人感を検出してディスプレイの明るさを自動的に調整する省電力機能を挙げている。ただその処理のどこに、どれだけ AI機能を活用しているかは不明。  
*Intel Visual Sensing Technology* を採用する **Dell Latitude 9420** では、人感を検出するとスリープ状態を解除、顔認識によりログインを行い、ノートPCから離れた場合は PC をロックし、セキュリティを保つ、という機能も紹介されている。[^dell]  

[^dell]: [New Latitude 9420 | Dell 日本](https://www.dell.com/ja-jp/work/shop/2-in-1%E3%83%8E%E3%83%BC%E3%83%88%E3%83%91%E3%82%BD%E3%82%B3%E3%83%B3/latitude-9420-%E3%83%8E%E3%83%BC%E3%83%88%E3%83%91%E3%82%BD%E3%82%B3%E3%83%B32-in-1%E3%82%82%E9%81%B8%E6%8A%9E%E5%8F%AF/spd/latitude-14-9420-2-in-1-laptop)

AMD の次世代モバイルプラットフォームも人感センサーを同様の省電力機能、セキュリティ機能に活用すると思われ、Intel に追従する形となるだろう。  
また、*Clover Falls* かその次世代チップになるかは不明だが、Intel *Alder Lake* プラットフォームでも引き続き *Intel Visual Sensing Controller* をサポートする見込みにある。[^adl-platform]  
ノートPC の消費電力においてディスプレイが占める割合は大きく、こうした省電力機能はバッテリー持続時間の増加に期待できる。  

[^adl-platform]: [Bug #1932376 “[ADL] Intel Visual Sensing Controller” : Bugs : intel](https://bugs.launchpad.net/intel/+bug/1932376)

{{< ref >}}
 * [Intelがコンパニオンチップ「Clover Falls」を発表、詳細は不明 - CIOニュース：CIO Magazine](https://project.nikkeibp.co.jp/idg/atcl/19/00002/00169/)
 * [Pushing the Boundaries of Modern Computers at Computex | Intel Newsroom](https://newsroom.intel.com/editorials/pc-personal-contribution-platform-pushing-boundaries-modern-computers-computex/)
 * [【イベントレポート】Intel、ノートPC向けディスプレイの消費電力を削減する新技術 - PC Watch](https://pc.watch.impress.co.jp/docs/news/event/1125405.html)
{{< /ref >}}
