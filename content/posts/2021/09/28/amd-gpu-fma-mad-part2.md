---
title: "AMD GPU の世代における FMA、MAD 命令の微妙な仕様と違い Part2"
date: 2021-09-28T02:53:53+09:00
draft: false
tags: [ "GFX8", "GFX9", "GFX10", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "AMD", "Database", "GPU", "Note" ]
noindex: false
# summary: ""
---

約 1年前に AMD GPU における、各世代の FMA, MAD 命令の性能について記事を書いたが、最近になってまた調べ直した結果、分かったことがあったため Part2 を書くことにした。  
{{< link >}} [AMD GPU の世代における FMA、MAD 命令の微妙な仕様と違い | Coelacanth's Dream](/posts/2020/09/16/amd-gcn-rdna-fma-mad/) {{< /link >}}

AMD GPU アーキテクチャにおいて、初期の GCN から *Polaris1x /VegaM* までを含む *GFX6-8* では FMA の処理が MAD の 4倍遅く、*Vega, Navi* の世代である *GFX9-10* では FMA も MAD も同じ処理性能であり、*RDNA 2 /GFX10.3* では MAD をサポートせず、MUL と ADD 2つに分けて処理するため遅くなり、FMA を使う方が速くなるとされる。  

 > 		      /*        |---------------------------------- Performance & Availability --------------------------------|
 > 		       *        |MAD/MAC/MADAK/MADMK|MAD_LEGACY|MAC_LEGACY|    FMA     |FMAC/FMAAK/FMAMK|FMA_LEGACY|PK_FMA_F16,|Best choice
 > 		       * Arch   |    F32,F16,F64    | F32,F16  | F32,F16  |F32,F16,F64 |    F32,F16     | F32,F16  |PK_FMAC_F16|F16,F32,F64
 > 		       * ------------------------------------------------------------------------------------------------------------------
 > 		       * gfx6,7 |     1 , - , -     |  1 , -   |  1 , -   |1/4, - ,1/16|     - , -      |  - , -   |   - , -   | - ,MAD,FMA
 > 		       * gfx8   |     1 , 1 , -     |  1 , -   |  - , -   |1/4, 1 ,1/16|     - , -      |  - , -   |   - , -   |MAD,MAD,FMA
 > 		       * gfx9   |     1 , 1 , -     |  1 , -   |  1 , -   | 1 , 1 ,1/16|     - , -      |  - , 1   |   2 , -   |FMA,MAD,FMA
 > 		       * gfx10  |     1 , 1 , -     |  1 , -   |  1 , -   | 1 , 1 ,1/16|     1 , 1      |  - , -   |   2 , 2   |FMA,MAD,FMA
 > 		       * gfx10.3|     - , - , -     |  - , -   |  - , -   | 1 , 1 ,1/16|     1 , 1      |  1 , -   |   2 , 2   |  all FMA
 > 		       *
 > 		       * Tahiti, Hawaii, Carrizo, Vega20: FMA_F32 is full rate, FMA_F64 is 1/4
 > 		       *
 > 		       * gfx8 prefers MAD for F16 because of MAC/MADAK/MADMK.
 > 		       * gfx9 and newer prefer FMA for F16 because of the packed instruction.
 > 		       * gfx10 and older prefer MAD for F32 because of the legacy instruction.
 > 		       */
 >
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/blob/f1284505f0fae78dee2af06e2d8a194d1bc5b442/src/gallium/drivers/radeonsi/si_get.c#L940-955> {{< /quote >}}


{{< pindex >}}
 * [FMA と MAD の性能違い](#perf)
 * [FMA と MAD の精度違い](#ulp)
{{< /pindex >}}

## FMA と MAD の性能違い {#perf}

AMD GPU アーキテクチャのバージョンを示す GPUID と、そのアーキテクチャでサポートしている機能について、LLVM では [llvm-project/AMDGPU.td](https://github.com/llvm/llvm-project/blob/main/llvm/lib/Target/AMDGPU/AMDGPU.td) に記述されている。  
そこで FMA命令が MAD命令 (MUL + ADD) と同速で処理できるかというのは `FeatureFastFMAF32` という機能名で管理されている。  
それを辿ると、上記の *GFX6-8* 世代は MAD の方が遅い、というのはだいぶざっくりとした説明で、実際には *GFX6-8* 世代の一部は FMA と MAD の処理性能が同じであることが分かる。  
*Tahiti (gfx600), Hawaii (gfx702), Hawaii Pro (gfx701), Carrizo (gfx801)* が該当し、それらは同時に FP64演算を FP32 の半分のレートで実行する `HalfRate64Ops` もサポートしている。  
*Hawaii (gfx702)* だけ `HalfRate64Ops` をサポートしていないとしているが、LLVM 内の AMD GPU ユーザーガイド [llvm-project/AMDGPUUsage.rst](https://github.com/llvm/llvm-project/blob/main/llvm/docs/AMDGPUUsage.rst) によると製品によって *Hawaii /Pro* を分けているため、コンシューマ向けでは FP64演算性能を抑えるための方策ではないかと思われる。  
`FeatureFastFMAF32` は *Vega (GFX9)* から世代全体でサポートするようになり、これは上記の説明と一致する。  

そして、*GFX6-8* 世代で `FeatureFastFMAF32` をサポートする AMD GPUアーキテクチャでは、`HalfRate64Ops` も(ほぼ) 同時にサポートするというのが性能に関係すると思われる。  
AMD が公開している AMD GPU の ISA (Instruction Set Architecture) ドキュメントには、FMA命令 `V_FMA_F32` の概要部分に *Only for double-precision parts.* という記述がある。  

 > Fused single-precision multiply-add. Only for double-precision parts.
 >
 > {{< quote >}} [AMD GCN3 Instruction Set Architecture (2016) ](http://developer.amd.com/wordpress/media/2013/12/AMD_GCN3_Instruction_Set_Architecture_rev1.1.pdf) {{< /quote >}}

これが、倍精度 (FP64) 演算ユニットでのみ `V_FMA_F32` が実行可能という意味ならば、`FeatureFastFMAF32` と `HalfRate64Ops` が同時にサポートされることと、コンシューマ向け AMD GPUアーキテクチャでは倍精度演算ユニットを減らしているため、FMA が MAD より遅くなることにも納得がいく。  
ただ 4倍遅い、という具体的な数字については不明。ドキュメント内ではそこまで言及はしていなかった。  
コンシューマ向けでは FP32:FP64 = 16:1 という比率になっているはずだが、どういう実装と計算式なのか。  

*Vega (GFX9)* では単精度演算ユニットでも `V_FMA_F32` を実行可能となるよう改良され、FMA と MAD の処理性能が同じになったのだと思われる。  
また、今まで見落としていたが、Hot Chips 29 (2017) における *Vega アーキテクチャ* の発表でこの点に触れており、*Full rate IEEE compliant FMA32* としてアピールされている。  

 * [AMD’s Radeon Next Generation GPU](https://old.hotchips.org/wp-content/uploads/hc_archives/hc29/HC29.21-Monday-Pub/HC29.21.10-GPU-Gaming-Pub/HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf)

## FMA と MAD の精度違い {#ulp}

FMA も MAD の計算精度について、MAD は乗算の結果を、掛けた値と同じビット数に丸めてから加算の処理を行なうが、 FMA は乗算の結果を掛けた値の倍のビット長で保持してから加算を行なうため、FMA の方が計算精度が高くなる。  
{{< link >}} 参考: [科学技術計算向け演算能力が引き上げられたGPUアーキテクチャ「Fermi」 (2) 科学技術計算向けのさまざまな工夫 | マイナビニュース](https://news.mynavi.jp/article/20091007-nvidia_fermi/2) {{< /link >}}

*Vega (GFX9)* から ISAドキュメントでは FMA と MAD の計算精度、丸め誤差 ULP (Unit in the Last Place) についても記述するようになり、`V_MAD_F32` は 1ULP、`V_FMA_F32` は 0.5ULP としている。  
IEEE-754 では丸め誤差 0.5ULP を正しい結果としており、`V_FMA_F32` がそれに準拠した命令となる。  

{{< ref >}}
 * [ULP，相対誤差，およびマシン・イプシロン](https://jp.xlsoft.com/documents/intel/cvf/vf-html/pg/pg22_02_01_01.htm)
 * [ネイティブ浮動小数点での数値の考慮事項 - MATLAB & Simulink - MathWorks 日本](https://jp.mathworks.com/help/hdlcoder/ug/numerical-considerations-with-native-floating-point.html#bvevu5c-2)
 * GFX6: [AMD_Southern_Islands_Instruction_Set_Architecture](http://developer.amd.com/wordpress/media/2013/07/AMD_Southern_Islands_Instruction_Set_Architecture1.pdf)
 * GFX7: [AMD_Sea_Islands_Instruction_Set_Architecture](http://developer.amd.com/wordpress/media/2013/07/AMD_Sea_Islands_Instruction_Set_Architecture1.pdf)
 * GFX8: [AMD GCN3 Instruction Set Architecture (2016) ](http://developer.amd.com/wordpress/media/2013/12/AMD_GCN3_Instruction_Set_Architecture_rev1.1.pdf)
 * GFX9: [AMD Vega Shader Instruction Set Architecture](https://developer.amd.com/resources/developer-guides-manuals/)
{{< /ref >}}
