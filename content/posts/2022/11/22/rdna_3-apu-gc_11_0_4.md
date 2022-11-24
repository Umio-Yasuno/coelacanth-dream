---
title: "もう 1つの RDNA 3 APU IP、GC 11.0.4"
date: 2022-11-22T20:26:56+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "GFX11", "RDNA_3", "Phoenix" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Yifan Zhang 氏により、Linux Kernel における AMDGPU ドライバーに *GC 11.0.1 (gfx1103, Phoenix APU)* に続く RDNA 3 APU IP、*GC 11.0.4* のサポートを追加するパッチが投稿されている。  

 * [[PATCH 01/19] drm/amdgpu/discovery: enable soc21 common for GC 11.0.4](https://lists.freedesktop.org/archives/amd-gfx/2022-November/086807.html)

 >         
 >          	case IP_VERSION(11, 0, 1):
 >         +	case IP_VERSION(11, 0, 4):
 >          		adev->flags |= AMD_IS_APU;
 >          		break;
 >         
 > {{< quote >}} [[PATCH 07/19] drm/amdgpu/discovery: set the APU flag for GC 11.0.4](https://lists.freedesktop.org/archives/amd-gfx/2022-November/086813.html) {{< /quote >}}

*GC 11.0.4* は現時点では基本 *GC 11.0.1* と共通したコードパスを用いており、AMDGPUファミリーには `AMDGPU_FAMILY_GC_11_0_1`、GPU ID には *gfx1103* が設定されている。  
同時に *SMU IP v13.0.11* と *PSP IP v13.0.11* をサポートするパッチも投稿されているが、異なるのはレジスタのオフセット値などで、他 IP バージョンとの機能的な違いは今の所見られない。  

 >          	case IP_VERSION(11, 0, 1):
 >         +	case IP_VERSION(11, 0, 4):
 >          		adev->family = AMDGPU_FAMILY_GC_11_0_1;
 >          		break;
 >
 > {{< quote >}} [[PATCH 06/19] drm/amdgpu: set GC 11.0.4 family](https://lists.freedesktop.org/archives/amd-gfx/2022-November/086812.html) {{< /quote >}}
 >
 >          		case IP_VERSION(11, 0, 1):
 >         +		case IP_VERSION(11, 0, 4):
 >          			gfx_target_version = 110003;
 >          			f2g = &gfx_v11_kfd2kgd;
 >          			break;
 >         
 > {{< quote >}} [[PATCH 11/19] drm/amdkfd: add GC 11.0.4 KFD support](https://lists.freedesktop.org/archives/amd-gfx/2022-November/086817.html) {{< /quote >}}

AMD は 2023年のモバイル向けプロセッサに、**Ryzen 7040 Series** として *Phoenix APU* を、**Ryzen 7045 Series** に *Dragon Range APU* を投入することを発表している。[^dragon_range]  
今回パッチが投稿された *GC 11.0.4* はその *Dragon Range APU* の GPU IP という可能性が考えられる。  

*Phoenix APU* は TDP 35-45W、モバイル用途と薄型ゲーミング PC 向け、*Dragon Range* は TDP 55W+、最高の性能を求めるゲーミング PC 向けという位置付けになっている。  
メモリは *Phoenix APU* が LPDDR5 メモリをサポートし、*Dragon Range APU* は DDR5 メモリをサポートするとしている。  
それぞれの CPU/GPU の規模はまだ発表されておらず、当然 AMDGPU ドライバーのソースコードから読み取れるものではないが、ターゲット帯と TDP 帯の違いから、*Phoenix APU* と *Dragon Range APU* の CPU/GPU アーキテクチャは共通するが規模は異なることが考えられる。  

[^dragon_range]: [Announcing New Model Numbers for 2023+ Mobile Proc... - AMD Community](https://community.amd.com/t5/corporate/announcing-new-model-numbers-for-2023-mobile-processors/ba-p/543985)

{{< ref >}}
 * [Announcing New Model Numbers for 2023+ Mobile Proc... - AMD Community](https://community.amd.com/t5/corporate/announcing-new-model-numbers-for-2023-mobile-processors/ba-p/543985)
 * [AMD announces 2023 'extreme gaming laptop CPU,' Dragon Range | PCWorld](https://www.pcworld.com/article/697301/amd-announces-2023-extreme-gaming-laptop-cpu-dragon-range.html)
{{< /ref >}}
