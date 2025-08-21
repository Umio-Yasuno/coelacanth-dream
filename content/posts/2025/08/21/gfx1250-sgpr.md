---
title: "GFX1250 では User SGPR が 32個に"
date: 2025-08-21T05:17:18+09:00
draft: false
categories: [ "AMD", "GPU", "APU" ]
tags: [ "GFX12.5", "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

次世代の CDNA 系 APU *gfx1250* にて、User SGPR (Scalar general-purpose registers) が従来の 16個から 32個に増加することが明らかになった。  

 * [[AMDGPU] User SGPR count increased to 32 on gfx1250 by rampitec · Pull Request #154205 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/154205)

AMD GPU における SGPR とは Wave ごとに割り当てられ、保持されるレジスタであり、SIMD ユニット全体で共有するような定数やメモリアドレス、ディスクリプタを格納されるのに有用とされている。  
User SGPR は Wave の起動前に値がセットされる SGPR を指す。  
また、CDNA 1 を除く CDNA 系の AMD GPU はカーネル引数を User SGPR にプリロードする機能をサポートしている。  
*gfx1250* で User SGPR の数が 32個に増えたことには、そのプリロード機能をさらに活用する狙いがあるのかもしれない。  

{{< ref >}}
 * <https://github.com/llvm/llvm-project/blob/main/llvm/docs/AMDGPUUsage.rst#preloaded-kernel-arguments>
 * [PowerPoint Presentation - AMD_Graphics_pipeline_GIC2020.pdf](https://gpuopen.com/download/AMD_Graphics_pipeline_GIC2020.pdf)
 * [PowerPoint Presentation - VulkanFastPaths.pdf](https://gpuopen.com/download/VulkanFastPaths.pdf)
 * [PowerPoint Presentation - GDC2017-Advanced-Shader-Programming-On-GCN.pdf](https://gpuopen.com/download/GDC2017-Advanced-Shader-Programming-On-GCN.pdf)
{{< /ref >}}
