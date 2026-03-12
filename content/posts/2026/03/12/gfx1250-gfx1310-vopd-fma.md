---
title: "RDNA 5 ではより多くのケースで FP32 性能が倍になる可能性"
date: 2026-03-12T12:05:03+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX13", "RDNA_5", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

LLVM で現在対応が進められている *AMD RDNA 5 アーキテクチャ (gfx1310)* だが、最近になり Dual Issue VALU/Wave32 に関するプルリクエストが公開された。  

 * [[AMDGPU] Add VOPD to gfx13 (#182815) · llvm/llvm-project@1911488](https://github.com/llvm/llvm-project/commit/1911488ca3b27e006588f5e2a02e63dc85ba3f3f)

Dual Issue VALU は *RDNA 3 アーキテクチャ (GFX11)* から実装された機能であり、`VOPD` 系命令 (`V_DUAL_*`) を最大 2種類を組み合わせて、X側と Y側に同時に発行することができる。  
ただし、対応する命令は限られており、Y側だけでしか選択できない命令もある。また、Wave32 モードでしか対応しない、レジスタバンクの競合を避けるために X側と Y側で別のレジスタバンクを使用する必要がある等、いくつかの条件がある。  

*RDNA 5 (gfx1310)* では `VOPD` に `V_{MAX,MIN}_I32_e32, V_SUB_U32_e32, V_LSHRREV_B32_e32, V_ASHRREV_I32_e32` といった、整数を入力に取る最大値、最小値を取り出す命令、減算命令、ビットシフト命令が追加された。  
一方でビット論理積命令の `V_AND_B32` は削除されている。  
ここと以降で参照している LLVM のコード中に書いている命令は名前から `DUAL` を省略し、元となった命令名で表現した疑似命令となっている。  

そして、*RDNA 5 (gfx1310)* では Dual Issue 系命令として、新たな命令エンコーディングを採用した `VOPD3` が追加された。  
`VOPD3` には、`V_{FMAAK,FMAMK}_F32` 命令を除いた `VOPD` の命令が含まれており、そして `V_FMA_F32_e64` 命令が追加されている。  

 >         +defvar VOPD3XPseudosExtraGFX13 = ["V_ADD_U32_e32", "V_LSHLREV_B32_e32", "V_FMA_F32_e64", "V_SUB_U32_e32",
 >         +                                  "V_LSHRREV_B32_e32", "V_ASHRREV_I32_e32"];
 >         +defvar VOPD3XPseudosExtraGFX1250 = ["V_FMA_F64_e64", "V_ADD_F64_pseudo_e32",
 >         +                                    "V_MUL_F64_pseudo_e32", "V_MAX_NUM_F64_e32", "V_MIN_NUM_F64_e32"];
 >         +defvar VOPD3XPseudosGFX13 = !listconcat(
 >         +                              !filter(x, VOPDXPseudosGFX12, !and(!eq(!find(x, "FMAAK"), -1),
 >         +                                                                 !eq(!find(x, "FMAMK"), -1))),
 >         +                              VOPD3XPseudosExtraGFX13);
 >         +defvar VOPD3XPseudosGFX1250 = !listconcat(VOPD3XPseudosGFX13, VOPD3XPseudosExtraGFX1250);
 >
 > {{< quote >}} [[AMDGPU] Add VOPD to gfx13 (#182815) · llvm/llvm-project@1911488](https://github.com/llvm/llvm-project/commit/1911488ca3b27e006588f5e2a02e63dc85ba3f3f) {{< /quote >}}

`V_{FMAAK,FMAMK}_F32` 命令と `V_FMA_F32` 命令の違いは入力にリテラル定数を用いるかどうかであり、`V_FMAAK_F32` 命令は加算部分の引数に、`V_FMAMK_F32` 命令は乗算部分の片方の引数にリテラル定数を用いる。  
`V_FMA_F32` 命令は命令エンコーディングの違いから、3つの入力にそれぞれ別のレジスタを指定可能となっている。  
FMA 系命令には他に `V_FMAC_F32` 命令が存在するが、こちらは累算向けで、加算部分に指定したレジスタに演算結果を出力する命令となる。  

従来の Dual Issue、`VOPD` では `V_{FMAC,FMAAK,FMAMK}_F32` 命令をサポートし、 `V_FMA_F32` 命令はサポートしていなかった。  
そのため、[nihui/vkpeak](https://github.com/nihui/vkpeak) のような GPU ベンチマークツールでは Dual Issue が機能せず、公称ピーク FP32 性能の半分しか性能が出ないケースがあった。  

 * [Radeon RX 9060 XT 16GB を購入したので簡易レビュー | Coelacanth's Dream](/posts/2025/06/22/rx-9060-xt-review/#%e6%80%a7%e8%83%bd)

それが *RDNA 5 (gfx1310)* では X側 Y側の両方に `V_FMA_F32 (V_DUAL_FMA_F32)` 命令を発行可能となったことで改善される可能性がある。  

 >         defvar VOPD3XPseudosExtraGFX13 = ["V_ADD_U32_e32", "V_LSHLREV_B32_e32", "V_FMA_F32_e64", "V_SUB_U32_e32",
 >                                           "V_LSHRREV_B32_e32", "V_ASHRREV_I32_e32"];
 >         defvar VOPD3XPseudosExtraGFX1250 = ["V_FMA_F64_e64", "V_ADD_F64_pseudo_e32",
 >                                             "V_MUL_F64_pseudo_e32", "V_MAX_NUM_F64_e32", "V_MIN_NUM_F64_e32"];
 >         defvar VOPD3XPseudosGFX13 = !listconcat(
 >                                       !filter(x, VOPDXPseudosGFX12, !and(!eq(!find(x, "FMAAK"), -1),
 >                                                                          !eq(!find(x, "FMAMK"), -1))),
 >                                       VOPD3XPseudosExtraGFX13);
 >         defvar VOPD3XPseudosGFX1250 = !listconcat(VOPD3XPseudosGFX13, VOPD3XPseudosExtraGFX1250);
 >         defvar VOPD3YPseudosExtra = ["V_BITOP3_B32_e64", "V_FMA_F32_e64"];
 >         defvar VOPD3YPseudosGFX1250 = !listconcat(
 >                                         !filter(x, VOPDYPseudosGFX1250, !and(!eq(!find(x, "FMAAK"), -1),
 >                                                                              !eq(!find(x, "FMAMK"), -1))),
 >                                         VOPD3YPseudosExtra);
 >         
 >         def GFX1250GenD3 : GFXGenD<GFX1250Gen, VOPD3XPseudosGFX1250, VOPD3YPseudosGFX1250>;
 >         
 >         def GFX13GenD3 : GFXGenD<GFX13Gen, VOPD3XPseudosGFX13, VOPD3YPseudosGFX1250>;
 >         
 > {{< quote >}} <https://github.com/llvm/llvm-project/blob/bc211002c2e84f20fb3a028aadcca9606d8045f5/llvm/lib/Target/AMDGPU/VOPDInstructions.td#L249-L266> {{< /quote >}}

また、*RDNA 5 (gfx1310)* では Dual Issue における制限が一部緩和され、X側と Y側で同じ入力レジスタを指定できるようになっており、これも Dual Issue が使えるケースを増やすのに役立つだろう。  
