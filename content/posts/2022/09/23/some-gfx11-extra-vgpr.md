---
title: "追加のベクタレジスタを持つ Navi31/GFX1100 と Navi32/GFX1101"
date: 2022-09-23T19:39:39+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Jay Foad 氏により公開された LLVM へのパッチから、*RDNA 3/GFX11* のいくつかは通常の *RDNA 1/2/3* より 50% 大きい物理ベクタレジスタ (VGPR, Vector general-perpose register) を持つことが明らかにされた。  

 * [⚙ D134522 [AMDGPU] Add GFX11 feature for subtargets with extra VGPRs](https://reviews.llvm.org/D134522)

 >         Some GFX11 subtargets have 50% extra physical VGPRs compared to standard
 >         GFX10/GFX11. This affects occupancy calculations.
 > 
 > {{< quote >}} [⚙ D134522 [AMDGPU] Add GFX11 feature for subtargets with extra VGPRs](https://reviews.llvm.org/D134522) {{< /quote >}}
 >
 >         +def FeatureGFX11ExtraVGPRs : SubtargetFeature<"gfx11-extra-vgprs",
 >         +  "HasGFX11ExtraVGPRs",
 >         +  "true",
 >         +  "GFX11 with 50% extra VGPRs and 50% larger allocation granule"
 >         +>;
 >         +
 >
 > {{< quote >}} [⚙ D134522 [AMDGPU] Add GFX11 feature for subtargets with extra VGPRs](https://reviews.llvm.org/D134522) {{< /quote >}}

## Extra VGPR {#extra}
追加のベクタレジスタを持つことを示す `FeatureGFX11ExtraVGPRs` フラグは *Navi31/gfx1100, Navi32/gfx1101* に追加されている。一方、*Navi32/gfx1102* と *Phoenix APU/gfx1103* には追加されていない。  

 >          def FeatureISAVersion11_0_0 : FeatureSet<
 >            !listconcat(FeatureISAVersion11_Common.Features,
 >         -    [FeatureUserSGPRInit16Bug])>;
 >         +    [FeatureGFX11ExtraVGPRs,
 >         +     FeatureUserSGPRInit16Bug])>;
 >         
 >          def FeatureISAVersion11_0_1 : FeatureSet<
 >            !listconcat(FeatureISAVersion11_Common.Features,
 >         -    [])>;
 >         +    [FeatureGFX11ExtraVGPRs])>;
 >         
 >         
 >
 > {{< quote >}} [⚙ D134522 [AMDGPU] Add GFX11 feature for subtargets with extra VGPRs](https://reviews.llvm.org/D134522) {{< /quote >}}

*RDNA 1/2 アーキテクチャ* のベクタレジスタファイルのサイズは、SIMDユニットあたり 128KiB となっている。  
128KiB の内訳はベクタレジスタ幅 32-bit、レーンあたり 1024本、そして SIMD32 であるから 32x1024x32/8 となる。  

今回 Jay Foad 氏より公開されたパッチを見るに、*Navi31/gfx1100* と *Navi32/gfx1101* ではレーンあたりのベクタレジスタが +50% 増やされ、1536本 (Wave32) となっている。  
ベクタレジスタ幅に変更がなければ、*Navi31/gfx1100* と *Navi32/gfx1101* は SIMDユニットあたり 192KiB のベクタレジスタファイルを持つと考えられる。  

 >         unsigned getTotalNumVGPRs(const MCSubtargetInfo *STI) {
 >           if (STI->getFeatureBits().test(FeatureGFX90AInsts))
 >             return 512;
 >           if (!isGFX10Plus(*STI))
 >             return 256;
 >           bool IsWave32 = STI->getFeatureBits().test(FeatureWavefrontSize32);
 >           if (STI->getFeatureBits().test(FeatureGFX11ExtraVGPRs))
 >             return IsWave32 ? 1536 : 768;
 >           return IsWave32 ? 1024 : 512;
 >         }
 >
 > {{< quote >}} [⚙ D134522 [AMDGPU] Add GFX11 feature for subtargets with extra VGPRs](https://reviews.llvm.org/D134522) {{< /quote >}}

ベクタレジスタの割り当ての粒度も変更され、`FeatureGFX11ExtraVGPRs` が有効な場合は 24 (Wave32) とされている。  
割り当て粒度は *RDNA 1/GFX10.1* では 8 (Wave32)、*RDNA 2/GFX10.3* では 16 (Wave32) と毎世代変更されている。  
アドレス可能なベクタレジスタは 256 (v0-v255) のまま保たれているため、命令のフォーマットに大きな変更は無く、互換性は保たれている。  

 >         unsigned getVGPRAllocGranule(const MCSubtargetInfo *STI,
 >                                      Optional<bool> EnableWavefrontSize32) {
 >           if (STI->getFeatureBits().test(FeatureGFX90AInsts))
 >             return 8;
 >         
 >           bool IsWave32 = EnableWavefrontSize32 ?
 >               *EnableWavefrontSize32 :
 >               STI->getFeatureBits().test(FeatureWavefrontSize32);
 >         
 >           if (STI->getFeatureBits().test(FeatureGFX11ExtraVGPRs))
 >             return IsWave32 ? 24 : 12;
 >         
 >           if (hasGFX10_3Insts(*STI))
 >             return IsWave32 ? 16 : 8;
 >         
 >           return IsWave32 ? 8 : 4;
 >         }
 >
 > {{< quote >}} [⚙ D134522 [AMDGPU] Add GFX11 feature for subtargets with extra VGPRs](https://reviews.llvm.org/D134522) {{< /quote >}}

ベクタレジスタを増やすことで、レジスタ溢れが発生してキャッシュへの退避が必要になるケースを減らすことができる。  

*CDNA 1 アーキテクチャ* では `MFMA (Matrix-Fused-Multiply-Add)` 命令専用の SIMDユニット (miSIMD) と一緒に専用の AccVGPR (Accumulation VGPR) を追加し、*CDNA 2 アーキテクチャ* では通常の VGPR と AccVGPR を統合した。  
*RDNA 3/GFX11* では `MFMA` 命令とは別の行列積和演算命令、`WMMA (Wave Matrix Multiply-accumulate)` 命令に対応するが、*CDNA 1/2 アーキテクチャ* のように専用のレジスタファイルは追加していない。  
*Navi31/gfx1100* と *Navi32/gfx1101* のみではあるが、レーンあたりのレジスタ本数を増やした背景には `WMMA` 命令の対応が関係しているのかもしれない。  
また、*RDNA 3/GFX11* では最大 2種類の命令を同時に発行可能にすると思われる `VOPD (Dual issue wave32)` 命令にも対応する。  

*RDNA 3/GFX11* では部分的に構成が異なる SIMDユニットが 2種類存在することになるが、*Navi31/gfx1100* と *Navi32/gfx1101* のみに留まっている理由は今回触れられていない。触れられるとしたら正式発表時か HotChips での発表だろう。  
まず考えられる理由はダイサイズへの影響だが、追加のベクタレジスタの変更で興味深いのは実性能への影響と、そして *RDNA 3/GFX11* の次世代でも GPU によって分けるのか、それともすべてに追加のベクタレジスタを搭載するのか、という点ではないかと考えている。  

*RDNA 2/GFX10.3* では、Infinity Cache/ L3 Cache (MALL) がメモリチャネルあたり 8MiB か 4MiB、あるいは搭載しないという風に複数のキャッシュメモリ構成があったが、*RDNA 3/GFX11* ではさらに多様化することとなる。  

| VGPR per SIMD Unit | RDNA 1 | RDNA 2 | RDNA 3<br>(extra VGPR) |
| :--  | :--:   | :--:   | :--:   |
| VGPR File Size | 128KiB | 128KiB | 192KiB? |
| VGPR per Lane | 1024 | 1024 | 1536 |
| VGPR Alloc Granularity (Wave32) | 8 | 16 | 24 |

{{< ref >}}
 * [AMD RDNA™ - GPUOpen](https://gpuopen.com/rdna/)
 * [RDNA 3/GFX11 では行列積和演算命令、WMMA (Wave Matrix Multiply-accumulate) をサポート | Coelacanth's Dream](/posts/2022/06/29/gfx11-wmma-inst/)
 * [GFX11 でサポートされる VOPD (Dual issue wave32) 命令の範囲と特徴 | Coelacanth's Dream](/posts/2022/06/21/gfx11-vopd-instruction/)
 * [L0ベクタキャッシュと GL1データキャッシュが増やされる RDNA 3/GFX11 APU | Coelacanth's Dream](/posts/2022/09/02/gfx11-l0c-gl1c/)
 * [CDNA 2 アーキテクチャでは Unified Register Files を実装 | Coelacanth's Dream](/posts/2021/07/01/aldebaran-unified-vgpr/)
{{< /ref >}}
