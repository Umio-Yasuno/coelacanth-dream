---
title: "AMD Instinct MI200/CDNA2 Instruction Set Architecture が公開される"
date: 2021-11-20T13:47:42+09:00
draft: false
tags: [ "CDNA_2", "gfx90a", "Aldebaran", "MI200" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

*AMD Instinct MI200 シリーズ* (コードネーム: [Aldebaran](/tags/aldebaran)) の ISA (命令セット) ドキュメント、*AMD Instinct MI200/CDNA2 Instruction Set Architecture* が公開された。  

 * [Developer Guides, Manuals & ISA Documents - AMD](https://developer.amd.com/resources/developer-guides-manuals/)
    * ["AMD Instinct MI200" Instruction Set Architecture: Reference Guide - CDNA2_Shader_ISA_18November2021.pdf](https://developer.amd.com/wp-content/resources/CDNA2_Shader_ISA_18November2021.pdf)

*CDNA 2/MI200/Aldebaran* の ISA での大きな変更には、以下があげられる。  

 * DPP (Data Parallel Primitives/Processeing) が 64-bit データタイプに対応
    * DPP は 1つの Wavefront の中で異なるスレッド間でデータを交換する機能
 * Float64 におけるアトミック操作に対応
 * 通常のベクタレジスタ (ArchVGPR) と、MFMA命令を実行する専用の SIMDユニットが持つベクタレジスタ (AccVGPR) が統合
 * AccVGPR へのメモリ操作を直接行うことが可能に
 * GDS (Global Data Share) の削除
 * 一部の `IMAGE` , `GATHER` 命令を削除
 * 一部 MFMA (Matrix-Fused-Multiply-Add) 命令の追加
  * `V_MFMA_F32_{4x4x4, 16x16x4, 16x16x16, 32x32x4, 32x32x8, 16x16x16}BF16_1K`
  * `V_MFMA_F64_{16x16x4f64, 4x4x4f64 }`
 * FP32 Packed Math 命令の追加
  * `V_PK_{FMA,MUL,ADD,MOV}_F32`


ベクタレジスタの統合については、*CDNA 1/MI100/Arcturus* では ArchVGPR と AccVGPR が分かれており、メモリとそのキャッシュ、LDS (Local Data Share) への入出力には ArchVGPR のみが対応していた。そのため、常に AccVGPR に出力される MFMA 命令の結果は、ArchVGPR を経由してメモリ、または LDS に格納する必要があった。  
その手間が減らされるのと同時に、統合されたベクタレジスタは ArchVGPR と AccVGPR はオフセットを指定して分割使用する形になるが、非対称的に分割可能であるため、スレッドに割り当てる ArchVGPR を *CDNA 1/MI100/Arcturus* より多くすることもできる。  
{{< link >}} [CDNA 2 アーキテクチャでは Unified Register Files を実装 | Coelacanth's Dream](/posts/2021/07/01/aldebaran-unified-vgpr/) {{< /link >}}
こうした改善は実行性能の向上に寄与していると思われる。  

GDS の削除は、AMDGPU ドライバーで *Aldebaran* をサポートする最初のパッチで触れられていた。  
{{< link >}} [Linux Kernel に 「Aldebaran」 GPU をサポートするパッチが投稿される　―― CDNA 2/MI200? | Coelacanth's Dream](/posts/2021/02/25/amd-aldebaran-gpu/#removed-gds) {{< /link >}}
GDS への変更は *CDNA 1/MI100/Arcturus* の時点で行われており、*Vega10/Vega20* では GDS は 64KB (32バンク) だったのが [^gds-size]、*CDNA 1/MI100/Arcturus* では 4KB (4バンク) に減らされている。[^arct-gds]  
GDS はすべての CU からアクセス可能なオンチップメモリであり、1つの GPU に搭載される CU数が増えたことで、性能の確保と実装コストが高く付くようになったのかもしれない。  

LDS について、FP64性能の引き上げに伴ってエントリあたり 8 bytes (64-bit) に拡張されたのではないかと考えていたが、ドキュメントによると 64 KB (32 [banks] x 512 [entries] x 4 [bytes per entries]) のままとされていた。  

[^gds-size]: [Vega_7nm_Shader_ISA_26November2019.pdf](https://gpuopen.com/wp-content/uploads/2019/11/Vega_7nm_Shader_ISA_26November2019.pdf) ["AMD Instinct MI100" Instruction Set Architecture: Reference Guide - CDNA1_Shader_ISA_14December2020.pdf](https://developer.amd.com/wp-content/resources/CDNA1_Shader_ISA_14December2020.pdf)
[^arct-gds]: [[PATCH 089/102] drm/amdgpu: init gds config for arct](https://lists.freedesktop.org/archives/amd-gfx/2019-July/036836.html)

{{< ref >}}
 * ["AMD Instinct MI100" Instruction Set Architecture: Reference Guide - CDNA1_Shader_ISA_14December2020.pdf](https://developer.amd.com/wp-content/resources/CDNA1_Shader_ISA_14December2020.pdf)
 * ["AMD Instinct MI200" Instruction Set Architecture: Reference Guide - CDNA2_Shader_ISA_18November2021.pdf](https://developer.amd.com/wp-content/resources/CDNA2_Shader_ISA_18November2021.pdf)
{{< /ref >}}
