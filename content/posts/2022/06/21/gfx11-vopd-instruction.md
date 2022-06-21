---
title: "GFX11 でサポートされる VOPD (Dual issue wave32) 命令の範囲と特徴"
date: 2022-06-21T05:57:44+09:00
draft: false
categories: [ "Hardware", "Software", "AMD", "GPU" ]
tags: [ "GFX11", "LLVM" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Joe Nash 氏より、*GFX11* に追加される `VOPD (Dual issue wave32)` 命令を LLVM MC (Machine Code) でサポートするパッチが投稿されている。  

 * [⚙ D128218 [AMDGPU] gfx11 VOPD instructions MC support](https://reviews.llvm.org/D128218)

以前のパッチで `HasVOPDInsts (VOPD dual issue wave32)` の機能フラグは出てきており、今回のパッチでさらなる詳細が明かされた。  
{{< link >}} [GFX11 のサポートに向けた LLVM へのさらなるパッチ ―― True16Bit, Dot8, Dual issue wave32 | Coelacanth's Dream](/posts/2022/05/10/llvm-gfx11-dual-issue/#vopd) {{< /link >}}


## VOPD (Dual issue wave32) {#vopd}

今回のパッチから `VOPD (Dual issue wave32)` 命令の機能は、対応する一部の命令から最大 2種類を同時に発行、実行することで性能と効率を向上させるものではないかと推測される。  
*GFX11* への対応が進むまで、*GFX11* の VGPR (Vector general-purpose registe) 数を 128、従来の半分に制限するパッチが別に投稿されているが、これも `VOPD` 命令への対応とそれによる設計変更に影響するものと思われる。  
*GFX11* ではオペランドのエンコードが変更され、従来は VGPR 128..255 が v128..v255 に対応していたのが、v0.hi..v127.hi に対応するようになる。  

 > 		 unsigned getAddressableNumVGPRs(const MCSubtargetInfo *STI) {
 > 		+  if (LimitTo128VGPRs.getNumOccurrences() ? LimitTo128VGPRs
 > 		+                                          : isGFX11Plus(*STI)) {
 > 		+    // GFX11 changes the encoding of 16-bit operands in VOP1/2/C instructions
 > 		+    // such that values 128..255 no longer mean v128..v255, they mean
 > 		+    // v0.hi..v127.hi instead. Until the compiler understands this, it is not
 > 		+    // safe to use v128..v255.
 > 		+    // TODO-GFX11: Remove this when full 16-bit codegen is implemented.
 > 		+    return 128;
 > 		+  }
 >
 > {{< quote >}} [[AMDGPU] Limit GFX11 to using 128 VGPRs · llvm/llvm-project@7050d5b](https://github.com/llvm/llvm-project/commit/7050d5b98c0952b24b61f88653de86443cbabd7c) {{< /quote >}}

なお、個人の推測であり、公式のドキュメント等からではなく、パッチ内容から読み取った結果であるため外れている可能性は当然ある。  

以下は `VOPD` 命令でサポートされる擬似的な命令のリストであり、実際の命令名は `V_DUAL_*` の形式となる。  
倍精度 (64-bit) 向け命令や半精度 (16-bit) 向けのパックド命令は含まれず、単精度 (32-bit) 向けの命令のみがサポートされている。  
FP64:FP32 の演算性能レートが 1:32 になったことと合わせ、*GFX11* では FP32 性能をより重視した設計になっているという印象を受ける。  
{{< link >}} [GFX11 では FP64 演算性能が FP32 の 1/32 に | Coelacanth's Dream](/posts/2022/06/18/gfx11-dpfp-rate/) {{< /link >}}

命令の種類としては、基本的な演算命令 `(ADD, SUB, MUL)`、FMA (`FMA{C,AK,MK}`) 命令、データの移動命令 (`MOV`)、比較命令 (`MAX, MIN`)、条件命令 (`CNDMASK`)、ビット演算命令 (`LSHLREV, AND`) となる。ビット演算命令に `OR, XOR` は含まれていない。  
また、ドット積命令に唯一 `DOT2C_F32_F16 (D.f32 = S0.f16[0] * S1.f16[0] + S0.f16[1] * S1.f16[1] + D.f32)` 命令がサポートされている。  

 > 		// V_DUAL_DOT2ACC_F32_BF16  is a legal instruction, but V_DOT2ACC_F32_BF16 is not.
 > 		// Since we generate the DUAL form by converting from the normal form we will
 > 		// never generate it.
 > 		defvar VOPDYPseudos = [
 > 		  "V_FMAC_F32_e32", "V_FMAAK_F32", "V_FMAMK_F32", "V_MUL_F32_e32",
 > 		  "V_ADD_F32_e32", "V_SUB_F32_e32", "V_SUBREV_F32_e32", "V_MUL_LEGACY_F32_e32",
 > 		  "V_MOV_B32_e32", "V_CNDMASK_B32_e32", "V_MAX_F32_e32", "V_MIN_F32_e32",
 > 		  "V_DOT2C_F32_F16_e32", "V_ADD_U32_e32", "V_LSHLREV_B32_e32", "V_AND_B32_e32"
 > 		];
 >
 > {{< quote >}} [⚙ D128218 [AMDGPU] gfx11 VOPD instructions MC support](https://reviews.llvm.org/D128218) {{< /quote >}}

今回のパッチで追加されたテストファイルの内容から、最大 2種類の `VOPD` 命令を組み合わせ、同時に発行、実行するものと推測した。  
`Dual issue wave32` とあるように、対応するのは Wave32 の場合のみとなる。  
*GFX11* における Wave64 の処理方法がどのようになっているかはまだ不明。*RDNA 1/GFX10.1, RDNA 2/GFX10.3* では Wave64 を、VGPR を共有する 2x Wave32 に分けて処理する方式を取っていた。  

 > 		# W32: v_dual_add_f32 v5, 0xaf123456, v2 :: v_dual_fmaak_f32 v6, v3, v1, 0xaf123456 ; encoding: [0xff,0x04,0x02,0xc9,0x03,0x03,0x06,0x05,0x56,0x34,0x12,0xaf]
 > 		0xff,0x04,0x02,0xc9,0x03,0x03,0x06,0x05,0x56,0x34,0x12,0xaf
 > 		
 > 		# W32: v_dual_cndmask_b32 v20, v21, v22 :: v_dual_mov_b32 v41, v42 ; encoding: [0x15,0x2d,0x50,0xca,0x2a,0x01,0x28,0x14]
 > 		0x15,0x2d,0x50,0xca,0x2a,0x01,0x28,0x14
 > 		
 > 		# W32: v_dual_fmaak_f32 v122, s74, v161, 0x402f6c8b :: v_dual_and_b32 v247, v160, v98 ; encoding: [0x4a,0x42,0x65,0xc8,0xa0,0xc5,0xf6,0x7a,0x8b,0x6c,0x2f,0x40]
 > 		0x4a,0x42,0x65,0xc8,0xa0,0xc5,0xf6,0x7a,0x8b,0x6c,0x2f,0x40
 >
 > {{< quote >}} [⚙ D128218 [AMDGPU] gfx11 VOPD instructions MC support](https://reviews.llvm.org/D128218) {{< /quote >}}

`VOPD` 命令のメリットとしては、単純にすれば *RDNA 1/GFX10.1, RDNA 2/GFX10.3* では 2x Wave32 に処理に 2サイクル掛かっていたが、対応命令であれば 1サイクルで済み、性能と効率が向上することが考えられる。  
デメリットとしては、限られた範囲の命令とはいえ、組み合わせた命令を生成しなければならないため、コンパイラへの負担が大きくなることが予想される。  
最大 2種類であり、同じ命令の組み合わせに対応しているため、これも単純に考えれば、同じ命令を実行する Wave が続く場合における性能向上が期待できるが、依存関係の判定が複雑になるように思われる。  
また、`VOPD` 命令においても VGPR を共有しているように見えるため、*RDNA 1/GFX10.1, RDNA 2/GFX10.3* では 2x Wave32 に分けていた Wave64 が、*GFX11* では 1サイクルで処理できるようになっている可能性もある。  
その場合、`VOPD` 命令の最大の特徴は異なる命令を組み合わせられる点にあると考えられる。  

 > 		v_dual_mul_f32 v0, v0, v2 :: v_dual_mul_f32 v1, v1, v3
 > 		// GFX11: encoding: [0x00,0x05,0xc6,0xc8,0x01,0x07,0x00,0x00]
 > 		// W64-ERR: :[[@LINE-2]]:{{[0-9]+}}: error
 >
 > {{< quote >}} [⚙ D128218 [AMDGPU] gfx11 VOPD instructions MC support](https://reviews.llvm.org/D128218) {{< /quote >}}

AMDGPU 向けのコンパイラバックエンドには LLVM の他に、*RADV (Vulkan)* ドライバーに採用されている [ACO](/tags/aco) がある。  
それらコンパイラバックエンドにおいて、どのように `VOPD` 命令への対応、最適化が行われるかは興味深いポイントではないかと思う。  

