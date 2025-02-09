---
title: "RDNA 4 における WMMA 命令と改善"
date: 2025-02-09T10:12:17+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "RDNA_4", "GFX12", "RDNA_3", "GFX11" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

*RDNA 3/GFX11 アーキテクチャ* からサポートされている `WMMA (Wave Matrix Multiply-accumulate)` 命令だが、*RDNA 4/GFX12 アーキテクチャ* でも引き続きサポートしており、そして対応フォーマットの追加や対応する行列レイアウトの追加、必要なベクタレジスタサイズの削減などが実装されている。  

## 使用ベクタレジスタの削減 {#vgpr}
*RDNA 3/GFX11* における `WMMA` 命令は内部的に複数のドット積命令に分解して実行する方式となっていた。  
また、性能を最大化するには積算に使われる入力行列 A, B の配置を工夫する必要があり、thread/work-item/vector-lane 0-15 のデータを 16-31 にも複製する必要があった。これは Wave32 モードの場合であり、Wave64 の場合は 0-31 のデータを 32-63 に複製する必要がある。  
*RDNA 3/GFX11* では 32-bit VGPR (ベクタレジスタ) を 2つの 16-bit VGPR に分割する機能 (16-bit VGPR-pairs) を持ち、また `WMMA` 命令は入力行列 A, B に F16/BF16/IU8 のフォーマットのみをサポートしている。  
そのため、理論上は 16x16 の行列は 4 VGPR (4 [vgpr]\* 2 [16-bit pair] \* 32 [lane]) に収まるはずだが、先の理由のため 8 VGPR を割り当てる必要があった。  
また、アキュムレータとなる行列 C も出力フォーマットが F16/BF16 の場合でも 8 VGPR を必要とする。  
これは `WMMA` 命令では設定したフラグによって 32-bit VGPR の上半分と下半分のどちらに結果を格納するか決まるためとされている。  

それが *RDNA 4/GFX12* では一部データを複製する必要がなくなり、行列 A, B が 4 VGPR (Wave32) に収まるようになった。  
また、結果の格納に関するフラグを設定する必要もなくなり、行列 C も 4 VGPR に収まるようになった。  

 >         +/* This pass lowers cooperative matrix.
 >         + *
 >         + * On GFX11, the A&B matrices needs to be replicated, lanes 0..15 are replicated
 >         + * to 16..31 and for wave64 also into lanes 32..47 and 48..63. A&B matrices are
 >         + * always vectors of 16 elements.
 >         + *
 >         + * On GFX12, there is no data replication and the matrices layout is described
 >         + * as below:
 >         + *
 >         + * Wave32:
 >         + * A&B:
 >         + *         0..15  | 16..31 (lanes)
 >         + * v0 lo:  row 0  | row 4
 >         + * v0 hi:  row 1  | row 5
 >         + * v1 lo:  row 2  | row 6
 >         + * v1 hi:  row 3  | row 7
 >         + * v2 lo:  row 8  | row 12
 >         + * v2 hi:  row 9  | row 13
 >         + * v3 lo:  row 10 | row 14
 >         + * v3 hi:  row 11 | row 15
 >         + *
 >         + * C:
 >         + *         0..15  | 16..31 (lanes)
 >         + * v0 lo:  row 0  | row 8
 >         + * v0 hi:  row 1  | row 9
 >         + * v1 lo:  row 2  | row 10
 >         + * v1 hi:  row 3  | row 11
 >         + * v2 lo:  row 4  | row 12
 >         + * v2 hi:  row 5  | row 13
 >         + * v3 lo:  row 6  | row 14
 >         + * v3 hi:  row 7  | row 15
 >         + *
 >         + * Wave64:
 >         + * A&B:
 >         + *         0..15 | 16..31 | 32..47 | 48..63 (lanes)
 >         + * v0 lo:  row 0 | row 4  | row 8  | row 12
 >         + * v0 hi:  row 1 | row 5  | row 9  | row 13
 >         + * v1 lo:  row 2 | row 6  | row 10 | row 14
 >         + * v1 hi:  row 3 | row 7  | row 11 | row 15
 >         + *
 >         + * C:
 >         + *         0..15 | 16..31 | 32..47 | 48..63 (lanes)
 >         + * v0 lo:  row 0 | row 8  | row 4  | row 12
 >         + * v0 hi:  row 1 | row 9  | row 5  | row 13
 >         + * v1 lo:  row 2 | row 10 | row 6  | row 14
 >         + * v1 hi:  row 3 | row 11 | row 7  | row 15
 >         + */
 >
 > {{< quote >}} [radv: add support for VK_KHR_cooperative_matrix on GFX12 (!33378) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33378) {{< /quote >}}

