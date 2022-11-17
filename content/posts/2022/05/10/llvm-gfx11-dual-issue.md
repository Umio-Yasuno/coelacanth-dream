---
title: "GFX11 のサポートに向けた LLVM へのさらなるパッチ ―― True16Bit, Dot8, Dual issue wave32"
date: 2022-05-10T12:14:27+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", "RDNA_3", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Joe Nash 氏より、LLVM への *GFX11* のサポートに向けたさらなるパッチが投稿、公開されている。  
{{< link >}} [LLVM で GFX11 のサポートが進み始める | Coelacanth's Dream](/posts/2022/04/28/llvm-gfx11/) {{< /link >}}
今回のパッチには *GFX11* で追加される新機能のサポートも一部含まれている。  
しかし、新機能のフラグ管理と *GFX11* のテストファイル追加が主であり、新機能の詳細、新機能が有効になっていることで分岐する部分のコード等は追加されていない。  
そのため、機能の詳細については推測がほとんどとなる。  

 * [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261)

{{< pindex >}}
 * [FeatureGFX11](#gfx11)
    * [True16Bit](#true16)
    * [Dot8Insts](#dot8)
    * [Dual issue wave32](#vopd)
    * [GPU ID](#gpuid)
{{< /pindex >}}

## FeatureGFX11 {#gfx11}

*GFX11* がサポートする機能、命令の情報が `AMDGPU.td` に追加された。  
*GFX11* での新機能には `FeatureTrue16BitInsts`, `FeatureDot8Insts`, `FeatureVOPD` が挙げられる。  

 > 		+def FeatureGFX11 : GCNSubtargetFeatureGeneration<"GFX11",
 > 		+  "gfx11",
 > 		+  [FeatureFP64, FeatureLocalMemorySize65536, FeatureMIMG_R128,
 > 		+   FeatureFlatAddressSpace, Feature16BitInsts,
 > 		+   FeatureInv2PiInlineImm, FeatureApertureRegs,
 > 		+   FeatureCIInsts, FeatureGFX8Insts, FeatureGFX9Insts, FeatureGFX10Insts,
 > 		+   FeatureGFX10_AEncoding, FeatureGFX10_BEncoding, FeatureGFX10_3Insts,
 > 		+   FeatureGFX11Insts, FeatureVOP3P, FeatureVOPD, FeatureTrue16BitInsts,
 > 		+   FeatureMovrel, FeatureFastFMAF32, FeatureDPP, FeatureIntClamp,
 > 		+   FeatureFlatInstOffsets, FeatureFlatGlobalInsts, FeatureFlatScratchInsts,
 > 		+   FeatureAddNoCarryInsts, FeatureFmaMixInsts,
 > 		+   FeatureNoSdstCMPX, FeatureVscnt,
 > 		+   FeatureVOP3Literal, FeatureDPP8, FeatureExtendedImageInsts,
 > 		+   FeatureNoDataDepHazard, FeaturePkFmacF16Inst,
 > 		+   FeatureGFX10A16, FeatureFastDenormalF32, FeatureG16,
 > 		+   FeatureUnalignedBufferAccess, FeatureUnalignedDSAccess
 > 		+  ]
 > 		+>;
 > 		+
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}

### True16Bit {#true16}

`True16Bit` については `True 16 bit operand instructions` という説明があるのみで、どういった命令がそこに含まれるかは今回明かされていない。  

以前からサポートされている機能として、他に `Feature16BitInsts` があるが、これは i16/f16 フォーマットを扱う命令となる。  
後述の `Dot8Insts` により、*GFX11* では bf16 フォーマットに対応するため、i16/f16 以外の 16-bit 精度フォーマットに対応することを示す機能フラグであることが考えられる。  

 > 		+def FeatureTrue16BitInsts : SubtargetFeature<"true16",
 > 		+  "HasTrue16BitInsts",
 > 		+  "true",
 > 		+  "True 16 bit operand instructions"
 > 		+>;
 > 		+
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}

### Dot8Insts {#dot8}

*GFX11* で新たに `Dot8Insts` をサポートすることは以前のパッチで明かされていたが、今回でそこに含まれる命令が公開された。  
{{< link >}} [LLVM で GFX11 のサポートが進み始める | Coelacanth's Dream](/posts/2022/04/28/llvm-gfx11/) {{< /link >}}
これまで bf16 フォーマットには、*CDNA 系アーキテクチャ* の行列演算命令でしかネイティブでサポートされていなかったが、*GFX11* ではドット積命令の一部でサポートされる。  
また入力と出力の精度を同じにすると思われる `v_dot2_f16_f16, v_dot2_bf16_bf16` が追加されている。 (`D.f16 = S0.f16[0] * S1.f16[0] + S0.f16[1] * S1.f16[1] + S2.f16`)  

 > 		+def FeatureDot8Insts : SubtargetFeature<"dot8-insts",
 > 		+  "HasDot8Insts",
 > 		+  "true",
 > 		+  "Has v_dot2_f16_f16, v_dot2_bf16_bf16, v_dot2_f32_bf16, "
 > 		+  "v_dot4_i32_iu8, v_dot8_i32_iu4 instructions"
 > 		+>;
 > 		+
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}

前回のパッチで *GFX11* は、*GFX10.3/RDNA 2* 世代と比較して、`Dot2Insts` のサポートが外され、`Dot8Insts` が追加されるという形だったが、  
今回のパッチでは、`Dot[1, 2, 6]Insts` が外され、`Dot8Insts` のサポートが追加される形になっている。  
それぞれに含まれる命令は、`Dot1Insts` は `v_dot4_i32_i8, v_dot8_i32_i4` 、`Dot2Insts` は `v_dot2_i32_i16, v_dot2_u32_u16`、`Dot6Insts` は `v_dot4c_i32_i8` となる。  
`Dot1Insts` の範囲については、`Dot8Insts` 中の `v_dot4_i32_iu8, v_dot8_i32_iu4` で置き換え可能と思われる。  

 > 		+def FeatureISAVersion11_Common : FeatureSet<
 > 		+  [FeatureGFX11,
 > 		+   FeatureLDSBankCount32,
 > 		+   FeatureDLInsts,
 > 		+   FeatureDot5Insts,
 > 		+   FeatureDot7Insts,
 > 		+   FeatureDot8Insts,
 > 		+   FeatureNSAEncoding,
 > 		+   FeatureNSAMaxSize5,
 > 		+   FeatureWavefrontSize32,
 > 		+   FeatureShaderCyclesRegister,
 > 		+   FeatureArchitectedFlatScratch,
 > 		+   FeatureAtomicFaddRtnInsts,
 > 		+   FeatureAtomicFaddNoRtnInsts,
 > 		+   FeatureImageInsts,
 > 		+   FeaturePackedTID,
 > 		+   FeatureVcmpxPermlaneHazard]>;
 > 		+
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}

### Dual issue wave32 {#vopd}

*GFX11* では `FeatureVOPD`、`Has VOPD dual issue wave32 instructions` と説明される命令をサポートする。 この機能もまた、それ以上の詳細は今回明かされていない。  
推測するならば、1個の命令から 2x Wave32 分のスレッドを発行する機能と思われるが、ただ SIMDユニットに対応する Wave Controller (Buffer) に 2x Wave32 がスタックされるだけなのか、  
または Wave Controller と SIMDユニットの構成が変更され、一部の `FeatureVOPD` に含まれる命令は従来の 2x レートで処理できるのか、等は不明。  

前者は、*GFX10/RDNA 1* 世代からソフトウェアで実装されている Sub-Vector Mode に近い。Sub-Vector Mode では、Wave64 をベクタレジスタを共有する 2x Wave32 に分解して実行する。  
{{< link >}} [RadeonSI ドライバーでは RDNA GPU も Wave64モードで各シェーダーを実行するように | Coelacanth's Dream](/posts/2020/07/02/radeonsi-shader-wave64-with-rdna/#consider) {{< /link >}}
後者は、Wave Controller に対して SIMDユニットを増やした可能性が考えられる。ただ一部の命令に限られるとすると効率が気になる所ではある。  
CU 内部の構造が大きく変わっている可能性、自分の推測がすべて外れている可能性も高い。  

 > 		+def FeatureVOPD : SubtargetFeature<"vopd",
 > 		+  "HasVOPDInsts",
 > 		+  "true",
 > 		+  "Has VOPD dual issue wave32 instructions"
 > 		+>;
 > 		+
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}

{{< figure src="/image/2020/11/17/amd-cdna-rdna-simd.webp" title="CDNA/RDNA SIMD" caption="画像出典: [amd-cdna-whitepaper.pdf](https://www.amd.com/system/files/documents/amd-cdna-whitepaper.pdf) (Page 6), <br>&emsp; [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) (Page 9)" >}}

### GPU ID {#gpuid}

*gfx1100 (dGPU), gfx1101 (dGPU)* と *gfx1102 (dGPU), gfx1103 (APU)* でサポートする機能は、`FeatureISAVersion11_0` と `FeatureISAVersion11_0_2` で分かれているが、現時点で違いは特にない。  

 > 		+
 > 		+//===----------------------------------------------------------------------===//
 > 		+// GCN GFX11.
 > 		+//===----------------------------------------------------------------------===//
 > 		+
 > 		+def : ProcessorModel<"gfx1100", GFX11SpeedModel,
 > 		+  FeatureISAVersion11_0.Features
 > 		+>;
 > 		+
 > 		+def : ProcessorModel<"gfx1101", GFX11SpeedModel,
 > 		+  FeatureISAVersion11_0.Features
 > 		+>;
 > 		+
 > 		+def : ProcessorModel<"gfx1102", GFX11SpeedModel,
 > 		+  FeatureISAVersion11_0_2.Features
 > 		+>;
 > 		+
 > 		+def : ProcessorModel<"gfx1103", GFX11SpeedModel,
 > 		+  FeatureISAVersion11_0_2.Features
 > 		+>;
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}
 >
 > 		+// Features for GFX 11.0.0 and 11.0.1
 > 		+def FeatureISAVersion11_0 : FeatureSet<
 > 		+  !listconcat(FeatureISAVersion11_Common.Features,
 > 		+    [])>;
 > 		+
 > 		+def FeatureISAVersion11_0_2 : FeatureSet<
 > 		+  !listconcat(FeatureISAVersion11_Common.Features,
 > 		+    [])>;
 > 		+
 >
 > {{< quote >}} [⚙ D125261 [AMDGPU] gfx11 subtarget features & early tests](https://reviews.llvm.org/D125261) {{< /quote >}}

| GC IP ver | GFX ID  |      |
| :-------- | :-----: | :--: |
| 11.0.0    | gfx1100 | dGPU |
| 11.0.1    | gfx1103 | APU  |
| 11.0.2    | gfx1102 | dGPU |
| 11.0.3?   | gfx1101? | dGPU? |

それ以外にも `GCNSubtarget.h` には *GFX11* でサポートされる機能フラグに関するコードが追加されている。  

 > 		+  bool hasVOP3DPP() const { return getGeneration() >= GFX11; }
 > 		+
 > 		+  bool hasLdsDirect() const { return getGeneration() >= GFX11; }
 > 		+
 > 		+  bool hasVALUPartialForwardingHazard() const {
 > 		+    return getGeneration() >= GFX11;
 > 		+  }
 > 		+
 > 		+  bool hasVALUTransUseHazard() const { return getGeneration() >= GFX11; }
 > 		+

*GFX11* では *NGG (Next Generation Geometry)* パイプラインのみをサポートし、レガシーなパイプラインはサポートしない、というのは *RadeonSI* ドライバーの内容と一致している。  
{{< link >}} [RadeonSI ドライバーで GFX11 のサポートが進められる | Coelacanth's Dream](/posts/2022/05/05/radeonsi-gfx11/) {{< /link >}}

 > 		+  // \returns true if the target supports the pre-NGG legacy geometry path.
 > 		+  bool hasLegacyGeometry() const { return getGeneration() < GFX11; }
 > 		+

{{< ref >}}
 * [【後藤弘茂のWeekly海外ニュース】Armの新世代フラグシップGPU「Mali-G76」 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1125383.html)
{{< /ref >}}
