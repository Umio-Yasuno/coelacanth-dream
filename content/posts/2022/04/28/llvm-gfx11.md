---
title: "LLVM で GFX11 のサポートが進み始める"
date: 2022-04-28T08:41:07+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "LLVM", "GFX11" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Joe Nash 氏より、LLVM に GFX11 (`gfx11.x.x`) のサポートを追加するパッチが投稿され、既に一部がメインラインに取り込まれている。  
TargetID/GPU ID は `{major}.{minor}.{stepping}` のフォーマットで表現され、*RDNA 1* から *RDNA 2* では minor version が更新されたが、今回で major version が更新された。  

 * [[AMDGPU][clang] Definition of gfx11 subtarget · llvm/llvm-project@8bdfc73](https://github.com/llvm/llvm-project/commit/8bdfc73f633dca9859123b8596bcb521700c6a7f)
 * [[AMDGPU] Add gfx11 subtarget ELF definition · llvm/llvm-project@813e521](https://github.com/llvm/llvm-project/commit/813e521e55b11165138b071f446eda94b14570dc)

## gfx1100/gfx1101/gfx1102/gfx1103 {#gfx11}

追加された TargetID/GPU ID は *gfx1100/gfx1101/gfx1102/gfx1103* 。  
ドキュメントに追加された内容では、*gfx1100/gfx1101/gfx1102* は dGPU、*gfx1103* は APU としている。  

 > 		+     **GCN GFX11** [AMD-GCN-GFX11]_
 > 		+     -----------------------------------------------------------------------------------------------------------------------
 > 		+     ``gfx1100``                 ``amdgcn``   dGPU  - cumode          - Architected   - *pal-amdpal*  *TBA*
 > 		+                                                    - wavefrontsize64   flat
 > 		+                                                                        scratch                       .. TODO::
 > 		+                                                                      - Packed
 > 		+                                                                        work-item                       Add product
 > 		+                                                                        IDs                             names.
 > 		+
 > 		+     ``gfx1101``                 ``amdgcn``   dGPU  - cumode          - Architected                   *TBA*
 > 		+                                                    - wavefrontsize64   flat
 > 		+                                                                        scratch                       .. TODO::
 > 		+                                                                      - Packed
 > 		+                                                                        work-item                       Add product
 > 		+                                                                        IDs                             names.
 > 		+
 > 		+     ``gfx1102``                 ``amdgcn``   dGPU  - cumode          - Architected                   *TBA*
 > 		+                                                    - wavefrontsize64   flat
 > 		+                                                                        scratch                       .. TODO::
 > 		+                                                                      - Packed
 > 		+                                                                        work-item                       Add product
 > 		+                                                                        IDs                             names.
 > 		+
 > 		+     ``gfx1103``                 ``amdgcn``   APU   - cumode          - Architected                   *TBA*
 > 		+                                                    - wavefrontsize64   flat
 > 		+                                                                        scratch                       .. TODO::
 > 		+                                                                      - Packed
 > 		+                                                                        work-item                       Add product
 > 		+                                                                        IDs                             names.
 > 		+
 >
 > {{< quote >}} [[AMDGPU] Add gfx11 subtarget ELF definition · llvm/llvm-project@813e521](https://github.com/llvm/llvm-project/commit/813e521e55b11165138b071f446eda94b14570dc) {{< /quote >}}

## 対応命令/機能 {#isa}

部分的ではあるが、*GFX11* での ISA、対応命令範囲にも触れられている。  
*RDNA 2/GFX10.3 (gfx103x)* と *GFX11 (gfx110x)* と比較したものが以下。  
`dot2-insts (v_dot2_i32_i16, v_dot2_u32_u16)` のフラグが消され、`dot8-insts` が新たに追加されているが、`dot8-insts` の内容はパッチには含まれていない。今後公開されるパッチで明かされるものと思われる。  

今回のパッチの範囲で言えば、*GFX11* には *CDNA 系アーキテクチャ* でサポートしている MFMA (Matrix FMA) 系命令はサポートせず、  
*CDNA 系* では最新の *GFX940* でもサポートしていないドット積命令が追加される形となる。  

 > 		    case GK_GFX1036:			      |	    case GK_GFX1103:
 > 		    case GK_GFX1035:			      |	    case GK_GFX1102:
 > 		    case GK_GFX1034:			      |	    case GK_GFX1101:
 > 		    case GK_GFX1033:			      |	    case GK_GFX1100:
 > 		    case GK_GFX1032:			      <
 > 		    case GK_GFX1031:			      <
 > 		    case GK_GFX1030:			      <
 > 		      Features["ci-insts"] = true;		      Features["ci-insts"] = true;
 > 		      Features["dot1-insts"] = true;		      Features["dot1-insts"] = true;
 > 		      Features["dot2-insts"] = true;	      <
 > 		      Features["dot5-insts"] = true;		      Features["dot5-insts"] = true;
 > 		      Features["dot6-insts"] = true;		      Features["dot6-insts"] = true;
 > 		      Features["dot7-insts"] = true;		      Features["dot7-insts"] = true;
 > 							      >	      Features["dot8-insts"] = true;
 > 		      Features["dl-insts"] = true;		      Features["dl-insts"] = true;
 > 		      Features["flat-address-space"] = true;	      Features["flat-address-space"] = true;
 > 		      Features["16-bit-insts"] = true;		      Features["16-bit-insts"] = true;
 > 		      Features["dpp"] = true;			      Features["dpp"] = true;
 > 		      Features["gfx8-insts"] = true;		      Features["gfx8-insts"] = true;
 > 		      Features["gfx9-insts"] = true;		      Features["gfx9-insts"] = true;
 > 		      Features["gfx10-insts"] = true;		      Features["gfx10-insts"] = true;
 > 		      Features["gfx10-3-insts"] = true;		      Features["gfx10-3-insts"] = true;
 > 		      Features["s-memrealtime"] = true;	      |	      Features["gfx11-insts"] = true;
 > 		      Features["s-memtime-inst"] = true;      <
 > 		      break;					      break;
 >
 > {{< quote >}} [[AMDGPU][clang] Definition of gfx11 subtarget · llvm/llvm-project@8bdfc73](https://github.com/llvm/llvm-project/commit/8bdfc73f633dca9859123b8596bcb521700c6a7f) {{< /quote >}}