## 対応フォーマットとレイアウト {#format-layout}
*RDNA 3/GFX11* では `WMMA` 命令の入力フォーマットに F16/BF16/IU8/IU4、出力フォーマットに F32/F16/BF16/I32 をサポートし、行列のレイアウトは入出力ともに 16x16 のみをサポートしていた。  
*RDNA 4/GFX12* では入力フォーマットに FP8 (1 bit sign, 4 bit exponent, 3 bit mantissa) と BF8 (1 bit sign, 5 bit exponent, 2 bit mantissa) のサポートが追加され、  
入力フォーマットが IU4 の場合は 16x16x32 (A: 16x32, B: 32x16, C: 16x16) のレイアウトもサポートする。  

また、疎行列を入力に取る `SWMMAC (Wave Matrix(sparse) Multiply-Accumulate)` 命令のサポートが追加された。  
`SWMMAC` 命令の対応フォーマットは `WMMA` 命令と同じであり、行列のレイアウトは 16x16x32 (A: sparse 32x16, B: 16x32, C: 16x16) と、入力フォーマットが IU4 の場合は 16x16x64 (A: sparse 16x64, B: 64x16, C: 16x16) をサポートする。  

*RDNA 4/GFX12* でも *RDNA 3/GFX11* の内部的に複数のドット積命令に分解して発行する形式を継続するのか、それとも *CDNA* 系のように専用の行列演算ユニットを実装するのかまだ不明である。  
*RDNA 3/GFX11* から行列演算の IPC が向上しているのかについても、*RDNA 4/GFX12* におけるサイクル数が公開されていないため不明。  
しかし、モデルのサイズや VRAM の使用サイズの削減に役立つ FP8/BF8 フォーマットのサポート、疎行列のサポート、必要な VGPR の削減と *RDNA 3/GFX11* から進化しているのは確かである。  

{{< ref >}}
 * [radv: add support for VK_KHR_cooperative_matrix on GFX12 (!33378) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33378)
 * [How to accelerate AI applications on RDNA 3 using WMMA - AMD GPUOpen](https://gpuopen.com/learn/wmma_on_rdna3/)
 * ["RDNA3" Instruction Set Architecture: Reference Guide - rdna3-shader-instruction-set-architecture-feb-2023_0.pdf](https://www.amd.com/content/dam/amd/en/documents/radeon-tech-docs/instruction-set-architectures/rdna3-shader-instruction-set-architecture-feb-2023_0.pdf)
 * ["RDNA3.5" Instruction Set Architecture: Reference Guide - rdna35_instruction_set_architecture.pdf](https://www.amd.com/content/dam/amd/en/documents/radeon-tech-docs/instruction-set-architectures/rdna35_instruction_set_architecture.pdf)
 * ["AMD Instinct MI300" Instruction Set Architecture: Reference Guide - amd-instinct-mi300-cdna3-instruction-set-architecture.pdf](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/instruction-set-architectures/amd-instinct-mi300-cdna3-instruction-set-architecture.pdf)
 * [Pipeline descriptions — ROCm Compute Profiler 3.0.0 documentation](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/latest/conceptual/pipeline-descriptions.html)
 * [FP8 Numbers — HIP 6.3.42134 Documentation](https://rocm.docs.amd.com/projects/HIP/en/latest/reference/fp8_numbers.html)
{{< /ref >}}
