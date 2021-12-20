---
title: "RadeonSIドライバーに FMA32命令を強制するオプションが追加"
date: 2021-12-18T13:18:20+09:00
draft: false
tags: [ "RadeonSI", "GFX9", "GFX10", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

これまでに取り上げた AMDGPU における FMA、MAD命令の違いが関係する話。  
{{< link >}} [AMD GPU の世代における FMA、MAD 命令の微妙な仕様と違い | Coelacanth's Dream](/posts/2020/09/16/amd-gcn-rdna-fma-mad/) {{< /link >}}
{{< link >}} [AMD GPU の世代における FMA、MAD 命令の微妙な仕様と違い Part2 | Coelacanth's Dream](/posts/2021/09/28/amd-gpu-fma-mad-part2/) {{< /link >}}

タイトルの通り、**RadeonSI (OpenGL)** ドライバーに FMA32命令を強制的に使用するオプションが追加された。  

 * [driconf: support META application (!13686) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13686)

{{< pindex >}}
 * [性能](#perf)
 * [丸め誤差 ULP](#ulp)
{{< /pindex >}}

## 性能 {#perf}

段階を踏んで解説すると、AMD GPU では *GFX6-8* の世代だと FMA、MAD命令の処理性能に違いがあり、FMA命令は MAD命令の 1/4 という性能だった。  
*GFX9 (Vega)* の世代からは FMA命令が MAD命令と同等の処理性能となり、*GFX10.3 (RDNA 2)* の世代では MAD命令が取り除かれ、FMA命令に置き換えられた。  
ドライバー、シェーダーコンパイラでは、*GFX6-8* では MAD命令を、*GFX9-10* ではどちらでも性能は変わらないが MAD命令を、*GFX10.3* では FMA命令を使う形で対応していた。*GFX10.3* の場合、MAD命令だと MUL+ADD に分けて処理するため、逆に遅くなる。  

一応、*GFX6-8* 世代でも倍精度演算への対応が強化されている一部 GPU では、FMA、MAD命令の性能が同等となっており、*Tahiti (gfx600), Hawaii (gfx702), Hawaii Pro (gfx701), Carrizo (gfx801)* が該当するが、ドライバーとしては対応を簡潔にするために、上のような形にしていると思われる。  


 > 		      /*        |---------------------------------- Performance & Availability --------------------------------|
 > 		       *        |MAD/MAC/MADAK/MADMK|MAD_LEGACY|MAC_LEGACY|    FMA     |FMAC/FMAAK/FMAMK|FMA_LEGACY|PK_FMA_F16,|Best choice
 > 		       * Arch   |    F32,F16,F64    | F32,F16  | F32,F16  |F32,F16,F64 |    F32,F16     | F32,F16  |PK_FMAC_F16|F16,F32,F64
 > 		       * ------------------------------------------------------------------------------------------------------------------
 > 		       * gfx6,7 |     1 , - , -     |  1 , -   |  1 , -   |1/4, - ,1/16|     - , -      |  - , -   |   - , -   | - ,MAD,FMA
 > 		       * gfx8   |     1 , 1 , -     |  1 , -   |  - , -   |1/4, 1 ,1/16|     - , -      |  - , -   |   - , -   |MAD,MAD,FMA
 > 		       * gfx9   |     1 ,1|0, -     |  1 , -   |  - , -   | 1 , 1 ,1/16|    0|1, -      |  - , 1   |   2 , -   |FMA,MAD,FMA
 > 		       * gfx10  |     1 , - , -     |  1 , -   |  1 , -   | 1 , 1 ,1/16|     1 , 1      |  - , -   |   2 , 2   |FMA,MAD,FMA
 > 		       * gfx10.3|     - , - , -     |  - , -   |  - , -   | 1 , 1 ,1/16|     1 , 1      |  1 , -   |   2 , 2   |  all FMA
 > 		       *
 > 		       * Tahiti, Hawaii, Carrizo, Vega20: FMA_F32 is full rate, FMA_F64 is 1/4
 > 		       * gfx9 supports MAD_F16 only on Vega10, Raven, Raven2, Renoir.
 > 		       * gfx9 supports FMAC_F32 only on Vega20, but doesn't support FMAAK and FMAMK.
 > 		       *
 > 		       * gfx8 prefers MAD for F16 because of MAC/MADAK/MADMK.
 > 		       * gfx9 and newer prefer FMA for F16 because of the packed instruction.
 > 		       * gfx10 and older prefer MAD for F32 because of the legacy instruction.
 > 		       */
 >
 > {{< quote >}} [src/gallium/drivers/radeonsi/si_get.c · 9ff086052ab7bff3cb55c06365543190a3afe188 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/9ff086052ab7bff3cb55c06365543190a3afe188/src/gallium/drivers/radeonsi/si_get.c#L1017-1034) {{< /quote >}}

## 丸め誤差 ULP {#ulp}

AMD GPU における FMA、MAD命令の性能以外での違いには計算精度、丸め誤差 ULP (Unit in the Last Place) の違いが挙げられる。  
FMA命令では 0.5ULP、MAD命令では 1ULP が保証されており、FMA命令の方が精度が高くなっている。  
今回 FMA命令を強制するオプションが追加されたのは、この計算精度のためで、該当マージリークエストによれば、精度の違いにより結果の最後のビットが異なることがある。  
シミュレーションによる試作、試験のポストプロセッシングツール [META](https://www.ansa-usa.com/software/meta/) では精度の違いが赤いブロックとして表れ、FMA、MAD命令で目に見える結果も異なってしまう。  

RadeonSIドライバーでは、FMA命令を強制するオプション (`radeonsi_force_use_fma32`) が METAアプリケーションに対してはデフォルトで有効化される。  
またオプションの影響を受ける AMDGPU の世代は、FMA、MAD命令で性能が変わらない *GFX9-10* のみで、性能に差が大きくある *GFX6-8* ではそのまま MAD命令が使われる。  

 > 		+   /* fma32 is too slow for gpu < gfx9, so force it only when gpu >= gfx9 */
 > 		+   bool force_fma32 =
 > 		+      sscreen->info.chip_class >= GFX9 && sscreen->options.force_use_fma32;
 > 		+
 >
 > {{< quote >}} [driconf: support META application (!13686) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13686) {{< /quote >}}

{{< ref >}}
 * [META | BETA CAE Systems USA, Inc.](https://www.ansa-usa.com/software/meta/)
 * [META｜製品紹介｜BETA CAE Systems Japan](https://beta-cae.jp/products/eta/featured/nvh.html)
{{< /ref >}}
