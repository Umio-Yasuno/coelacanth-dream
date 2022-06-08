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
| DFSM | ? | DSBR, full primitive binning |
| PBB | Pipeline Binning[^pbb] |

[^pbb]: [xgl/settings_xgl.json at 0d602bbcfa1a86fd52f66966554b05e384a10d38 · GPUOpen-Drivers/xgl](https://github.com/GPUOpen-Drivers/xgl/blob/0d602bbcfa1a86fd52f66966554b05e384a10d38/icd/settings/settings_xgl.json#L5132)
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
| GCA | Graphics and Compute Array[^gca] | |
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
| PFP | Prefetch Parser[^pfp] | |
| CE | Constant Engine | |
| DE | Dispatch Engine | |
| ME | Micro Engine | |
| MES | Micro Engine Scheduler | GFX10? |
| MEC | Micro Engine Compute? | |
| PSP | Platform Security Processor | |
| RAP | Register Access Policy[^rap] | |
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
| SPI | Shader Processor Interpolator / Shader Processor Input | |
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
| DMUB | Display Micro-Controller Unit[^dmub] |  |
| AMFT | Audio formating | |
| VPG | Video Package ganerator | |
| SPL | Security patch level | |
| THM | Thermal Controller[^thm] |
| PC_LINES | Parameter Cache Lines? | |
| HDP | Host Data Path[^hpd] | |
| MGCG | Medium Grain Clock Gating | |
| MGLS | Medium Grain Light Sleep | |
| IH | Interrupt Handler[^ih] | |
| TMR | Trust Memory Region[^tmr] | |
| SQ | (Distributed) Sequencer | |
| OSS | OS Service[^oss] | |

[^pfp]: [drm/radeon: add initial ucode loading for CIK (v5) · torvalds/linux@02c8132](https://github.com/torvalds/linux/commit/02c813274114976cb104a32407a21c10c89b0465)
[^oss]: [drm/amdgpu: add OSS 2.0 register headers · torvalds/linux@599bd21](https://github.com/torvalds/linux/commit/599bd21552e191913f26f581b549ec41d66984be#diff-52a7043d2b1a6cdc0d495711c1c1fff298b7d9f800cc1c59c623fb666acfd663)
[^thm]: [drm/amdgpu/include: add thm 11.0.2 headers · torvalds/linux@e6af616](https://github.com/torvalds/linux/commit/e6af616a7822294dac294d92a04772a467ef9fd7#diff-e6bbf8860aa3bcf36ac100d5c2aea51b1a818a025b1fe97b74e479aacf1d71c8)
[^gca]: [drm/amdgpu: add GCA 7.0 register headers · torvalds/linux@9f24d8c](https://github.com/torvalds/linux/commit/9f24d8ce2522808cdb719775dbb32cb296e20f47#diff-171aaced998e08b7e48c5b4a96eadc9fd50d95dbc1be7c5ba821c29abf671888)
[^rap]: [drm/amdgpu: add RAP TA header file](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=a189d0ae0cd68643204cbe9b11e824414b2b06c9)
[^dmub]: [drm/amd/display: Add DCN3 DMUB](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=5baebf61ba0ce14253a513ac92be661b35a19676)
[^tmr]: [drm/amdgpu:add tmr mc address into amdgpu_firmware_info](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=abf412b3efb2f943d9b98a489e9aca836be21333)
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
| TCC | Texture Channel  Cache?[^tcc] | == L2cache |
| TCP | Texture Cache Private ??? | GCN L1$, RDNA L0$ |
| BACO | Bus Active, Chip Off[^2] | |
| BOCO | Bus Off, Chip Off[^2] | |
| OPN | ordering Part Number | |
| ULV | Ultra Low Voltage[^ulv] |
| GL2a | GL2 Arbiter[^gl2a] | == TCA[^gl2a-eq-tca]<br>(Navi10 = 4, Navi14 = 2)
| GL2c | <!-- GL2 Cache ??? --> | == TCC[^gl2a-eq-tca]<br>(Navi10 = 16, Navi14 = 8)
| SC | Shader Compiler? / Scan Converter[^scan-converter] | Rasterizer |
| SX | Shader Export[^scan-converter] | |
| GPA | Guest Pysical Address[^gpa] | |
| EDC | Error Correction and Detection[^edc] | |
| SPM | Streaming Performance Counter[^spm] | |
| ICD | Installable Client Driver | |
| MALL | Memory Access (at) Last Level[^mall] | GFX10.3+ |
| SRD | Shader Resource Descriptor[^srd] | |
| PRT | Partially Resident Texture[^prt] | |
| TDR | Timeout Detection and Recovery ?[^tdr] | |

[^tdr]: [AMD_APP_SDK_Release_Notes_Developer.fm - AMD_APP_SDK_Release_Notes_Developer.pdf](http://developer.amd.com/download/AMD_APP_SDK_Release_Notes_Developer.pdf)
[^tcc]: [[radeonsi] Tahiti LE: GFX block is not functional, CP is okay (#1208) · Issues · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/issues/1208#note_240161)
[^prt]: [pal/palDeveloperHooks.h at c9937b277c491a55c689eec7cdd48fb238b8004c · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/c9937b277c491a55c689eec7cdd48fb238b8004c/inc/core/palDeveloperHooks.h#L325)
[^srd]: [pal/palCodingStandards.md at 4ae736bdbc5d5dee59851ac564c5e21d807b44b0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/4ae736bdbc5d5dee59851ac564c5e21d807b44b0/doc/process/palCodingStandards.md)
[^mall]: [[PATCH 2/3] drm/amdgpu: add support to configure MALL for sienna_cichlid (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/055006.html)
[^scan-converter]: [AMD PowerPoint- White Template - nordic-game-2019-triangles-are-precious.pdf](https://gpuopen.com/presentations/2019/nordic-game-2019-triangles-are-precious.pdf)
[^spm]: <https://github.com/GPUOpen-Drivers/pal/blob/477c8e78bc4f8c7f8b4cd312e708935b0e04b1cc/inc/core/palPerfExperiment.h#L92>
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
