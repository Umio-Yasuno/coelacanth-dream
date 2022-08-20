---
title: "Meteor Lake に向けた Intel Graphics Compiler へのさらなるパッチ"
date: 2022-08-19T04:51:03+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Meteor_Lake", "Xe-HPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

以前に Intel の Pete Chou 氏より、[Intel Graphics Compiler (IGC)](https://github.com/intel/intel-graphics-compiler) のサポートターゲット、vISA (virtual ISA) に *Meteor Lake* を追加するコミットが投稿された。  
そして今回、Intel の Mateusz Borzyszkowski 氏により、*Meteor Lake* のサポートに向けたさらなるパッチが IGC に投稿されている。  
{{< link >}} [Intel Graphics Compiler で Meteor Lake のサポートが進み始める ―― Xe-HPG、XMXユニットは非搭載か、再度 FP64 に対応 | Coelacanth's Dream](/posts/2022/07/06/igc-mtl/) {{< /link >}}

 * [Add MTL support · intel/intel-graphics-compiler@2013a0c](https://github.com/intel/intel-graphics-compiler/commit/2013a0c271b136be2629fe74cb2b4eefffad78c0)

## {{< xe >}}-HPG {#xe-hpg}
前回触れたように、*Meteor Lake GPU* の IGC におけるアーキテクチャ、プラットフォームは *{{< xe class="hpg" >}}* とされている。  
{{< link >}} [Intel Graphics Compiler で Meteor Lake のサポートが進み始める ―― Xe-HPG、XMXユニットは非搭載か、再度 FP64 に対応 | Coelacanth's Dream](/posts/2022/07/06/igc-mtl/) {{< /link >}}
しかし、*Alchemist/DG2* とまったく同じアーキテクチャ、構成という訳ではなく、*Meteor Lake GPU* では XMXユニットを用いて実行される `DPAS (Dot Product Accumulate Systolic)` 命令をサポートしないといった違いがある。  
*Meteor Lake GPU* では `FP64 (DP)` のサポートが復活することから、EU の構成もある程度 *Alchemist/DG2* とは異なると考えられる。  

 > 		     IGFX_XE_HP_SDV = 1250,
 > 		     IGFX_DG2 = 1270, // aka - ACM/Alchemist
 > 		     IGFX_PVC = 1271,
 > 		+    IGFX_METEORLAKE = 1272,
 >
 > {{< quote >}} [Add MTL support · intel/intel-graphics-compiler@2013a0c](https://github.com/intel/intel-graphics-compiler/commit/2013a0c271b136be2629fe74cb2b4eefffad78c0) {{< /quote >}}

*Meteor Lake GPU* の HWレイトレーシングのサポートについては、`IGC/AdaptorCommon/RayTracing/PrologueShaders.cpp` に `IGFX_METEORLAKE` が追加されていることからサポートしていると考えられる。  
それ以外の HWレイトレーシングのサポートを判定するコード部においても、変更がないことからも *Meteor Lake GPU* は *Alchemist/DG2* と *Ponte Vechhio* 同様にサポートしていると考えられる。  

 > 		bool supportRayTracing() const
 > 		{
 > 		    return isProductChildOf(IGFX_DG2);
 > 		}
 >
 > {{< quote >}} <https://github.com/intel/intel-graphics-compiler/blob/2013a0c271b136be2629fe74cb2b4eefffad78c0/IGC/Compiler/CISACodeGen/Platform.hpp#L742-L745> {{< /quote >}}

[^mtl-rt]: <https://github.com/intel/intel-graphics-compiler/blob/2013a0c271b136be2629fe74cb2b4eefffad78c0/IGC/AdaptorCommon/RayTracing/PrologueShaders.cpp>

## 倍精度と整数除算 {#intdiv}
IGC では整数の除算を *XE_HP_SDV* 世代からエミュレーションを用いるようにしており、`FP64 (DP)` をネイティブでサポートしている場合は `FP64` で、サポートしていない場合は `FP32 (SP)` でエミュレートする。  
*XE_HP_SDV* や *Ponte Vecchio* を除き、*Intel Gen11 アーキテクチャ* から `FP64 (DP)` のネイティブサポートは外されていたが、*Meteor Lake GPU* からは復活する。しかし、性能を理由に *Meteor Lake* でも整数除算は引き続き `FP32 (SP)` でエミュレートされる。  

*Meteor Lake GPU* の `FP64 (DP)` 演算は *Gen9 アーキテクチャ* と同じ Extended Math (EM) Pipe で実行する方式だが、世代を経て EU 構成は変更されており、*Gen11* までは *SIMD-4 (Int/FP) + SIMD-4 (FP/EM)* の構成だったが、*{{< xe >}}-LP* では *SIMD-8 (Int/FP) + SIMD-2 (EM)* の構成となっている。  
EM Pipe の比率が小さくなったため、`FP32 (SP)` 性能に対する `FP64 (DP)` 性能も低くなっていると考えられる。性能を理由として `FP32 (SP)` で整数除算をエミュレートするのは、そうした EU の構成変更が関わっているのだろう。  

 > 		// Has 64bit support but use 32bit for perf reasons
 > 		bool preferFP32Emu() const {
 > 		    return m_platformInfo.eProductFamily == IGFX_METEORLAKE;
 > 		}
 >
 > {{< quote >}} [Add MTL support · intel/intel-graphics-compiler@2013a0c](https://github.com/intel/intel-graphics-compiler/commit/2013a0c271b136be2629fe74cb2b4eefffad78c0) {{< /quote >}}
 >
 > 		    if (ctx.platform.preferFP32Emu() && IGC_IS_FLAG_DISABLED(Force32BitIntDivRemEmu)) {
 > 		        // Prefer using FP32 emulation even though DP support is available
 > 		        theEmuKind |= EmuKind::EMU_I32DIVREM_SP;
 > 		    }
 > 		    else if (ctx.platform.Enable32BitIntDivRemEmu())
 > 		    {
 > 		        if (!ctx.platform.hasNoFP64Inst())
 > 		        {
 > 		            // Use DP (and float) opeations to emulate int32 div/rem
 > 		            theEmuKind |= EmuKind::EMU_I32DIVREM;
 > 		        }
 > 		        else
 > 		        {
 > 		            // Use SP floating operations to emulate int32 div/rem
 > 		            theEmuKind |= EmuKind::EMU_I32DIVREM_SP;
 > 		        }
 > 		    }
 >
 > {{< quote >}} <https://github.com/intel/intel-graphics-compiler/blob/2013a0c271b136be2629fe74cb2b4eefffad78c0/IGC/Compiler/CISACodeGen/ShaderCodeGen.cpp> {{< /quote >}}

