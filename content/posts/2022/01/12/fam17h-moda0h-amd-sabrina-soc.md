---
title: "Coreboot に Family 17h Model A0h APU/SoC のサポートが追加 ―― Sabrina SoC"
date: 2022-01-12T14:02:42+09:00
draft: false
tags: [ "Coreboot", "Sabrina", "Mendocino" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

ソフトウェアエンジニアである [Felix Held](https://github.com/felixheld) 氏より Coreboot に、AMD の新たな APU/SoC、`Family 17h Model A0h (0x008A0F00)` のサポートを追加するパッチが投稿された。  
パッチは、PCIe GPP Bridge や USBコントローラ、Data Fabric (DF) の PCI ID (DeviceID) を追加する、初期的なサポートを目的としたものとなっている。  

 * [include/device/pci_ids.h: add PCI IDs for AMD Family 17h Model A0h SoC (I41e0a576) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/60984/2)
 * [soc/amd/common/block: add new PCI IDs to common code (I50960e50) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/60985/2)

{{< pindex >}}
 * [Sabrina SoC](#sabrina)
    * [Mendocino](#mendocino)
{{< /pindex >}}

## Sabrina SoC {#sabrina}
パッチで [Felix Held](https://github.com/felixheld) 氏は、SMBus (System Management Bus)、LPC (Low Pin Count) の DeviceID は前世代の Zen-based APU と同じとコメントしている。  
他の情報と照らし合わせたところ、Display HD Audio Controller (HDA) の DeviceID `0x1640` が *VanGogh APU* のものと一致した。  
{{< link >}} [AMD VanGogh APU ブートログ | Coelacanth's Dream](/posts/2021/03/17/vgh-bootlog/#pci_id) {{< /link >}}
*VanGogh* 関連では、*Green Sardine (Cezanne)* の PSP (Platform Security Processor) DeviceID も *VanGogh* と同じとされ、同時期に複数の APU を並行して開発してたことが窺える。  
{{< link >}} [Green Sardine/Cezanne APU の PSP DeviceID が追加される　―― VanGogh APU と同じ PSP か | Coelacanth's Dream](/posts/2021/04/17/green_sardine-apu-psp/) {{< /link >}}

 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL18H_HDA0			0x15DE
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL60H_HDA0			0x1637
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODELA0H_HDA0			0x1640
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/60984/2/src/include/device/pci_ids.h#371> {{< /quote >}}

GPU部の DeviceID が追加されていることから、`Family 17h Model A0h` は APU だと推察される。  
しかし `0x1506` はまだ AMDGPUドライバにサポートが追加されていない DeviceID であり、GPU部の世代/アーキテクチャ等は不明。  

 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL18H_GPU			0x15D8
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL60H_GPU			0x1636
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODEL68H_GPU			0x164C
 > 		#define PCI_DEVICE_ID_ATI_FAM17H_MODELA0H_GPU			0x1506
 > 		#define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU_CEZANNE		0x1638
 > 		#define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU_BARCELO		0x15e7
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/60984/2/src/include/device/pci_ids.h#377> {{< /quote >}}

`Family 17h` ということから CPUアーキテクチャは *Zen /2* 系とされる。  
ただ CPU部においても、ハードウェアモニタリングドライバ `hwmon` 等に `Family 17h Model A0h` のサポートが追加されていないため、詳細は不明。  
Coreboot には *Zen 3 APU* 、*Green Sardine (Cezanne), Barcelo* のサポートがすでに追加されており、CPUアーキテクチャとしては前世代のものが新たに追加されることとなる。  

今回のパッチ 2つには *amd-sabrina-soc* というトピックが付与されており、`Family 17h Model A0h` を指していると思われる。  
ただ、GPU部に対するソフトウェアではコードネーム *VanGogh* が使われつつも、SoC としては *Aerith* という別のコードネームを持つ前例がある。[^aerith] 同様に、AMDGPUドライバ等では `Family 17h Model A0h` に対し、*Sabrina* とは別のコードネームを用いる可能性がある。  

 * [topic:amd-sabrina-soc · Gerrit Code Review](https://review.coreboot.org/q/topic:amd-sabrina-soc)

[^aerith]: {{< youtube id="SsqvY0buseQ" start="23" title="Steam Deck Hardware (an Overview)" >}}

### Mendocino {#mendocino}
Coreboot には以前、`amdfwtool` にコードネーム *Mendocino* のサポートが追加されている。  
{{< link >}} [AMD の新たな APU/SoC 「Mendocino」 | Coelacanth's Dream](/posts/2021/08/12/amd-mendocino-soc/) {{< /link >}}
*Mendocino* もまた詳細が明らかにされていない APU/SoC であり、*Sabrina SoC* と近い存在ではあるが、それらが一致するかどうかの情報もまだ明らかにされていない。  
だが今回のパッチは、新たな APU/SoC を OSS に対して公開し、サポートを追加する段階となったことを示しており、近いうちにさらなる情報が公開されるのではないかと思われる。  

*Mendocino APU* では、PSP部が更新され、PSP に ISH と呼ぶヘッダ情報が追加されることは判明している。[^psp-ish]  

[^psp-ish]: [amdfwtool: Add ISH header support for A/B recovery layout (I4710df40) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59384)


| AMD APU | CPU Arch | (Family, Model) | GPU Arch |
| :-- | :--: | :--: | :--: |
| Raven | Zen/+ | (0x17, 0x11) | Vega (gfx902) |
| Picasso (& some Dali)[^dali-mod] | Zen/+ | (0x17, 0x18) | Vega (gfx902) |
| Raven2 (Dali, Pollock) | Zen/+ | (0x17, 0x20) | Vega (gfx909) |
| Renoir | Zen 2 | (0x17, 0x60) | Vega (gfx90c) |
| Lucienne | Zen 2 | (0x17, 0x68) | Vega (gfx90c) |
| VanGogh | Zen 2 | (0x17, 0x90) | RDNA 2 (gfx1033) |
| Sabrina SoC | ? | (0x17, 0xA0) | ? |
| Mendocino | ? | ? | ? |
| Green Sardine (Cezanne)<br>/Barcelo | Zen 3 | (0x19, 0x50) | Vega (gfx90c) |
| Yellow Carp (Rembrandt) | Zen 3+ | (0x19, 0x44)[^yc-rmb] | RDNA 2 (gfx1035) |

[^dali-mod]: [AMD Dali APU データベース | Coelacanth's Dream](/posts/2020/06/24/amd-dali-apu-database/#dali-x86model)
[^yc-rmb]: [AMD Yellow Carp/Rembrandt APU ブートログ ―― LilacKD-RMB, 100-000000560-40_Y | Coelacanth's Dream](/posts/2021/12/14/yc-rmb-bootlog/#cpu)

{{< ref >}}
 * [linux/k10temp.c at 4aa1b8257fba5931511a7e152bcbbb3dd673c6c1 · torvalds/linux](https://github.com/torvalds/linux/blob/4aa1b8257fba5931511a7e152bcbbb3dd673c6c1/drivers/hwmon/k10temp.c)
{{< /ref >}}
