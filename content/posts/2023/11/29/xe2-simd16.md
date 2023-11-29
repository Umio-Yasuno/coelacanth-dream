---
title: "計算ユニットが SIMD16 になる Intel Xe2 アーキテクチャ"
date: 2023-11-29T16:41:04+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Xe2", "Xe2-LPG", "Lunar_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Mesa3D, Intel Graphics Compiler (IGC), oneDNN では *Xe2 アーキテクチャ*、*Lunar Lake* のサポートが現在進められており、*Xe2 アーキテクチャ* が詳細が徐々に公開されてきている。  

 * [src: gpu: added Xe2 architecture support · oneapi-src/oneDNN@c1f86dc](https://github.com/oneapi-src/oneDNN/commit/c1f86dc7d87b6ad771db39d5e2c4dce86e396694)

## SIMD16
*Xe2 アーキテクチャ* では *Xe-HPC* と同様に EU (Xe Vector Engine) 内の計算ユニットが SIMD16 となる。  

*Xe-LP, Xe-HP, Xe-HPG, Xe-LPG* は 2基の EU で Thread Control を共有し、それらをペアとすることで SIMD16 を実行可能としていた。  
コンパイラ内では EU 2基をペアとすることを EU Fusion または FusedEU、ペアとなる EU 8基 (EU 16基) を持つ Subslice を Dual Subslice と呼んでいた。  
それが *Xe-HPC* の EU と Subslice ではその構成を引き継がず、EU 内の計算ユニットを SIMD16 とし、Subslice あたりの EU は 8基の構成を採った。  
IGC 内の `hasFusedEU` の条件に `Xe2` がないため、*Xe2* も *Xe-HPC* と同様に EU 2基をペアとする機能を持たない構成と思われる。  

 >         bool hasFusedEU() const {
 >           return (getPlatform() == GENX_TGLLP || getPlatform() == Xe_XeHPSDV ||
 >                   getPlatform() == Xe_DG2 || getPlatform() == Xe_MTL ||
 >                   getPlatform() == Xe_ARL);
 >         }
 >
 > {{< quote >}} [intel-graphics-compiler/visa/HWCaps.inc at 2998e867d91dec7198b77aa589e82e65c26ad45f · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2998e867d91dec7198b77aa589e82e65c26ad45f/visa/HWCaps.inc#L616C1-L620C2) {{< /quote >}}

*Xe2 アーキテクチャ* は EU 内の実行ポートも *Xe-HPC* にかなり近いものになっていると思われ、開発にあたって *Xe-HPG* ではなく *Xe-HPC* をベースにしたと考えられる。[^reg]  

[^reg]: [レジスタファイルサイズが倍になり、Int64 のサポートが復活する Intel Xe2 | Coelacanth's Dream](/posts/2023/10/07/xe2-grf-long-pipe/)

 >             int hw_simd() const {
 >                 switch (hw) {
 >                     case ngen::HW::Gen9:
 >                     case ngen::HW::Gen10:
 >                     case ngen::HW::Gen11:
 >                     case ngen::HW::XeLP:
 >                     case ngen::HW::XeHP:
 >                     case ngen::HW::XeHPG: return 8;
 >                     case ngen::HW::Xe2:
 >                     case ngen::HW::XeHPC: return 16;
 >                     default: ir_error_not_expected();
 >                 }
 >                 return -1;
 >             }
 >
 > {{< quote >}} [oneDNN/src/gpu/jit/codegen/bank_conflict_allocation.cpp at c1f86dc7d87b6ad771db39d5e2c4dce86e396694 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/c1f86dc7d87b6ad771db39d5e2c4dce86e396694/src/gpu/jit/codegen/bank_conflict_allocation.cpp#L83-L96) {{< /quote >}}
 >
 >         // EU native execution size for 32-bit types
 >         G4_ExecSize getNativeExecSize() const {
 >           return getPlatform() >= Xe_PVC ? g4::SIMD16 : g4::SIMD8;
 >         }
 >
 > {{< quote >}} [intel-graphics-compiler/visa/HWCaps.inc at 2998e867d91dec7198b77aa589e82e65c26ad45f · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2998e867d91dec7198b77aa589e82e65c26ad45f/visa/HWCaps.inc#L469-L472) {{< /quote >}}

前述したように *Xe-LP, Xe-HP, Xe-HPG, Xe-LPG* はペアとなる EU 2基で SIMD16 が実行可能であることから、*Xe2* での SIMD16 化による性能の影響は自分では測れない。  
*Xe2* ではレジスタエントリ数も従来の倍となる 256エントリとなるため、SIMD幅に対するレジスタファイル数が減るということもない。  

*Xe-LPG* EU における FP32 演算性能に対する FP64 演算性能は 1/16 とされている。[^xe-lpg]  
EU (FP+INT/EM/FP64) 2基でその性能であったため、ペア構成をやめて SIMD16 化した *Xe2* では、FP64 演算をサポートしつつ、さらに FP64 演算性能を減らすことが可能になるとは思われる。  

ちなみに最近の GPU アーキテクチャにおける FP64 演算性能は以下のようになっている。  

| FP64 peak perf per FP32 peak perf | |
| :-- | :--: |
| AMD RDNA 2 | 1/16 |
| AMD RDNA 3 | 1/32[^w7600] |
| AMD CDNA 2 | 1 (Vector) |
| Intel Xe-HPG | N/A |
| Intel Xe-LPG | 1/16 |
| Intel Xe-HPC | 1[^xe-hpc-fp64] |
| NVIDIA AD102 | 1/64[^ad102] |
| NVIDIA H100/H200 | 1/2 (Vector)[^h100] |

[^w7600]: <https://www.amd.com/system/files/documents/radeon-pro-w7600-datasheet.pdf>
[^ad102]: <https://images.nvidia.com/aem-dam/Solutions/geforce/ada/nvidia-ada-gpu-architecture.pdf>
[^h100]: [H100 Tensor Core GPU | NVIDIA](https://www.nvidia.com/en-us/data-center/h100/)
[^xe-hpc-fp64]: [Intel Data Center GPU Max Series Overview](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-data-center-gpu-max-series-overview.html)

[^xe-lpg]: [Graphics and Media Deep Dive](https://www.intel.com/content/www/us/en/content-details/788848/graphics-and-media-deep-dive.html)

{{< ref >}}
 * [oneAPI GPU Optimization Guide](https://www.intel.com/content/www/us/en/docs/oneapi/optimization-guide-gpu/2023-2/overview.html)
{{< /ref >}}
