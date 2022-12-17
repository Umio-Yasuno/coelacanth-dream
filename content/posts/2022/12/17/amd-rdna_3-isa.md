---
title: "AMD、RDNA 3 Instruction Set Architecture: Reference Guide を公開"
date: 2022-12-17T16:33:02+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD は 2022-12-16 付で 「"RDNA3" Instruction Set Architecture: Reference Guide」 (以下、ドキュメント) を公開した。  
その内容から自分が気になったものをピックアップしていく。  

 * [Developer Guides, Manuals & ISA Documents - AMD](https://developer.amd.com/resources/developer-guides-manuals/)
    * ["RDNA3" Instruction Set Architecture: Reference Guide - RDNA3_Shader_ISA_December2022.pdf](https://developer.amd.com/wp-content/resources/RDNA3_Shader_ISA_December2022.pdf)

## GDS {#gds}
すべての ShaderEngine、CU からアクセス可能なスクラッチメモリ、GDS (Global Data Share) のサイズが変更され、*RDNA 2 アーキテクチャ* では 64KiB だったのが、*RDNA 3 アーキテクチャ* では 4KiB に減らされている。[^gds]  
同様の変更は *CDNA 1/2 アーキテクチャ* でも見られ、*CDNA 1/MI100/Arcturus* では 4KiB、*CDNA 2/MI200/Aldebaran* では GDS 自体が削除されている。[^gds-cdna]  
1つの GPU に搭載される ShaderEngine、CU 数が増えるにつれ、Atomic 操作周りの実装コストが高くなっているのだろうか。  
*RDNA 3 アーキテクチャ* では最大 ShaderEngine 数が 6基と、*RDNA 2 アーキテクチャ* の 4基から増えている。  

[^gds]: RDNA3_Shader_ISA_December2022.pdf: Page 7
[^gds-cdna]: [AMD Instinct MI200/CDNA2 Instruction Set Architecture が公開される | Coelacanth's Dream](/posts/2021/11/20/cdna_2-isa/)

## Wave64 {#wave64}
AMD は *RDNA 3 アーキテクチャ* における Wave (スレッドの塊、SIMD ユニットにおいて同時に実行される単位) の発行について、Wave64 を 1クロックで発行すると説明していたが[^wave64-1clk]、
ドキュメントでは通常 2回に分けて命令が発行される、あるいは 2x Wave32 に分けて発行される *場合* があるとしている。[^wave64-issue]  
*RDNA 2 アーキテクチャ* では 2x Wave32 に分けて発行されるとはっきり書いていた文章部に、*may* が追加されて曖昧になってもいる。  
ドキュメント公開前の説明では、`FMA` 命令を例に用いていたことから、Dual Issue Wave32 に対応する一部の命令と同様の場合に限って、Wave64 を 1クロックで発行可能という可能性も考えられる。  

[^wave64-1clk]: [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (5/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/005_o.jpg.html)
[^wave64-issue]: RDNA3_Shader_ISA_December2022.pdf: Page 9, Page 61

*RDNA 1/2 アーキテクチャ* では Wave64 を low と high に分けて実行し、またいくつかの Wave64 をまとめて low 群と high 群に分け、先に low 群を実行することでメモリ効率を向上させる Subvector Execution に対応していた。(`s_subvector_loop_begin, s_subvector_loop_end`)  
それが *RDNA 3 アーキテクチャ* ではドキュメントから Subvector Execution の記述が無くなっている。  

## Dual issue VALU/Wave32, VOPD {#vopd}
Dual issue VALU/Wave32, `VOPD` 命令は対応する命令を最大 2種類同時に発行することができるが、ドキュメントによれば選択可能な命令は X側と Y側で異なり、`V_DUAL_ADD_NC_U32, V_DUAL_LSHLREV_B32, V_DUAL_AND_B32` 命令は Y側でのみ選択できる。  
また `VOPD` 命令として `V_DUAL_DOT2ACC_F32_BF16` が実装されているが、通常の命令として `V_DOT2ACC_F32_BF16` はサポートされていないため、
元になる命令から `VOPD` 命令を生成する方針を取る LLVM では `V_DUAL_DOT2ACC_F32_BF16` には対応しない。[^llvm-vopd]  

[^dual-issue]: RDNA3_Shader_ISA_December2022.pdf: Page 68
[^llvm-vopd]: [⚙ D128218 [AMDGPU] gfx11 VOPD instructions MC support](https://reviews.llvm.org/D128218)

## WMMA {#wmma}
事前の発表と同様に `WMMA (Wave Matrix Multiply Accumulate)` 命令は内部的には複数のドット積命令を発行するとドキュメントで説明されている。[^wmma]  
`WMMA` 命令の性能を出す上で、ソースオペランドに指定された行列 A, B のデータにおいて lane 0-15 が lane 16-31 に複製されるように配置する必要があるとしている。  

[^wmma]: RDNA3_Shader_ISA_December2022.pdf: Page 73

## S_DELAY_ALU {#s_delay_alu}
オプションの命令として、依存関係を持つ命令を同時に発行することを避けることを目的として、ALU への命令発行を遅らせる `S_DELAY_ALU` 命令が *RDNA 3 アーキテクチャ* では追加された。[^s_delay_alu]  
Dual issue VALU/Wave32 と併せて、*RDNA 3 アーキテクチャ* ではコンパイラ側での最適化余地が増えたが、同時に負担も大きくなっているように思う。  

[^s_delay_alu]: RDNA3_Shader_ISA_December2022.pdf: Page 44

## SLC, GLC, DLC {#cache}
*RDNA 1/2/3 アーキテクチャ* ではスカラ、ベクタ共にメモリ命令 (load, store, atomic) ではキャッシュポリシー指定用のフラグビット `SLC, GLC, DLC` が用意されているが、*RDNA 3* ではそれぞれの意味が変更され、MALL (Memory-Attached Last-Level cache, Infinity Cache) のキャッシュコントロールが可能になった。[^cache]  
*RDNA 2 アーキテクチャ* では、`GLC (Globally Coherent)` は CU 内の L0キャッシュ、`DLC (Device Level Coherent)` と `SLC (System Level Coherent)` は L2キャッシュのコントロールビットとなっていたa。`DLC` と `SLC` のパターンでキャッシュポリシーは変わる。  
*RDNA 3 アーキテクチャ* では `GLC` は GL1 (first-level cache)、`SLC` は GL2 (L2キャッシュ)、`DLC` は MALL 用と再定義された。  
また、SRD (Shader Resouce Descriptor) 部に `LLC NoAlloc` 2-bits が用意されており、その部分の値でも MALL に対するキャッシュポリシーは変わる。  

[^cache]: RDNA3_Shader_ISA_December2022.pdf: Page 35
