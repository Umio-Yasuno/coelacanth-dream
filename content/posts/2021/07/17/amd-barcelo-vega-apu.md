---
title: "新たな Zen 3 + Vega APU 「Barcelo」"
date: 2021-07-17T06:24:35+09:00
draft: false
tags: [ "Barcelo", "Cezanne", "Zen_3", "Coreboot" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

プロプライエタリな BIOS (ファームウェア) の代替を目的としたフリーソフトプロジェクト [Coreboot](https://www.coreboot.org/) に、Felix Held 氏より、新たな AMD APU *Barcelo* をサポートする最初のパッチが投稿された。  
パッチは *Barcelo* の PCI (Device) ID を追加するものとなっている。  

 * [soc/amd/common/block/graphics: add GPU PCI ID for Barcelo (I1c5446c1) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56396)
 * [soc/amd/cezanne/graphics: add VBIOS ID remapping for Barcelo (I2eed4370) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56392)

 > 		 #define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU_CEZANNE		0x1638
 > 		 #define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU_BARCELO		0x15e7
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/56396/3/src/include/device/pci_ids.h#507> {{< /quote >}}

*Barcelo* の Device ID は `0x15e7` となっている。この ID はちょうど先日 Linux Kernel における AMD GPUドライバーに追加されており、その時のパッチタイトルは *「[PATCH] drm/amdgpu: add another Renior DID」* 。[^another-rn]  
{{< link >}} [AMD Renoir APU の GPU ID が 「gfx90c」 に | Coelacanth's Dream](/posts/2020/10/31/amd-renoir-apu-gfx90c/) {{< /link >}}
*Barcelo* は *Renoir・Lucienne・Cezanne* 同様に Vega GPU を搭載し、GPU ID は *gfx90c* になると見られる。  

 > 		 	/* Renoir */
 > 		+	{0x1002, 0x15E7, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 > 		 	{0x1002, 0x1636, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 > 		 	{0x1002, 0x1638, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 > 		 	{0x1002, 0x164C, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 >
 > {{< quote >}} [[PATCH] drm/amdgpu: add another Renior DID](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066502.html) {{< /quote >}}

[^another-rn]: [[PATCH] drm/amdgpu: add another Renior DID](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066502.html)

CPU に関してはパッチではあまり触れられていないが、*Cezanne* 同様に *Zen 3 アーキテクチャ* を採用しているのではないかと思う。  
*Barcelo* は *Raven* に対する *Picasso* 、*Renoir* に対する *Lucienne* のような、構成はほぼ同じだが製造プロセスや電力管理機能等にわずかな改良を施した後継 APU というポジションにいるのだろう。  

| GPUID | Arch | Codename |
| :-- | :--: | :--: |
| *gfx909* | Vega | Raven2 (Dali/Pollock) |
| *gfx90c* | Vega | Renoir/Lucienne/Cezanne (Green Sardine)[^green_sardine]
| *gfx1010* | RDNA | Navi10 |
| *gfx1011* | RDNA | Navi12 |
| *gfx1012* | RDNA | Navi14 |
| *gfx1013* | RDNA | ?????<br>(APU, Navi10-base, Support RayTracing Inst.)[^gfx1013] |
| *gfx1030* | RDNA 2 | Sienna Cichlid/Navi21 |
| *gfx1031* | RDNA 2 | Navy Flounder/Navi22 |
| *gfx1032* | RDNA 2 | Dimgrey Cavefish/Navi23 ? |
| *gfx1033* | RDNA 2 | VanGogh? (APU) |
| *gfx1034* | RDNA 2 | Beige Goby? |
| *gfx1035* | RDNA 2 | Yellow Carp? (APU) |

[^green_sardine]: [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/)
[^gfx1013]: [gfx1013 | Coelacanth's Dream](/tags/gfx1013/)
