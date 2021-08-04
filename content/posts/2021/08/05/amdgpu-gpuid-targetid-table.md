---
title: "AMDGPU GPUID/TargetID table"
date: 2021-08-05T03:51:30+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Database", ]
noindex: false
# summary: ""
---

## Table

`gfx{major}_{minor}_{stepping}`

| GPUID | Arch | Type | Codename (aka) |
| :-- | :--: | :--: | :--: |
| **GFX9 (Vega/ CDNA 1/ CDNA 2)** |
| *gfx900* | Vega | dGPU | Vega10 |
| *gfx904* | Vega | APU | Raven /Picasso |
| *gfx904* | Vega | dGPU | Vega12 |
| *gfx906* | Vega | dGPU | Vega20 |
| *gfx908* | CDNA | dGPU | Arcturus (MI100) |
| *gfx909* | Vega | APU | Raven2 |
| *gfx90a* | CDNA 2 | dGPU | Aldebaran (MI200) |
| *gfx90c* | Vega | APU | Renoir/ Lucienne/<br>Cezanne (Green Sardine)[^green_sardine] |
| *gfx940* [^gfx940] | ? | ? |
| **GFX10.1 (RDNA 1)** |
| *gfx1010* | RDNA | dGPU | Navi10 |
| *gfx1011* | RDNA | dGPU | Navi12 |
| *gfx1012* | RDNA | dGPU | Navi14 |
| *gfx1013* | RDNA | APU | Cyan Skilfish[^gfx1013] |
| **GFX10.3 (RDNA 2)** |
| *gfx1030* | RDNA 2 | dGPU | Sienna Cichlid (Navi21) |
| *gfx1031* | RDNA 2 | dGPU | Navy Flounder (Navi22) |
| *gfx1032* | RDNA 2 | dGPU | Dimgrey Cavefish (Navi23) |
| *gfx1033* | RDNA 2 | APU | VanGogh |
| *gfx1034* | RDNA 2 | dGPU | Beige Goby (Navi24) |
| *gfx1035* | RDNA 2 | APU | Yellow Carp (Rembrandt) |

[^gfx940]: [SWDEV-292904 - Extend HIP coherency tests to gfx940 · ROCm-Developer-Tools/HIP@9fbd19a](https://github.com/ROCm-Developer-Tools/HIP/commit/9fbd19a6759b0ed091562ad286a790783998b88a)
[^green_sardine]: [Green Sardine APU の PCI ID が追加、正体は Cezanne APU だったか | Coelacanth's Dream](/posts/2021/01/14/green_sardine-pciid/)
[^gfx1013]: [gfx1013 | Coelacanth's Dream](/tags/gfx1013/)

{{< ref >}}
 * [llvm-project/AMDGPUUsage.rst at main · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/main/llvm/docs/AMDGPUUsage.rst#processors)
 * [common_src_device_info/DeviceInfo.cpp at master · GPUOpen-Tools/common_src_device_info](https://github.com/GPUOpen-Tools/common_src_device_info/blob/master/DeviceInfo.cpp)
 * [ROCm-OpenCL-Runtime/OCLDeviceQueries.cpp at develop · RadeonOpenCompute/ROCm-OpenCL-Runtime](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/blob/develop/tests/ocltst/module/runtime/OCLDeviceQueries.cpp)
 * [[PATCH] drm/amdkfd: Expose GFXIP engine version to sysfs](https://lists.freedesktop.org/archives/amd-gfx/2021-August/067318.html)
{{< /ref >}}

