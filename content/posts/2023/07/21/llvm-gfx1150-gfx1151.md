---
title: "LLVM に新たな RDNA 3 APU の GPUID、gfx1150 と gfx1151 が追加される"
date: 2023-07-21T04:34:27+09:00
draft: false 
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "GFX11", "RDNA_3", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Jay Foad 氏により、LLVM に AMD の新たな RDNA 3 APU となる GPUID、*gfx1150/gfx1151* を追加するパッチが公開された。  
 
 * <https://reviews.llvm.org/rG92542f2a400024e8a878242afe8231e17df345e5>
 * [[AMDGPU] Add targets gfx1150 and gfx1151 · llvm/llvm-project@92542f2](https://github.com/llvm/llvm-project/commit/92542f2a400024e8a878242afe8231e17df345e5)

AMD GPU における GPUID のフォーマット (`{major}.{minor}.{patch}`) において、*gfx1150/gfx1151* は従来の *RDNA 3/GFX11* APU/dGPU からマイナーバージョンが上がった形となる。  

## gfx1150/gfx1151

*gfx1150/gfx1151* は `llvm/docs/AMDGPUUsage.rst` に追加された記述から APU とされている。  

 >         +     ``gfx1150``                 ``amdgcn``   APU   - cumode          - Architected                   *TBA*
 >         +                                                    - wavefrontsize64   flat
 >         +                                                                        scratch                       .. TODO::
 >         +                                                                      - Packed
 >         +                                                                        work-item                       Add product
 >         +                                                                        IDs                             names.
 >         +
 >         +     ``gfx1151``                 ``amdgcn``   APU   - cumode          - Architected                   *TBA*
 >         +                                                    - wavefrontsize64   flat
 >         +                                                                        scratch                       .. TODO::
 >         +                                                                      - Packed
 >         +                                                                        work-item                       Add product
 >         +                                                                        IDs                             names.
 >         +
 >
 > {{< quote >}} <https://reviews.llvm.org/rG92542f2a400024e8a878242afe8231e17df345e5> {{< /quote >}}

現状、*gfx1150/gfx1151* に従来の *RDNA 3/GFX11* から追加された新命令や新機能は無いことになっている。言い換えれば、*Phoenix APU (gfx1103)* と同様に dGPU と同じ命令をサポートする。  
ただ、*RDNA 3/GFX11* に共通して存在していた `FeatureVALUTransUseHazard` の問題が *gfx1150/gfx1151* では修正されている。  
また、*gfx1151* では `FeatureGFX11FullVGPRs` が有効になっている。`FeatureGFX11FullVGPRs` は SIMDユニットが追加のベクタレジスタを持つことを示す機能フラグであり、通常のベクタレジスタサイズは SIMDユニットあたり 128KiB だが、`FeatureGFX11FullVGPRs` が有効な場合は 192KiB となる。  
`FeatureGFX11FullVGPRs` は *RDNA 3/GFX11* では一部の GPUID でのみ有効であり、これまでは *Navi31/gfx1100* と *Navi32/gfx1101* だけだった。  
このことから *gfx1151* は GPU 性能を重視した APU となるのではないかという推測が立てられる。  

 >         def FeatureVALUTransUseHazard : SubtargetFeature<"valu-trans-use-hazard",
 >           "HasVALUTransUseHazard",
 >           "true",
 >           "Hazard when TRANS instructions are closely followed by a use of the result"
 >         >;
 >
 > {{< quote >}} [llvm-project/llvm/lib/Target/AMDGPU/AMDGPU.td at f12a5561b2cbfae384c9a31293938ee2acea79fd · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/f12a5561b2cbfae384c9a31293938ee2acea79fd/llvm/lib/Target/AMDGPU/AMDGPU.td#L764-L768) {{< /quote >}}
 >
 >         @@ -1309,26 +1309,37 @@ def FeatureISAVersion11_Common : FeatureSet<
 >             FeatureImageInsts,
 >             FeaturePackedTID,
 >             FeatureVcmpxPermlaneHazard,
 >         -   FeatureVALUTransUseHazard,
 >             FeatureMADIntraFwdBug]>;
 >          
 >         -def FeatureISAVersion11_0_0 : FeatureSet<
 >         +def FeatureISAVersion11_0_Common : FeatureSet<
 >            !listconcat(FeatureISAVersion11_Common.Features,
 >         +    [FeatureVALUTransUseHazard])>;
 >         +
 >         +def FeatureISAVersion11_0_0 : FeatureSet<
 >         +  !listconcat(FeatureISAVersion11_0_Common.Features,
 >              [FeatureGFX11FullVGPRs,
 >               FeatureUserSGPRInit16Bug])>;
 >          
 >          def FeatureISAVersion11_0_1 : FeatureSet<
 >         -  !listconcat(FeatureISAVersion11_Common.Features,
 >         +  !listconcat(FeatureISAVersion11_0_Common.Features,
 >              [FeatureGFX11FullVGPRs])>;
 >          
 >          def FeatureISAVersion11_0_2 : FeatureSet<
 >         -  !listconcat(FeatureISAVersion11_Common.Features,
 >         +  !listconcat(FeatureISAVersion11_0_Common.Features,
 >              [FeatureUserSGPRInit16Bug])>;
 >          
 >          def FeatureISAVersion11_0_3 : FeatureSet<
 >         +  !listconcat(FeatureISAVersion11_0_Common.Features,
 >         +    [])>;
 >         +
 >         +def FeatureISAVersion11_5_0 : FeatureSet<
 >            !listconcat(FeatureISAVersion11_Common.Features,
 >              [])>;
 >          
 >         +def FeatureISAVersion11_5_1 : FeatureSet<
 >         +  !listconcat(FeatureISAVersion11_Common.Features,
 >         +    [FeatureGFX11FullVGPRs])>;
 >         +
 >
 > {{< quote >}} <https://reviews.llvm.org/rG92542f2a400024e8a878242afe8231e17df345e5> {{< /quote >}}

{{< ref >}}
 * [追加のベクタレジスタを持つ Navi31/GFX1100 と Navi32/GFX1101 | Coelacanth's Dream](/posts/2022/09/23/some-gfx11-extra-vgpr/)
{{< /ref >}}
