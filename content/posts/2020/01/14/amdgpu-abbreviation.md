---
title: "AMDGPU関連用語略称まとめ"
date: 2020-01-14T23:38:40+09:00
draft: false
# tags: [ "Database", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU", "Database" ]
noindex: false
summary: " "
---

個人的なメモ

## Software
| Abbr | Full | memo |
| :--: | :--: | :--: |
| UMD | User Mode graphics Drivers | |
| KMD | Kernel Mode graphics Drivers | |
| KDG | Kernel Graphics Driver | |
| KMS | Kernal Mode Setting | drm[^1] |
| TTM | Translation Table Maps | drm[^1] |
| MMU | Memory Managiment Unit | |
| MN | MMU Notifier | drm[^1] |
| GEM | Graphics Execution Manager | drm[^1] |
| GTT | Graphics Address Remapping Table | drm[^1] |
| PD | Primitive Discard | RadeonSI /RADV |
| HSA | Heterogeneous System Architecture | |
| HMM | Heterogeneous Memory Management | |
| OA | Ordered Append | |
| MSI | Massage Signaled Interrupts | drm[^1] |
| EMU | Emulation mode | drm[^1] |
| ABM | Adaptive Backlight Management | drm[^1] |
| BAPM | Bidirectional Application Power Management | |
| LBPW | Load Balancing Per Watt | |
| DPM | Dynamic Power Management | |
| DPMS | DMP State? | |
| CWSR | Compute Wave Save and Restore | drm[^1] |
| GWS | Global Wave Sync | drm[^1] |
| CRAT | Component Resource Association Table | GPU cache info |
| VF | Virtual Function | MxGPU |
| DPBB | Deferred Primitive Batch Binning | DSBR, partial primitive binning |
| DFSM | Deterministic Finite State Machine | DSBR, full primitive binning |

[^1]:[drm/amdgpu AMDgpu driver — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu.html)

## Hardware
| Abbr | Full | memo |
| :--: | :--: | :--: |
| SE | Shader Engine | |
| SA/SH | Shader Array | GCN(1SE =1SA =16CU), RDNA(1SE =2SA =10WGP =20CU) |
| HWS | Hardware Schedulers | |
| ACE | Asynchronous Compute Engine | |
| RB | Render Backend | =4 ROPs |
| ROP | Render Output Pipeline | |
| GMC | Graphics(GFX?) Memory Controller | |
| GDS | Global Data Share | |
| LDS | Local Data Share | |
| DMA | Direct Memory Access | |
| SDMA | System DMA | |
| GCP | Graphics Command Processor | |
| MCBP | Mid Command Buffer Preemption | |
| CS | Command Stream | |
| EOP | End Of Packet | |
| SMU | System Manasgement Unit | |
| PG | Powergating | |
| RLC | Rear left Center? | |
| PFP | Prefetch Parser | |
| CE | Constant Engine | |
| DE | Dispatch Engine | |
| ME | Micro Engine | |
| MES | Micro Engine Scheduler | GFX10? |
| MEC | Micro Engine Compute? | |
| PSP | Platform Security Processor | |
| SDP | Scalable Data Port | |
| PA | Primitive Assembly  | |
| IA | Input Assembly | |
| HTILE | | Hi-Z Depth Compression |
| IB | Indirect Buffer | = GPU command buffer |
| LRU | Least Recently Used | |
| CGPG | Coarse Grained PoweGating | |
| UVD | Unified Video Decoder | |
| VCE | Video Compression (/Codec) Engine | |
| VCN | Video Core Next | |
| DC | Display Core | |
| PSR | Panel Self-Refresh | eDP PowerSave |
| DCE | Display Core Engine? | |
| DMCUB | Display Micro Controller Unit B | |
| HBCC | High Bandwidth Cache Controller | GFX9+ |
| DSBR | Draw Stream Binning Rasterrizer | GFX9+ |
| SGPR | Scalar General-Purpose Register | |
| VGPR | Vector General-Purpose Register | |
| ALU | Arithmetc Logic Unit | |
| SPI | Shader Processor Interpolator | |
| PRT | Partially Resident Textures | |
| DIO | Display IO | DCN3 |
| OPP | Output Plane Processing | DCN3 |
| MPC | Multiple pipe and plane combine | DCN3 |
| DPP | | DCN3 |
| HUBBUB | | DCN memory HUB interface /DCN3 |
| MMHUBBUB | | Multimedia HUB interface /DCN3 |
| HUBP | | Display to data fabric interface /DCN3 |
| DWB | Display Writeback | DCN3 |
| DML | Display mode library | DCN3 |
| AMFT | Audio formating | |
| VPG | Video Package ganerator | |
| SPL | Security patch level | |
| PC_LINES | Parameter Cache Lines? | |
| HDP | Host Data Path[^hpd] | |
| MGCG | Medium Grain Clock Gating | |
| MGLS | Medium Grain Light Sleep | |
| IH | Interrupt Handler[^ih] | |

