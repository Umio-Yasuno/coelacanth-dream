---
title: "AMDGPUの各種 IPバージョン"
date: 2022-01-31T17:53:21+09:00
draft: false
tags: [ "Linux_Kernel", "GFX9", "RDNA", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU", "GPU", "Database" ]
noindex: false
# summary: ""
---

以前の AMDGPUドライバーでは、DeviceID (DID) から GPU ASIC を検出し、そこから分岐して各種 IPブロック (GC, SDMA, DCN, UVD, VCN, VCE) をサポートする形を採っていたが、少し前 (2021/09) から、VBIOS、ファームウェアに含まれている各種 IPバージョンを取得、直接 IPブロックをサポートする形になった。  
これにより、ドライバーに直接記述された DeviceIDリスト や GPU ASIC に結びつけた各種 IPバージョンの情報が無くとも IPブロックをサポートすることができる。以前の方法では、ある GPU の DeviceID を後から追加した場合、ほとんどサポートされている状態でもその DeviceID が反映されなければ不明の GPU として扱われてしまう。  
例えば、*Renoir, Lucienne, Green Sardine (Cezanne), Barcelo* は共通した構成の GPU部となるが、DevceID は異なるため、サポートには DeviceID を追加するパッチを適用する必要があった。  

IPバージョンを VBIOS、ファームウェアに含むようになったのは *GFX9 (Vega)* 世代からとなり、それ以前の世代は従来の DeviceID から検出する方法が使われる。  

* [[PATCH 00/66] Move to IP driven device enumeration](https://lists.freedesktop.org/archives/amd-gfx/2021-September/069191.html)

ただパッチとコードを眺めているだけの自分にとっては、IPバージョンからではどの AMD GPU を想定した部分のコードなのかわかりづらいため、一度 AMD GPU と各種IPバージョンをまとめてみた。  

{{< pindex >}}
* [AMDGPU IP version table](#table)
{{< /pindex >}}

先に補足的な説明を置くと、各種 IPブロックには個人的に謎な略称がそれなりにあったのだが、最近になってドキュメントが追加され、さらにはその IPブロックがどういった役割を持っているのかも記載されるようになった。  

* [Core Driver Infrastructure — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/driver-core.html)
* [DC Glossary — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/display/dc-glossary.html)
* [AMDGPU Glossary — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/amdgpu-glossary.html)

今回 IPバージョンをまとめた分だけだと、以下のようになっている。  

* GC: Graphiics and Compute
* SDMA: System DMA
* DCE: Display Contoroller Engine
* DCN: Display Core Next
* VCN: Video Codec/Core Next
    * UVD: Unified Video Decoder
    * VCE: Video Compression Engine
* NBIO: North Bridge Input/Output
* PSP (MP0): Platform Security Processor
* SMU (MP1): System Management Unit
* UMC: Unified Memory Contoroller

GC が GFXコア部にあたり、IPブロックの中では最大のものとなる。  
AMD GPU には対応する ISA を示す GPU/GFX/Target ID があり、AMDGPU KFD (Kernel Fusion Driver) ではその情報を sysfs (/sys/class/kfd/kfd/topology/node/\*) に出力するが、GC IPバージョンから判定を行っている。[^gc-gpu_id]  
GC と GPU/GFX/Target ID は近いフォーマットとバージョンになっているが完全に一致はしない。  
コメントでわかりやすく示しているのもあるが、コメントが無い部分ではこんがらがりやすいと思ったのが今回の動機。  
そのため主要な IPブロック、実際の機能に出そうな部分のみをまとめている。すべての IPバージョンをまとめてはいないし、正直曖昧な部分も多い。
あくまでも個人的なデータベースとなっている。  

[^gc-gpu_id]: [kfd_device.c « amdkfd « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdkfd/kfd_device.c?h=next-20220128#n186)

## AMDGPU IP version table {#table}

IPバージョンは `{major}.{minor}.{revision}` のフォーマットで記述される。  

 > 		#define IP_VERSION(mj, mn, rv) (((mj) << 16) | ((mn) << 8) | (rv))
 >
 > {{< quote >}} [amdgpu.h « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/amdgpu.h?h=next-20220128#n767) {{< /quote >}}


| AMD GPU   | GC    | SDMA  | DCE/DCN | VCN (UVD, VCE) | NBIO  |PSP (MP0)|SMU (MP1)| UMC   |
| :-------- | :---: | :---: | :-----: | :------------: | :---: | :---: | :---: | :---: |
|Vega10 (gfx900)| 9.0.1| 4.0.0 | 12.0.0  | (7.0.0, 4.0.0) | 6.1.0 | 9.0.0 | 9.0.0 | 6.0.0 |
|Vega12 (gfx904)| 9.2.1| 4.0.1 | 12.0.1  | (7.0.0, 4.0.0) | 6.2.0 | 9.0.0 | 9.0.0 | 6.1.0 |
|Vega20 (gfx906)| 9.4.0| 4.2.0 | 12.1.0  | (7.2.0, 4.1.0) | 7.4.0 | 11.0.2| 11.0.2| 6.1.1 |
|Arcturus (gfx908)| 9.4.1| 4.2.2| -      | 2.5.0       | 7.4.1 | 11.0.4| 11.0.2| 6.1.2 | 
|Aldebaran (gfx90a)|9.4.2| 4.4.0| -      | 2.6.0       | 7.4.4 | 13.0.2| 13.0.2| 6.7.0 |
|Raven/Picasso (gfx902)| 9.1.0 | 4.1.1 | 1.0.0 | 1.0.0 | 7.0.0 | 10.0.0| 10.0.0| 7.0.0 |
|Raven2 (gfx909)       | 9.2.2 | 4.1.1 | 1.0.1 | 1.0.1 | 7.0.1 | 10.0.1| 10.0.1| 7.5.0 |
|Renoir/Lucienne<br>/Barcelo/Green Sardine \[Cezanne\] (gfx90c) |9.3.0|4.1.2|2.1.0|2.2.0|7.0.0?|12.0.x?|12.0.{1,2}[^rn-smu]|?|
|Navi10 (gfx1010)                |10.1.10| 5.0.0| 2.0.0?| 2.0.{0,2}?| 2.3.x| 11.0.0| 11.0.0| ?|
|Navi12 (gfx1011)                 |10.1.2| 5.0.5| 2.0.0?| 2.0.{0,2}?| 2.3.x| 11.0.9| 11.0.9| ?|
|Navi14 (gfx1012)                 |10.1.1| 5.0.2| 2.0.0?| 2.0.{0,2}?| 2.3.x| 11.0.5| 11.0.5| ?|
|Cyan Skilfish/Skillfish (gfx1013)|10.1.3|5.0.1|2.0.1?|?| 2.3.x? | 11.0.8 | 11.0.8 |?|
|Cyan Skilfish/Skillfish 2 (gfx1013)|10.1.4|5.0.1|2.0.1?|?| 2.3.x? | 11.0.8 | 11.0.8 |?|
|Sienna Cichlid/Navi21 (gfx1030)  | 10.3.0| 5.2.0| 3.0.0?| 3.0.0|2.3.x?| 11.0.7| 11.0.7| 8.7.x|
|VanGogh (gfx1033)                | 10.3.1| 5.2.1| 3.0.1?|3.0.33| 2.3.x?| 11.5.0| 11.5.0| ?|
|Navy Flounder/Navi22 (gfx1031)   | 10.3.2| 5.2.2| 3.0.0?| 3.x.x| 2.3.x?|11.0.11|11.0.11| ?|
|Dimgrey Cavefish/Navi23 (gfx1032)| 10.3.4| 5.2.4| 3.0.0?| 3.x.x| 2.3.x?|11.0.12|11.0.12| ?|
|Beige Goby/Navi24 (gfx1034)      | 10.3.5| 5.2.5| 3.0.3?|3.0.33| 2.3.x?|11.0.13|11.0.13| ?|
|Yellow Carp/Rembrandt (gfx1035)  | 10.3.3| 5.2.3| 3.1.2?| 3.1.1| 2.3.x?|13.0.{1,3}|13.0.{1,3}?| ?|

[^rn-nbio]: [soc15.c « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/soc15.c?h=next-20220128#n1399)
[^rn-smu]: [amdgpu_smu.c « swsmu « pm « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/pm/swsmu/amdgpu_smu.c?h=next-20220128#n526)

{{< ref >}}
* [database · main · Tom St Denis / umr · GitLab](https://gitlab.freedesktop.org/tomstdenis/umr/-/tree/main/database)
* [linux/kfd_device.c at d0a231f01e5b25bacd23e6edc7c979a18a517b2b · torvalds/linux](https://github.com/torvalds/linux/blob/d0a231f01e5b25bacd23e6edc7c979a18a517b2b/drivers/gpu/drm/amd/amdkfd/kfd_device.c)
* [kfd_device.c « amdkfd « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdkfd/kfd_device.c?h=next-20220128)
* [amdgpu_discovery.c « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/amdgpu_discovery.c?h=next-20220128)
* [soc15.c « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/soc15.c?h=next-20220128)
* [nv.c « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/nv.c?h=next-20220128)
* [psp_v13_0.c « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/psp_v13_0.c?h=next-20220128)
* [psp_v11_0.c « amdgpu « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/amdgpu/psp_v11_0.c?h=next-20220128)
{{< /ref >}}
