---
title: "Green Sardine/Cezanne APU の PSP DeviceID が追加される"
date: 2021-04-17T00:19:54+09:00
draft: false
tags: [ "Green_Sardine", "Cezanne", "VanGogh" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

AMDプロセッサには、メモリ暗号化等の機能を担当する PSP (Platform Security Processor)、あるいは CCP (Cryptographic Coprocessor) と呼ばれる内部デバイスが存在する。  
そして、Linux Kernel に *Green Sardine APU* に搭載された PSP/CCP の DeviceID を追加するパッチが投稿された。  

 * [LKML: Rijo Thomas: [PATCH] ccp: ccp - add support for Green Sardine](https://lkml.org/lkml/2021/4/16/273)

*Green Sardine* は Linux Kernel の AMD GPUドライバーで用いられているコードネームであり、内部 GPU の DeviceID から *Cezanne APU* と同一の APU と考えられる。  
{{< link >}} [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/) {{< /link >}}

 > 		diff --git a/drivers/crypto/ccp/sp-pci.c b/drivers/crypto/ccp/sp-pci.c
 > 		index f471dbaef1fb..f468594ef8af 100644
 > 		--- a/drivers/crypto/ccp/sp-pci.c
 > 		+++ b/drivers/crypto/ccp/sp-pci.c
 > 		@@ -356,6 +356,7 @@ static const struct pci_device_id sp_pci_table[] = {
 > 		 	{ PCI_VDEVICE(AMD, 0x1468), (kernel_ulong_t)&dev_vdata[2] },
 > 		 	{ PCI_VDEVICE(AMD, 0x1486), (kernel_ulong_t)&dev_vdata[3] },
 > 		 	{ PCI_VDEVICE(AMD, 0x15DF), (kernel_ulong_t)&dev_vdata[4] },
 > 		+	{ PCI_VDEVICE(AMD, 0x1649), (kernel_ulong_t)&dev_vdata[4] },
 >
 > {{< quote >}} [LKML: Rijo Thomas: [PATCH] ccp: ccp - add support for Green Sardine](https://lkml.org/lkml/2021/4/16/273) {{< /quote >}}

*Green Sardine/Cezanne APU* の PSP/CCP DeviceID は `0x1649` となる。  
興味が惹かれるのは、*Green Sardine/Cezanne APU* が PSP/CCP では近い世代の APU {{< comple >}} Raven/Picasso/Raven2/Renoir {{< /comple >}} とは異なる DeviceID (恐らく内部的にわずかでも違いが存在するのだろう) であり、そして *VanGogh APU* とは同じ DeviceID であることだ。  

OpenBenchmarking から拝借しているが、lspci の実行結果を見るに *Raven APU* と *Renoir APU* は同じ PSP/CCP (DeviceID: `0x15DF`) を搭載している。[^obm]  
*Cezanne APU* は、Coreboot に追加された DeviceID情報や OpenBenchmarking の実行結果から、*Renoir APU* と I/O部 (Root Complex)、オーディオコントローラー、GBE (Gigabit Ethernet) を共通にするとされる。機能的にも、PCIe Gen3 までといった点は同じだ。  
*Renoir* と *Cezanne* で異なる DeviceID には、データファブリックと内部 GPU がある。  
{{< link >}} [Coreboot に Renoir と Lucienne/Cezanne らしき DeviceID が追加される | Coelacanth's Dream](/posts/2020/11/19/rn-lcn-czn-coreboot-did/) {{< /link >}}

そして、以前 *VanGogh APU* のブートログが投稿されたが、その中で *VanGogh* の PSP/CCP が DeviceID: `0x1649` であることが分かった。*Green Sardine/Cezanne APU* と同じ DeviceID だ。  
{{< link >}} [AMD VanGogh APU ブートログ | Coelacanth's Dream](/posts/2021/03/17/vgh-bootlog/#pci_id) {{< /link >}}

*Green Sardine/Cezanne APU* が I/O部等を *Renoir APU* と同じ IP を採用する中、PSP/CCP は *Raven APU* から続き *Renoir APU* にも搭載されたものではなく、*VanGogh APU* と同じものを搭載するということからは、  
プロセスや CPUアーキテクチャ、I/O部で部分的に同一となることがある Zen系 APU、それぞれの開発時期の違いが覗かせている。  


[^obm]: [AMD Ryzen 5 2400G @ 3.60GHz - lspci - OpenBenchmarking.org](https://openbenchmarking.org/system/2007148-NI-ALEX6767474/AMD%20Ryzen%205%202400G%20@%203.60GHz/lspci), [AMD Renoir - lspci - OpenBenchmarking.org](https://openbenchmarking.org/system/2104156-HA-TEST6314648/AMD%20Renoir/lspci)
