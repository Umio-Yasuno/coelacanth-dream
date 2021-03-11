---
title: "Intel Sapphire Rapids は AVX512_FP16 をサポート"
date: 2021-01-11T10:24:10+09:00
draft: false
tags: [ "Sapphire_Rapids", "Alder_Lake" ]
# keywords: [ "", ]
categories: [ "Intel", "CPU", "Hardware" ]
noindex: false
# summary: ""
toc: false
---

モバイル向けでは *Ice Lake (client)* から、デスクトップ向けでは 2021年第一四半期での登場が予告されている *Rocket Lake* からついにサポートされる AVX512命令。  
これまでに何度も拡張を繰り返して来たが、*Sapphire Rapids* ではさらに拡張された `AVX512_FP16` がサポートされる。  

Linux Kernel に投稿されたパッチのコメントによれば、`AVX512_FP16` は *Sapphire Rapids* 等の Intelプロセッサでサポートされ、精度や大きさを確保しつつ、FP32 よりも高速で優れた性能を得られる、としている。  

 > AVX512_FP16 is supported by Intel processors, like Sapphire Rapids.  
 > It could gain better performance for it's faster compared to FP32  
 > while meets the precision or magnitude requirement. It's availability  
 > is indicated by CPUID.(EAX=7,ECX=0):EDX[bit 23]  
 >
 > {{< quote >}} [LKML: Kyung Min Park: [PATCH 2/2] x86: Expose AVX512_FP16 for supported CPUID](https://lkml.org/lkml/2020/12/7/1443) {{< /quote >}}

AVX512系命令では、`AVX512_VNNI` にて精度を下げて複数の要素を扱うことで処理性能を向上させることができ、データ精度 FP32 の場合に対してピーク理論性能が、 Int8 では 4倍、Int16 では 2倍となる。(ただし、メモリ性能がボトルネックとなるため、実際の性能はそれより低くなる可能性がある。)  
`AVX512_FP16` はこれを拡張したものと思われ、ピーク理論性能は Int16 と同じとなるが、FP16 であれば Int16 よりも数値が広い範囲で保持される。  

AVX512、FP16 というキーワードで、過去にバグ報告プラットフォームである [intel in Launchpad](https://launchpad.net/intel) にちらと出てきた `HFNI` 命令が思い出される。  
{{< link >}} [Intel Alder Lake、Sapphire Rapids にて追加される2つの命令 ――AVX2 VNNI /HFNI | Coelacanth's Dream](/posts/2020/05/26/intel-adl-spr-new-inst/) {{< /link >}}
その時のリンク先は消されてしまったが、メールアーカイブには残っていた。[^hfni]  
`HFNI` は AVX512命令において、FP32 の代わりに FP16 を使用し、性能を最大 2倍に引き上げるものと説明されている。  
*SPR (Sapphire Rapids)* で新しくサポートされる命令としているが、タイトルには *ADL (Alder Lake)* があることから、*Sapphire Rapids* と言うより *Golden Cove アーキテクチャ* で新たにサポートする命令と考えられる。  
用途として 5G基地局等が想定されており、Intel が過去に CPUマイクロアーキテクチャのロードマップを発表した時、*Golden Cove* の特徴に Network/5G Perf があったことも上と一致する。  

[^hfni]: [[Bug 1879865] Re: [ADL] Enable 5G ISA (FP16) / HFNI](https://www.mail-archive.com/ubuntu-bugs@lists.ubuntu.com/msg5786181.html)

`AVX512_FP16` と `HFNI` の関係については、`HFNI` が未だに Intel の {{< comple >}} 一般にも公開されている限りだが {{< /comple >}} 公式ドキュメントに現れていないことから (`AVX512_FP16` はある)、違うのは名前だけで、命令の中身は一緒ではないかと思われる。  
かなり似ていながら僅かに違いが存在するとかだと、ただでさえややこしい AVX512命令群がさらにややこしくなってしまう。  


## Intel AVX サポート表

CPUマイクロアーキテクチャの違いと併せて、対応範囲が混沌としている AVX512命令群だが、GCC 等コンパイラのドキュメントを頼りに頑張って整理したのが下の表となる。  
プロセッサのコードネームを略称で記載しているが、一応書いておくと、*SKX (Skylake server)* 、*CNL (Cannon Lake)* 、*CLX (Cascade Lake)* 、*CPX (Cooper Lake)* 、*ICL (Ice Lake client)* 、*ICX (Ice Lake server)* 、*TGL (Tiger Lake)* 、*SPR (Sapphire Rapids)* 、*ADL (Alder Lake)* と対応している。  
また、*Alder Lake* に関しては、[Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/content/www/us/en/develop/download/intel-architecture-instruction-set-extensions-programming-reference.html) に 「Intel Hybrid Technology は AVX512 をサポートしない」とあり、そのためアーキテクチャ自体がサポートしていない場合と区別してある。  

AVX512命令は **Xeon Phi シリーズ** 、*Knight’s Landing* 、*Knights Mill* でも一部サポートされているが、それも含めるとさらに表が肥大化してしまうため省いた。5つの命令をまとめている 2行目を細かく分ける必要が出てしまう。  
それと *Rocket Lake* も省いているが、その *Crypress Cove アーキテクチャ* は *Ice Lake (client)* と同じ機能を備えることから、AVX512命令の範囲も同様と思われる。  

| Intel AVX | SKX | CNL | CLX | CPX | ICL | ICX | TGL | SPR | ADL |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| AVX512F (Foundation),<br>AVX512CD (Conflict Detection),<br>AVX512BW (Byte and Word),<br>AVX512DQ (Doubleword and Quadword),<br>AVX512VL (Vector Length) | Y | Y | Y | Y | Y | Y | Y | Y | - |
| AVX512_IFMA (Integer Fused Multiply-Add),<br>AVX512_VBMI (Vector Byte Manipulation Instructions) |  | Y | Y | Y | Y | Y | Y | Y | - |
| AVX512_VNNI |  |  | Y | Y | Y | Y | Y | Y | - |
| AVX512_VBMI2 (Vector Byte Manipulation Instructions 2),<br>AVX512_VPOPCNTDQ,<br>AVX512_BITALG (Bit Algorithms),<br>GFNI 512-bit (Galois Field New Instructions),<br>VAES 512-bit (Vector AES instructions) |  |  |  |  | Y | Y | Y | Y | - |
| AVX512_BF16 |  |  |  | Y |  |  |  | Y | - |
| AVX512_VP2INTERSECT |  |  |  |  |  |  | Y | Y | - |
| AVX512_FP16 |  |  |  |  |  |  |  | Y | - |
| AVX_VNNI (128/256-bit) |  |  |  |  |  |  |  | Y | Y |

{{< ref >}}

 * [Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/content/www/us/en/develop/download/intel-architecture-instruction-set-extensions-programming-reference.html)
 * [Intel® Advanced Vector Extensions 512 (Intel® AVX-512) Overview](https://www.intel.com/content/www/us/en/architecture-and-technology/avx-512-overview.html)
 * [Lowering Numerical Precision to Increase Deep Learning Performance](https://www.intel.com/content/www/us/en/artificial-intelligence/posts/lowering-numerical-precision-increase-deep-learning-performance.html)
 * [[X86] Support Intel avxvnni · llvm/llvm-project@756f597](https://github.com/llvm/llvm-project/commit/756f5978410809530150f5e1cd425e85ad94d1cd)
 * [x86 Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)

{{< /ref >}}
