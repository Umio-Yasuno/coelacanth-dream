---
title: "Coreboot に Renoir と Lucienne/Cezanne らしき DeviceID が追加される"
date: 2020-11-18T22:00:24+09:00
draft: false
tags: [ "Renoir", "Lucienne", "Cezanne" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

弱気なタイトル。  

Coreboot に AMD APU/SoC の各 DeviceID (PCI ID) を追加するパッチが投稿された。  
{{< link >}} [include/device/pci_ids: add PCI IDs for new AMD SoCs (I0caea562) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/47703) {{< /link >}}
主に *Renoir APU* の DeviceID 追加となっており、勿論 Coreboot の *Renoir* が始まったことは喜ぶべきことで、*Renoir* を搭載した Chromebook という光も見えてくるのだが、  
それ以外に、将来的に登場するだろう APU の DeviceID も含まれていた。  

 >        #define PCI_DEVICE_ID_ATI_FAM17H_MODEL60H_GPU			0x1636
 >        #define PCI_DEVICE_ID_ATI_FAM17H_MODEL68H_GPU			0x164C
 >        #define PCI_DEVICE_ID_ATI_FAM19H_MODEL51H_GPU			0x1638
 >
 > {{< quote >}} [include/device/pci_ids: add PCI IDs for new AMD SoCs (I0caea562) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/47703) {{< /quote >}}

上引用部は今回追加された APU の GPU部に割り当てられた DeviceID だが、まず `FAM17H_MODEL60H` は *Renoir* の `x86_Family, x86_Model` を示している。`0x1636` というのも *Renoir* に割り当てられている DeviceID。  
その下 2行が現時点では未発表、将来的に登場する APU となる。  

`FAM17H_MODEL68H` は、先日 Geekbench に実行結果が現れた **Ryzen 5 5700U** のものと一致することから、コードネーム *Lucienne* ではないかと思われる。  
{{< link >}} [Ryzen 7 5700U は Zen 2 + Vega、Lucienne か　―― Cezanne との棲み分けは | Coelacanth's Dream](/posts/2020/11/12/ryzen-7-5700u-lcn/) {{< /link >}}
GPUアーキテクチャとしては変わらない、全く一致しているとも言える *Raven* 、*Picaso* でも、前者は `0x15DD` 、後者は `0x15D8` の一部が割り当てられているように、  
*Renoir* と *Lucienne* でも GPU には別々の DeviceID が当てられると考えられる。  {{< link >}} [AMD Renoir の兄弟？ 現れるは Lucienne | Coelacanth's Dream](/posts/2020/06/20/amd-lucianne-apu/) {{< /link >}}

`FAM19H_MODEL51H` についてはまだ確認されていないが、`Family 19 (Zen 3)` ということから *Cezanne* ではないかと思われる。  
また、*Renoir (Family 0x17, Model 0x60)* は他に PCIe GPP、USB、オーディオコントローラー、GBE (Gigabit Ethernet) の DeviceID が追加されているのに対し、  
*Cezanne (Family 0x19, Model 0x51?)* は Data Fabric と GPU部の DeviceID しか追加されていないが、  
これに関しては、以前 Openbenchmarking.org に現れた結果から、I/O部やオーディオコントローラーの多くが *Renoir* と共通しているからだと思われる。  
{{< link >}} [AMD Cezanne APU ベンチマーク結果　―― 8-Core/16-Thread, 3.6GHz、構成は Zen 3 + Vega か | Coelacanth's Dream](/posts/2020/08/31/amd-cezanne-benchmark/) {{< /link >}}
GPU部の DeviceID が異なるのは *Lucienne (Family 0x17, Model 0x68?)* と同様の理由だろう。  


