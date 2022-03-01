---
title: "新たな CDNA系の GPUID 「gfx940」"
date: 2022-03-01T17:15:05+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "gfx940", "CDNA_2", "CDNA", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMDGPU におけるコンパイラバックエンドとしても採用されている LLVM に、新たな Target/GPU/GFX ID、*gfx940* を追加するパッチが投稿されている。  

* [⚙ D120688 [AMDGPU] Add gfx940 target](https://reviews.llvm.org/D120688)

ISA として *gfx90a* は *gfx90a/CDNA 2/Aldebaran* とほぼ一致し、その特徴である 行列演算命令 `MFMA (Matrix-Fused-Multiply-Add)` やフルレートFP64演算 (`FullRate64Ops`)、パックドFP32演算 (`FeaturePackedFP32Ops`) をサポートしている。  
そのことから、*gfx940* は CDNA系列の GPU を示す Target/GPU/GFX ID と考えられる。  
CPU+GPU のメモリ空間統合に関係する `XNACK` 機能も同様にサポートされている。  

今回 *gfx940* のサポート追加にあたって新しい命令、機能は現時点で追加されておらず、またコードコメントでも *gfx940* は *gfx90a* の派生 (derivation) と語られている。  
GPU としては、コア部やメモリの規模、I/O部の改良がメインになるのではないかと思われる。  

 > 		  // GFX940 is a derivation to GFX90A. hasGFX940Insts() being true implies that
 > 		  // hasGFX90AInsts is also true.
 > 		  bool hasGFX940Insts() const { return GFX940Insts; }
 >
 > {{< quote >}} [⚙ D120688 [AMDGPU] Add gfx940 target](https://reviews.llvm.org/D120688) {{< /quote >}}

*gfx90a* と *gfx940* がサポートする機能の差分が以下。  
主要な命令、機能としては `FeatureMadMacF32Insts` が削除され、`FeatureArchitectedFlatScratch` が追加されている。  
`FeatureMadMacF32Insts` は `V_MAD_F32/V_MAC_F32/V_MADAK_F32/V_MADMK_F32` 命令をサポートすることを示す機能フラグとなる。[^mad-mac-flag] それら命令は *RDNA 2/GFX10.3* 世代ではサポートされておらず、より丸め誤差、ULP (Unit of Last Place) が小さい `V_FMA_F32` 命令に置き換えられた。  
{{< link >}} [RadeonSIドライバーに FMA32命令を強制するオプションが追加 | Coelacanth's Dream](/posts/2021/12/18/radeonsi-force-fma32/) {{< /link >}}
*gfx90a* ではまだサポートが残されていたが、`FeatureMadMacF32Insts` が削除されたことから *gfx940* では *RDNA 2/GFX10.3* に揃えたと見られる。  

[^mad-mac-flag]: [llvm-project/AMDGPU.td at a5d4f82b7392630e657fc8f4a46b62bfc25c7962 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/a5d4f82b7392630e657fc8f4a46b62bfc25c7962/llvm/lib/Target/AMDGPU/AMDGPU.td#L607-L611)

 > 		def FeatureISAVersion9_0_A : FeatureSe |  def FeatureISAVersion9_4_0 : FeatureSe
 > 		  [FeatureGFX9,                             [FeatureGFX9,
 > 		   FeatureGFX90AInsts,                       FeatureGFX90AInsts,
 > 		                                       >     FeatureGFX940Insts,
 > 		   FeatureFmaMixInsts,                       FeatureFmaMixInsts,
 > 		   FeatureLDSBankCount32,                    FeatureLDSBankCount32,
 > 		   FeatureDLInsts,                           FeatureDLInsts,
 > 		   FeatureDot1Insts,                         FeatureDot1Insts,
 > 		   FeatureDot2Insts,                         FeatureDot2Insts,
 > 		   FeatureDot3Insts,                         FeatureDot3Insts,
 > 		   FeatureDot4Insts,                         FeatureDot4Insts,
 > 		   FeatureDot5Insts,                         FeatureDot5Insts,
 > 		   FeatureDot6Insts,                         FeatureDot6Insts,
 > 		   FeatureDot7Insts,                         FeatureDot7Insts,
 > 		   Feature64BitDPP,                          Feature64BitDPP,
 > 		   FeaturePackedFP32Ops,                     FeaturePackedFP32Ops,
 > 		   FeatureMAIInsts,                          FeatureMAIInsts,
 > 		   FeaturePkFmacF16Inst,                     FeaturePkFmacF16Inst,
 > 		   FeatureAtomicFaddInsts,                   FeatureAtomicFaddInsts,
 > 		   FeatureMadMacF32Insts,              <
 > 		   FeatureSupportsSRAMECC,                   FeatureSupportsSRAMECC,
 > 		   FeaturePackedTID,                         FeaturePackedTID,
 > 		                                       >     FeatureArchitectedFlatScratch,
 > 		   FullRate64Ops]>;                          FullRate64Ops]>;

ほとんど余談だが、これまで *GFX9/Vega* 世代では Target/GPU/GFX ID (`{Major}.{Minor}.{Stepping}`) を、Stepping ver を刻む形で更新し割り当ててきたが、今回で初めて Minor ver が更新された。  
*RDNA/GFX10* 世代では Minor ver に、*RDNA 1* では 1 (GFX10.1.x)、*RDNA 2* では 3 (GFX10.3.x) を割り当てる形で活用しているため、そうした面も *RDNA/GFX10* 世代に合わせたのだろうか。  

{{< ref >}}
* ["AMD Instinct MI200" Instruction Set Architecture: Reference Guide - CDNA2_Shader_ISA_4February2022.pdf](https://developer.amd.com/wp-content/resources/CDNA2_Shader_ISA_4February2022.pdf)
* ["RDNA 2" Instruction Set Architecture: Reference Guide - RDNA2_Shader_ISA_November2020.pdf](https://developer.amd.com/wp-content/resources/RDNA2_Shader_ISA_November2020.pdf)
{{< /ref >}}
