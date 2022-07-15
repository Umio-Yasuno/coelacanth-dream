---
title: "AMD Sabrina SoC は GC 10.3.7"
date: 2022-05-03T04:48:37+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Sabrina", "Mendocino", "GC_10_3_7", "RDNA_2" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

*AMD Sabrina* は現在 Coreboot でサポートが進められている AMD SoC だが、Linux Kernel における AMDGPUドライバーが DeviceID (PCI ID) から IPブロックベースのサポートに移行したことにより、GPU部の情報は不透明だった。  
それが、AMD の Mandar Sahastrabuddhe 氏が投稿したパッチにより、*AMD Sabrina SoC* の GPU部は *RDNA 2 アーキテクチャ* とされる *GC 10.3.7* だということが明らかにされた。  
パッチの投稿先は、Android と ChromeOSデバイスの自動テストフレームワークである autotest のレポジトリとなっている。  

 * [autotest: Add pci id for skyrim (I0d0e1f1f) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/autotest/+/3616382/3/)

パッチによれば、*GC 10.3.7* の DeviceID は `0x1506`、これは *AMD Sabrina SoC* の DeviceID と一致する。  

 > 		--- a/client/bin/amd_pci_ids.json
 > 		+++ b/client/bin/amd_pci_ids.json
 > 		@@ -1,4 +1,5 @@
 > 		 {
 > 		+    "0x1506": "gc_10_3_7",
 > 		     "0x15d8": "picasso",
 > 		     "0x15e7": "green_sardine",
 > 		     "0x1638": "green_sardine",
 >
 > {{< quote >}} <https://chromium-review.googlesource.com/c/chromiumos/third_party/autotest/+/3616382/3/client/bin/amd_pci_ids.json#9> {{< /quote >}}

 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL18H_GPU			0x15D8
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL60H_GPU			0x1636
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL68H_GPU			0x164C
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODELA0H_GPU			0x1506
 > 		#define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU_CEZANNE		0x1638
 > 		#define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU_BARCELO		0x15e7
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/60984/2/src/include/device/pci_ids.h#377> {{< /quote >}}

## GC 10.3.7 {#gc_10_3_7}

*GC 10.3.7* のサポートは AMDGPUドライバーで既に一部行われており、必要とするファームウェアも AMD の Alex Deucher 氏により [linux-firmware.git](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/) にアップロードされている。  
{{< link >}} [新たな RDNA 2 APU の IPブロック ―― GC 10.3.7, DCN 3.1.6, VCN 3.1.1 | Coelacanth's Dream](/posts/2022/02/17/amdgpu-new-rdna_2-ip/) {{< /link >}}

[^amdgpu-firmware]: [kernel/git/firmware/linux-firmware.git - Repository of firmware blobs for use with the Linux kernel](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/amdgpu?id=c025e59f8bc987b626b8c5b623b2af77b6959330)

*AMD Sabrina SoC/GC 10.3.7* は [Yellow Carp/Rembrandt APU](/tags/yellow_carp) に近い構成となっており、マルチメディアエンジンには *Yellow Carp/Rembrandt APU* と同じ VCN 3.1.1 を採用しており、  
ディスプレイエンジンもリビジョンだけが異なる DCN 3.1.6 を採用している。  

*AMD Sabrina SoC* の CPU部、I/O部は Coreboot へのパッチから、`Family 17h Model A0h` であることから *Zen/+/2 アーキテクチャ* である可能性があること、CPPC2 (Collaborative Processor Performance Control 2) をサポートせず、また SATAコントローラを持たないことがわかっている。  
{{< link >}} [AMD Sabrina SoC に向けたさらなるパッチ | Coelacanth's Dream](/posts/2022/01/14/amd-sabrina-soc-more-patch/) {{< /link >}}
また、メモリには LPDDR5 をサポートしている。  
{{< link >}} [AMD Sabrina SoC は LPDDR5メモリをサポート | Coelacanth's Dream](/posts/2022/02/02/amd-sabrina-lpddr5/) {{< /link >}}
