---
title: "Green Sardine APU の PCI ID が追加　―― 正体は Cezanne APU だったか"
date: 2021-01-14T11:36:51+09:00
draft: false
tags: [ "Green_Sardine", "Cezanne", "Renoir", "Lucienne" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

先日の CES2021 {{< comple >}} よりも、主にその発表後のニュースリリース {{< /comple >}} で各種 SKU が正式発表された *Cezanne APU* 、*Lucienne APU* だが、[^ces2021]  
両者と同じく *Renoir APU* に近い存在として、Linux Kernel (amd-gfx) へのパッチに現れた *Green Sardine APU* がいた。  
{{< link >}} [AMD Renoir の兄弟？ Linux Kernel に "Green Sardine" APU をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2020/10/03/amd-apu-green_sardine/) {{< /link >}}

[^ces2021]: [AMD Announces World’s Best Mobile Processors¹ In CES 2021 Keynote :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/986/amd-announces-worlds-best-mobile-processors-in-ces)

それが、新たに投稿されたパッチから、*Green Sardine* は *Cezanne* の別名であることが明らかになった。  

 >    Add green_sardine PCI id support and map it to renoir asic type.
 >
 >          /* Renoir */
 >          {0x1002, 0x1636, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 >      +	{0x1002, 0x1638, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_RENOIR|AMD_IS_APU},
 >
 > {{< quote >}} [[PATCH 1/2] drm/amdgpu: add green_sardine device id (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058408.html) {{< /quote >}}

*Cezanne* の GPU部に割り当てられた DeviceID は、以前に Coreboot へのパッチからわかっており、*Green Sardine* の DeviceID `0x1638` は *Cezanne* と一致する。  
下引用部では、`Family: 0x19, Model: 0x51` となっているが、*Cezanne* は `Family: 0x19, Model: 0x50` であるため、誤字と思われる。  

 >        #define PCI_DEVICE_ID_ATI_FAM17H_MODEL60H_GPU			0x1636
 >        #define PCI_DEVICE_ID_ATI_FAM17H_MODEL68H_GPU			0x164C
 >        #define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU			0x1638
 >
 > {{< quote >}} [include/device/pci_ids: add PCI IDs for new AMD SoCs (I0caea562) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/47703) {{< /quote >}}

*Lucienne* の DeviceID `0x164C` も追加されているが、コードネームについてはこれまで Linux Kernel へのパッチでは触れられていない。  
*Lucienne* は *Renoir Refresh* とも言える {{< comple >}} いやプロセスも変わらないとすると、ほぼ *Renoir* そのままかもしれない {{< /comple >}} APU であり、恐らくはファームウェアも *Renoir* と同じものが使えると思われる。  
コードでも、*Renoir* と *Lucienne* 、*Cezanne/Green Sardine* とで分けられている。  

 >      	case CHIP_RENOIR:
 >      		adev->asic_funcs = &soc15_asic_funcs;
 >     -		if (adev->pdev->device == 0x1636)
 >     +		if ((adev->pdev->device == 0x1636) ||
 >     +		    (adev->pdev->device == 0x164c))
 >      			adev->apu_flags |= AMD_APU_IS_RENOIR;
 >      		else
 >      			adev->apu_flags |= AMD_APU_IS_GREEN_SARDINE;

 > {{< quote >}} [[PATCH 2/2] drm/amdgpu: add new device id for Renior](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058409.html) {{< /quote >}}

一応、*Renoir* は `x86_Model: 0x60` 、*Lucienne* は `x86_Model: 0x68` となるため、CPU部では分けられている。  
それでも I/O、GPU部は *Renoir* と分ける必要がなかったから、コードネームに言及する必要もなかった、ということなのかもしれない。  


