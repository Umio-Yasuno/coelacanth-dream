---
title: "AMDGPU関連用語略称まとめ"
date: 2020-01-14T23:38:40+09:00
draft: false
tags: [ "Database", ]
keywords: [ "", ]
categories: [ "", ]
noindex: false
---

最初に断っておくと、これはあくまで個人的なメモであり、資料としてはいくつかの問題を抱えている。  
わからない略語を見つける -> 探し出す -> メモする の繰り返しで貯まったものをテーブル形式に出力しただけだ。  
そのためソースが記載されていないのがほとんどで、順番も一応関係あるものを固めているがやっぱりメチャクチャ。  
かといってA-Z順ではそれはそれでメチャクチャだ。  
だからブラウザのページ内検索を前提にした、今後わからない略語に会った時の解決に役立てたらいいな、というものだ。  

#### Software
| Abbr | Full | memo |
| :--: | :--: | :--: |
| UMD | User Mode graphics Drivers | |
| KMD | Kernel Mode graphics Drivers | |
| KDG | Kernel Graphics Driver | |
| KMS | Kernal Mode Setting | drm[^1] |
| TMM | Translation Table Maps | drm[^1] |
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

#### Hardware
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
| MES | Micro Engine Scheduler | GFX10? |
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
| DCE |
| DMCUB | Display Micro Controller Unit B | |
| HBCC | High Bandwidth Cache Controller | GFX9+ |
| DSBR | Draw Stream Binning Rasterrizer | GFX9+ |

<!--
| SGPR | Scalar General-Purpose Register | |
| VGPR | Vector General-Purpose Register | |
| ALU | Arithmetc Logic Unit | |
| SPI | Shader Processor Interpolator | |
| PRT | Partially Resident Textures | |
-->

#### Other
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
| TA | Trusted Application | |
