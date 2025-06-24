---
title: "LLVM に gfx1250 が \"仮に\" 追加される ――Wave32 に対応して MFMA 系命令をサポートしない CDNA APU? [追記修正]"
date: 2025-06-20T19:57:33+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "LLVM", "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Stanislav Mekhanoshin 氏によって LLVM に新たな GFX ID、*gfx1250* が追加された。  
今のところ、これはただの仮サポート (stub) だとされている。  

 * [[AMDGPU] Initial support for gfx1250 target. by rampitec · Pull Request #144965 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/144965)

だがそれを置いても *gfx1250* は今後の AMD GPU を見る上でかなり興味深い存在である。  

まず、*gfx1250 (gfx12.5.0)* の GFX メジャーバージョンは RDNA 4 系と同じ GFX12 であり、Wave32 に対応しているが、WGP (Work-Group Processor) モードには対応せず、CU モードのみをサポートする。  
RDNA 系アーキテクチャにおける WGP モードと CU モードは主に LDS (Local Data Share) の割り当てに影響し、WGP モードではペアとなる 2基の CU の LDS 64KiB を合わせて割り当てることが可能になるが、CU モードでは常に CU 内の LDS のみに割り当てる。  

また、CDNA 系アーキテクチャ特有のパックド FP32 実行や SRAM ECC に対応する一方で、CDNA 系における行列演算専用命令である MFMA (Matrix fused-multiply-add) には対応せず、RDNA 3/4 が対応してる WMMA (Wave Matrix Multiply Accumulate) には対応している。  
他にもフルレートでの FP64 実行の機能フラグも *gfx1250* には設定されていない。  

RDNA 系アーキテクチャとして見ても、*gfx1250* は画像系命令のサポートやレイトレーシング関連命令のサポートを示す機能フラグが設定されていない。  

CDNA 4 アーキテクチャでは LDS を従来の 64KiB (32バンク) から 160KiB (64バンク) に増やしたが、*gfx1250* は 64KiB (32バンク) のままとされている。  

*gfx1250* は APU とされているが、MI300A/CDNA 3 と MI300X/CDNA 3 のように APU と dGPU のどちらにも対応したアーキテクチャなのかは不明。  

 >         +     ``gfx1250``                 ``amdgcn``   APU                     - Architected                   *TBA*
 >         +                                                                        flat
 >         +                                                                        scratch                       .. TODO::
 >         +                                                                      - Packed
 >         +                                                                        work-item                       Add product
 >         +                                                                        IDs                             names.
 >         +
 >         
 > {{< quote >}} [[AMDGPU] Initial support for gfx1250 target. by rampitec · Pull Request #144965 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/144965) {{< /quote >}}

すべて "現時点の内容では" という言葉が前に付くが、*gfx1250* は CDNA と RDNA を融合したような存在となっている。  
以前に AMD の Jack Huynh 氏はそのようなアーキテクチャとなる UDNA の構想を語っていたが、*gfx1250* がそうなのだろうか？
データセンター向けの GPU アーキテクチャとして考えると、*gfx1250* はフルレート FP64 実行に必要なリソースを減らし、FP32 性能と推論処理性能に特化したものなのかもしれない。  

 * [AMD RDNA と CDNA の違いを考える | Coelacanth's Dream](/posts/2024/11/22/rdna-cdna-udna/)

さらに仮と推測の話となってしまうが、*gfx1250* が UDNA とすると、グラフィクス処理に必要な画像系命令やレイトレーシング関連命令はオプション扱いになるのかもしれない。  

## 追記修正 [2025-06-24]
最初の版では、*gfx1250* は Dual issue、VOPD 命令をサポートしないと書いたが、これは間違いだった。本当に申し訳ない。  
`FeatureGFX12` 内に `FeatureVOPD` が含まれているため、*gfx1250* も VOPD 命令をサポートすると考えられる。  
パックド FP32 実行と VOPD 命令の両方をサポートすることの利点としては、パックド FP32 実行には FMA (`v_pk_fma_f32`) 命令があるが FMAC 命令は含まれず、VOPD 命令には FMAC (`v_dual_fmac_f32`) 命令 があるが FMA 命令が含まれないため、より多くのケースで FP32 演算性能レートを倍に引き上げられると考えられる。  
FMA 命令と FMAC 命令はそれぞれフォーマットが異なり、FMA 命令は `d = a * b + c`、FMAC 命令は `d = a * b + d` のフォーマットとなっている。  

 >         +def FeatureISAVersion12_50 : FeatureSet<
 >         +  [FeatureGFX12,
 >         +   FeatureGFX1250Insts,
 >         +   FeatureCuMode,
 >         +   FeatureLDSBankCount32,
 >         +   FeatureDLInsts,
 >         +   FeatureFmacF64Inst,
 >         +   FeaturePackedFP32Ops,
 >         +   FeatureDot7Insts,
 >         +   FeatureDot8Insts,
 >         +   FeatureWavefrontSize32,
 >         +   FeatureShaderCyclesHiLoRegisters,
 >         +   FeatureArchitectedFlatScratch,
 >         +   FeatureArchitectedSGPRs,
 >         +   FeatureAtomicFaddRtnInsts,
 >         +   FeatureAtomicFaddNoRtnInsts,
 >         +   FeatureAtomicDsPkAdd16Insts,
 >         +   FeatureAtomicFlatPkAdd16Insts,
 >         +   FeatureAtomicBufferGlobalPkAddF16Insts,
 >         +   FeatureAtomicGlobalPkAddBF16Inst,
 >         +   FeatureAtomicBufferPkAddBF16Inst,
 >         +   FeatureFlatAtomicFaddF32Inst,
 >         +   FeatureFP8ConversionInsts,
 >         +   FeaturePackedTID,
 >         +   FeatureVcmpxPermlaneHazard,
 >         +   FeatureSALUFloatInsts,
 >         +   FeaturePseudoScalarTrans,
 >         +   FeatureHasRestrictedSOffset,
 >         +   FeatureScalarDwordx3Loads,
 >         +   FeatureDPPSrc1SGPR,
 >         +   FeatureBitOp3Insts,
 >         +   FeatureBF16ConversionInsts,
 >         +   FeatureCvtPkF16F32Inst,
 >         +   FeatureMinimum3Maximum3PKF16,
 >         +   FeaturePrngInst,
 >         +   FeaturePermlane16Swap,
 >         +   FeatureAshrPkInsts,
 >         +   FeatureSupportsSRAMECC,
 >         +   FeatureMaxHardClauseLength63,
 >         +   FeatureAtomicFMinFMaxF64GlobalInsts,
 >         +   FeatureAtomicFMinFMaxF64FlatInsts,
 >         +   FeatureFlatBufferGlobalAtomicFaddF64Inst,
 >         +   FeatureMemoryAtomicFAddF32DenormalSupport,
 >         +   FeatureKernargPreload,
 >         +   FeatureLshlAddU64Inst,
 >         +]>;
 >         +
 >         
 > {{< quote >}} [[AMDGPU] Initial support for gfx1250 target. by rampitec · Pull Request #144965 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/144965) {{< /quote >}}
