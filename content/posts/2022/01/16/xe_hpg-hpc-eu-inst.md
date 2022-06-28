---
title: "DG2-G12, DG2 L3 banks, SIMD width"
date: 2022-01-16T18:58:13+09:00
draft: false
tags: [ "Xe-HPG", "Xe-HPC", "DG2", "Xe-HP", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

*DG2/Alchemist ({{< xe class="hpg" >}})* や *Ponte Vecchio ({{< xe class="hpc" >}})* に関するメモ書き。  

{{< pindex >}}
 * [DG2-G12](#dg2-12)
 * [DG2 L3 banks](#dg2-l3-banks)
 * [SIMD width](#simd_width)
    * [DPAS/DPASW](#dpas_w)
{{< /pindex >}}

## DG2-G12 {#dg2-12}
Linux Kernel に *DG2-G12* バリアント (サブプラットフォーム) 関連のパッチが投稿された。  
以前から [IGC (intel-graphics-compiler)](https://github.com/intel/intel-graphics-compiler)、[compute-runtime](https://github.com/intel/compute-runtime) に取り込まれた変更から、*DG2* には *G10/G11* に加え、*G12* バリアント (サブプラットフォーム) も存在することが示唆されていた。  
{{< link >}} [3種類存在するかも知れない Intel Alchemist/DG2 と 2種類存在する Ponte Vecchio | Coelacanth's Dream](/posts/2021/12/17/dg2_g10-g11-g12/#dg2) {{< /link >}}
今回のパッチで、ひとまず *G12* バリアントの存在が確定したと言える。  
パッチの内容は初期的なサポートを追加するもので、現時点で *G12* はハードウェアに存在するバグへの回避策や機能面において、*G10/G11* と共通していると推察される。  

 * [[Intel-gfx] [PATCH] drm/i915: Introduce G12 subplatform of DG2](https://lists.freedesktop.org/archives/intel-gfx/2022-January/287309.html)

## DG2 L3 banks {#dg2-l3-banks}
最近の Mesa3D へのパッチの中に、*DG2* の L3キャッシュバンク数に関するものがあった。  

補足すると、Intel はこれまで GPU が持つキャッシュについて、サンプラーごとに持つリードオンリーキャッシュを L2、GPU全体で共有するプログラマブルなキャッシュを L3 としていた。  
それが {{< xe >}}-Core、Vector Engine、Matrix Engine といった用語を打ち出すとともに、キャッシュ階層の呼称が整理され、従来の L3キャッシュを L2キャッシュとするようになった。{{< xe >}}-Core が Sub-Slice、Vector Engine + Matirx Engine が EU に相当する。  
しかし、ドライバ等のシェーダコンパイラ、GPU に最適化されたソフトウェアでは従来の用語を使い続けている。  
{{< link >}} [Intel Architecture Day 2021 個人的まとめ　―― 用語が整理された Xe GPU | Coelacanth's Dream](/posts/2021/08/26/intel-arch-day-2021-xe-gpu/) {{< /link >}}

 * [intel/devinfo: updates for gfx12.5 (!14297) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14297)

以下引用部によれば、*DG2* では Sub-Slice ({{< xe >}}-Core) の有効数が L3キャッシュバンク数と結びついている。  
*DG2 ({{< xe class="hpg" >}})* では Sub-Slice あたり EU 16基 (SP 128基) を持ち、また最近 [Intel(R) Media Driver](https://github.com/intel/media-driver) に追加された記述から、L3キャッシュバンクあたりのキャッシュサイズは 512KB とされる。  
{{< link >}} [Intel Alchemist/DG2 は AV1エンコードをサポートか | Coelacanth's Dream](/posts/2021/12/19/intel-dg2-av1-encode/#l3c) {{< /link >}}
以上の情報を合わせると、Sub-Slice 17-32基 (EU 272-512基) は L3キャッシュバンク 32基 (16MB)、Sub-Slice 9-16基 (EU 144-256基) は L3キャッシュバンク 16基 (8MB)、Sub-Slice 8基以下 (EU 128基以下) は L3キャッシュバンク 8基 (4MB) を持つという構成が考えられる。  

| DG2<br>Sub-Slice | 16< (<=32) | 8< (<=16) | <=8 |
| :-- | :--: | :--: | :--: |
| EU | 272-512 | 144-256 | <=128 |
| L3cache | 32 Banks<br> (16MB?) | 16 Banks<br> (8MB?) | 8 Banks<br> (4MB?) |

 > 		+static void
 > 		+update_l3_banks(struct intel_device_info *devinfo)
 > 		+{
 > 		+   if (devinfo->ver != 12 || devinfo->num_slices != 1)
 > 		+      return;
 > 		+
 > 		+   if (devinfo->verx10 >= 125) {
 > 		+      if (devinfo->subslice_total > 16) {
 > 		+         assert(devinfo->subslice_total <= 32);
 > 		+         devinfo->l3_banks = 32;
 > 		+      } else if (devinfo->subslice_total > 8) {
 > 		+         devinfo->l3_banks = 16;
 > 		+      } else {
 > 		+         devinfo->l3_banks = 8;
 > 		+      }
 > 		+   } else {
 >
 > {{< quote >}} [intel/devinfo: Adjust L3 banks for DG2 (7609e88d) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/7609e88d5fae3b93dd2b70dc5b96066f020f8d11) {{< /quote >}}

L3キャッシュバンク数の構成が 3種類あり、バリアント (サブプラットフォーム) 数と一致するが、ダイ自体のスペックが実際そのようになっているかはまだ不明。  
上位ダイの一部 Sub-Slice と L3キャッシュバンクを無効化し、製品化するというパターンも考えられる。  

## SIMD width {#simd_width}
*{{< xe class="lp" >}}* では EU内に、8-wide FP32/Int32 ALU と 2-wide Extened Math (EM) ALU を持つことが Intel Architecture Day 2020 ですでに発表されている。  
{{< link >}} [Intel Architecture Day 2020 個人的まとめ　―― XeHP は 1-Tile 512EU、XeLPアーキテクチャ詳細 | Coelacanth's Dream](/posts/2020/08/14/intel-architecture-day-2020/#xe-lp-detail) {{< /link >}}
*{{< xe class="hpg" >}}* 、*{{< xe class="Hpc" >}}* はまだ EU (Vector Engine) の詳細な構成は明かされていないが、SIMD幅 (要素数) については OSS のコードから読み取ることができる。  

OneDNN、IGC に記述されている情報によると、*{{< xe class="hpg" >}}* は 8-wide、*{{< xe class="hpc" >}}* は 16-wide だとされている。  

 > 		    int hw_simd() const {
 > 		        switch (hw) {
 > 		            case ngen::HW::Gen9:
 > 		            case ngen::HW::Gen10:
 > 		            case ngen::HW::Gen11:
 > 		            case ngen::HW::XeLP:
 > 		            case ngen::HW::XeHP:
 > 		            case ngen::HW::XeHPG: return 8;
 > 		            case ngen::HW::XeHPC: return 16;
 > 		            default: ir_error_not_expected();
 > 		        }
 > 		        return -1;
 > 		    }
 >
 > {{< quote >}} [oneDNN/bank_conflict_allocation.cpp at e3d0d9ed009a03dc72230037ce39bae1f61d109f · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/e3d0d9ed009a03dc72230037ce39bae1f61d109f/src/gpu/jit/conv/bank_conflict_allocation.cpp#L81-L93) {{< /quote >}}


これは *{{< xe class="hpc" >}}* とそれ以外 (*{{< xe class="lp/hp/hpg" >}}*) で異なる特性、方向性を示している。  
*{{< xe class="hpc" >}}* は 16-wide化とともに、倍精度 (FP64/Int64) にネイティブで対応しており (早期ステッピングでは部分的に非対応)、EU あたりの演算性能は倍以上になっている。GRF (General Register File) も、従来の 32 Bytes から 64 Bytes の増やされている。[^grf-64byte]  
*{{< xe class="hp" >}}* も倍精度演算にネイティブで対応しているが、SIMD幅は 8-wide に留めている。GRF も 32 Bytes のままだ。  
しかし、{{< xe >}}-Core、ドライバ中では従来通り Sub-Slice(s) と呼ぶユニットあたりの EU数では、*{{< xe class="hpc" >}}* は 8基、*{{< xe class="lp/hp/hpg" >}}* では 16基となっており、Sub-Slice あたりで見たときのスループットはそう変わらない。  
また、*{{< xe class="lp/hp/hpg" >}}* では、EU 2基をペアとし、スレッドコントロールを共通化する Fused EU (EU fusion とも) が実装されている。だが *{{< xe class="hpc" >}}* は Fused EU が実装されておらず、EU が単体で扱われる。  

オタクの戯言であるから、確かな考えではないだろうが、SIMD幅を広げることでスレッドあたりの性能を上げたのが *{{< xe class="hpc" >}}* 、スレッドコントロールを共通化することで効率を上げたのが *{{< xe class="lp/hp/hpg" >}}* と見られるのだろうか？  

[^grf-64byte]: [intel-graphics-compiler/Platform.hpp at aee335103dd48a7451f36e3b711c649660ec6dc5 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/aee335103dd48a7451f36e3b711c649660ec6dc5/IGC/Compiler/CISACodeGen/Platform.hpp#L928-L937)

| EU | {{< xe class="lp/hp/hpg" >}} | {{< xe class="hpc" >}} |
| :-- | :--: | :--: |
| SIMD ALU width | 8-wide | 16-wide |
| GRF size per entry | 32 Bytes | 64 Bytes |
| EU per Sub-Slice | 16 EU | 8 EU |
| Fused EU | Y | N |

### DPAS/DPASW {#dpas_w}
*{{< xe class="hp"  >}}* から行列演算のための命令として、DPAS/DPASW (Dot Product Accumulate Systolic /Wide) が用意されている。  
*{{< xe class="hp/hpg" >}}* では DPAS/DPASW をサポートするが、*{{< xe class="hpc" >}}* は DPAS のみをサポートする。  
これは DPASW が Fused EU を想定した命令であるため、その関係で *{{< xe class="hpc" >}}* ではサポートされてない。  

 > 		bool supportDpaswInstruction() const
 > 		{
 > 		    return hasFusedEU() && supportDpasInstruction();
 > 		}
 >
 > {{< quote >}} [intel-graphics-compiler/Platform.hpp at aee335103dd48a7451f36e3b711c649660ec6dc5 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/aee335103dd48a7451f36e3b711c649660ec6dc5/IGC/Compiler/CISACodeGen/Platform.hpp#L890-L893) {{< /quote >}}

oneDNN 内のコメントから、DPASW では内部的に行列ブロックを 2つに分解し、それを Fused EU によってペアとなる EU 2基でそれぞれ実行するものと推察される。  
スレッドコントロールを共通化した Fused EU において、行列演算の性能を向上させるための命令が DPASW だと言えるだろう。  

 > 		    // C is arranged in 8x8 column major blocks organized in a 4x6 row major array (allowing a tile size of up to 32x48).
 > 		    // Each 8x8 block is split in two 8x4 blocks (due to dpasw).
 > 		    // Rearrange into contiguous columns and use hword x4 stores, taking 4 columns at a time.
 > 		    //   (effectively a 4x4 whole-register transpose)
 > 		    // This burns through icache -- consider rewriting with indirect accesses.
 >
 > {{< quote >}} [oneDNN/xe_hp_systolic_gemm_kernel.cpp at 72e9993236c10bd2d676896b600fac05bdcfa2e1 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/72e9993236c10bd2d676896b600fac05bdcfa2e1/src/gpu/jit/gemm/xe_hp_systolic_gemm_kernel.cpp#L897-L901) {{< /quote >}}

{{< ref >}}
 * GRF Size: [intel-graphics-compiler/Platform.hpp at aee335103dd48a7451f36e3b711c649660ec6dc5 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/aee335103dd48a7451f36e3b711c649660ec6dc5/IGC/Compiler/CISACodeGen/Platform.hpp#L928-L937)
 * Gen12 FusedEU description: [intel/ir/gen12+: Work around FS performance regressions due to SIMD32 discard divergence. (!5910) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5910/)
 * HC33 PVC: [Intel's Ponte Vecchio GPU Architecture - hc2021_pvc_final.pdf](https://hc33.hotchips.org/assets/program/conference/day2/hc2021_pvc_final.pdf)
 * DPASW FusedEU: [oneDNN/xe_hp_systolic_gemm_kernel.cpp at 72e9993236c10bd2d676896b600fac05bdcfa2e1 · oneapi-src/oneDNN](https://github.com/oneapi-src/oneDNN/blob/72e9993236c10bd2d676896b600fac05bdcfa2e1/src/gpu/jit/gemm/xe_hp_systolic_gemm_kernel.cpp#L897-L901)
 * Gen12 PDF: [Develop for Intel Graphics—Today and into the Future Presentation - july-gdc-2021-developing-for-intel-graphics-today-and-into-the-future.pdf](https://www.intel.com/content/dam/develop/external/us/en/documents/pdf/july-gdc-2021-developing-for-intel-graphics-today-and-into-the-future.pdf)
 * [the-compute-architecture-of-intel-processor-graphics-gen9-v1d0.pdf](https://www.intel.com/content/dam/develop/external/us/en/documents/the-compute-architecture-of-intel-processor-graphics-gen9-v1d0.pdf)
{{< /ref >}}