[^hpd]: [drm/amdgpu: add the HDP 4.0 register headers](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=bcfb47cdd76a20b3c596981ea7b35fa23abac4c8)
[^ih]: [drm/amdgpu: add navi10 ih ip block (v3)](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdgpu/navi10_ih.c?h=amd-staging-drm-next&id=edc611475a8adbdea8ce358216c5b1a9d710049c)



## Other
| Abbr | Full | memo |
| :--: | :--: | :--: |
| CDIT | Component locality Distance Information Table | |
| AO | Always On | |
| DCC | Delta Color Compression | lossless |
| DSC | Display Stream Compression | DCN 2.0+ |
| TDP | Thermal Design Power | |
| TBP | Typical Board Power | |
| TMZ | Trusted Memory Zone | |
| DWB | Display WriteBack | |
| RAS | Reliability, Availability, Serviceability | |
| HDCP | Highbandwidth Digital Content Protection | |
| TA | Texture Address /Trusted Application?[^trusted-application] | |
| TC | Texture Cache? | |
| TCA | Texture Cache Arbiter ??? | |
| TCC | Texture Cache Channel ??? | == L2cache |
| TCP | Texture Cache Private ??? | GCN L1$, RDNA L0$ |
| BACO | Bus Active, Chip Off[^2] | |
| BOCO | Bus Off, Chip Off[^2] | |
| OPN | ordering Part Number | |
| ULV | Ultra Low Voltage[^ulv] |
| GL2a | GL2 Arbiter[^gl2a] | == TCA[^gl2a-eq-tca]<br>(Navi10 = 4, Navi14 = 2)
| GL2c | <!-- GL2 Cache ??? --> | == TCC[^gl2a-eq-tca]<br>(Navi10 = 16, Navi14 = 8)
| SC | Shader Compiler? / Scalar? | |
| GPA | Guest Pysical Address[^gpa] | |
| EDC | Error Correction and Detection[^edc] | |

[^ulv]: [drm/amd/powerplay: enable Ultra Low Voltage for sienna_cichlid](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/powerplay/sienna_cichlid_ppt.c?h=amd-staging-drm-next&id=e912f967af0182039fb9f2668c78967f0056a769)
[^gl2a]: [pal/gfx9PerfCtrInfo.cpp at c9937b277c491a55c689eec7cdd48fb238b8004c · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/c9937b277c491a55c689eec7cdd48fb238b8004c/src/core/hw/gfxip/gfx9/gfx9PerfCtrInfo.cpp#L1488)
[^gl2a-eq-tca]: [pal/devDriverUtil.cpp at a3a42838c576cc219e61500bc469a4a05ce0db68 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/a3a42838c576cc219e61500bc469a4a05ce0db68/src/core/devDriverUtil.cpp#L94)
[^gpa]: [drm/amdgpu: use gpu virtual address for interrupt packet write space for vangogh](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=bb627a32a69ed19c8ac14e386b674266061b8bb3)
[^edc]: [drm/amdgpu: add EDC support for CZ (v3)](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=ccba7691a580a0967f60a512473ce699b9edac0d)

[^trusted-application]: [drm/amdgpu/psp: add structure for xgmi ta and its shared buffer](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=f0cfa19579fae3bd06366ebccdba26020bb6214a)
[^2]: [drm/amdgpu: rename amdgpu_device_is_px to amdgpu_device_supports_boco (v2)](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=tmz&id=31af062acfbd5db8b0b99d0ad418b33d4458e206)

<!--
RVP (Reference Validation Platform)
-->
