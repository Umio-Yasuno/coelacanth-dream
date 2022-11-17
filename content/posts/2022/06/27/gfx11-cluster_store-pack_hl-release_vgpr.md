---
title: "GFX11 で有効化、サポートされる機能 ―― ReleaseVGPR, S_PACK_HL, Cluster stores"
date: 2022-06-27T12:13:54+09:00
draft: false
categories: [ "Hardware", "Software", "AMD", "GPU" ]
tags: [ "GFX11", "RDNA_3", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

LLVM において *GFX11* で有効化あるいはサポートされる機能のメモ。  

## Automatically Release VGPRs {#dealloc_vgpr}
AMDGPU からホスト CPU にメッセージ (割り込み) を送信する命令として `S_SENDMSG` があるが、*GFX11* ではメッセージの種類に新しく `MSG_DEALLOC_VGPRS` をサポートする。  
AMD の Jay Foad 氏により LLVM にパッチが投稿されている。  

 * [⚙ D128442 [AMDGPU] GFX11: automatically release VGPRs at the end of the shader](https://reviews.llvm.org/D128442)

`MSG_DEALLOC_VGPRS` はシェーダーの VGPR (ベクタレジスタ、Vector general-purpose register) の解放に使用できる。  
通常はシェーダーの最後に呼ばれる `S_ENDPGM` 命令で VGPR が解放されるが、`S_ENDPGM` の前に `MSG_DEALLOC_VGPRS` を送信することで、`S_ENDPGM` の段で VMEM (Vector Memory) への Store 処理の完了を待機する必要がある場合に、`MSG_DEALLOC_VGPRS` によって先に VGPR を解放できるため、システム全体のパフォーマンスが向上するとしている。  

## S_PACK_HL {#s_pack_hl}
*GFX11* では SALU (Scalar ALU) がサポートする命令に `S_PACK_HL` が追加される。  
`S_PACK` 系の命令では 2つのソースレジスタの上位 (H, High) 16-bit か下位 (Low, L) 16-bit をそれぞれ指定して、1つのレジスタに詰め込む (パック) する。  
*RDNA 2/GFX10.3* 世代では `S_PACK_{LL,LH,HH}` をサポートしており、*GFX11* 世代でサポートしていなかった `S_PACK_HL` が追加される形となる。  

 * [⚙ D128527 [AMDGPU] Use GFX11 S_PACK_HL instruction in more cases](https://reviews.llvm.org/D128527)

 > 		+  /// Return true if the target has the S_PACK_HL_B32_B16 instruction.
 > 		+  bool hasSPackHL() const { return GFX11Insts; }
 >
 > {{< quote >}} [[AMDGPU] gfx11 subtarget features & early tests · llvm/llvm-project@18ed279](https://github.com/llvm/llvm-project/commit/18ed279a3a4aa830830e5cfee3e3824f91c3ca6c) {{< /quote >}}

## Cluster stores {#cluster-store}
これまで AMDGPU 向けには Cluster loads のみが有効化されていたが、*GFX11* では Cluster stores も有効化される。  

 * [[AMDGPU] gfx11 subtarget features & early tests · llvm/llvm-project@18ed279](https://github.com/llvm/llvm-project/commit/18ed279a3a4aa830830e5cfee3e3824f91c3ca6c)
 * [⚙ D128517 [AMDGPU] Cluster stores as well as loads for GFX11](https://reviews.llvm.org/D128517)

 > 		+
 > 		+  // \returns true if it's beneficial on this subtarget for the scheduler to
 > 		+  // cluster stores as well as loads.
 > 		+  bool shouldClusterStores() const { return getGeneration() >= GFX11; }
 >
 > {{< quote >}} [[AMDGPU] gfx11 subtarget features & early tests · llvm/llvm-project@18ed279](https://github.com/llvm/llvm-project/commit/18ed279a3a4aa830830e5cfee3e3824f91c3ca6c) {{< /quote >}}

Cluster loads/stores は LLVM側のスケジューリングに関する機能であり、レジスタ使用量を増やすことで、キャッシュ効率の向上、Load/Store 命令後のスケジューリングにおける自由度の向上を目的とした機能と思われる。  

Cluster stores は一度 AMDGPU 向けに有効化されたが[^enable-store]、利点がほとんど無いとして、その後に無効化された経緯がある。[^disable-store]  
その時 AMD の Jay Foad 氏は、Cluster loads はキャッシングにおける利点があるが、Cluster stores はレジスタプレッシャーによりスケジューリングの自由度が下がる傾向にあると語っている。  

[^enable-store]: [⚙ D26414 AMDGPU: Enable store clustering](https://reviews.llvm.org/D26414)
[^disable-store]: [⚙ D85530 [AMDGPU] Don't cluster stores](https://reviews.llvm.org/D85530)

詳細はまだ不明だが、*GFX11* では Load/Store 周りに何かしらの改良が加えられたものと思われる。  
Cluster stores を無効化するパッチは 2020/08/07 に投稿されており、*GFX11* で再度有効化される背景には SW開発チームから HW開発チームへのフィードバックがあったのかもしれない。  

{{< ref >}}
 * [Developer Guides, Manuals & ISA Documents - AMD](https://developer.amd.com/resources/developer-guides-manuals/)
    * ["RDNA 2" Instruction Set Architecture: Reference Guide - RDNA2_Shader_ISA_November2020.pdf](https://developer.amd.com/wp-content/resources/RDNA2_Shader_ISA_November2020.pdf)
{{< /ref >}}
