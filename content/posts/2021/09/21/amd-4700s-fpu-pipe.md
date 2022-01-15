---
title: "FPU部を削減した AMD 4700S プロセッサ"
date: 2021-09-21T22:51:03+09:00
draft: false
tags: [ "AMD_4700S", "Zen_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

チップに関する解説や情報発信といった活動を行っている [Nemez (@GPUsAreMagic)](https://twitter.com/GPUsAreMagic) 氏 ([Nemez | Patreon](https://www.patreon.com/nemezor)) と、  
[Hardwareluxx](https://www.hardwareluxx.de/) のエディターである [Andreas Schilling (@aschilling)](https://twitter.com/aschilling) 氏により、*Zen 2 アーキテクチャ* を採用するとしている *AMD 4700S* と同アーキテクチャの *Ryzen Threadripper 3960X* の IPC 計測結果を比較したものが公開された。  

 * <https://twitter.com/GPUsAreMagic/status/1439205765559066627>
 * [PS5-Custom-Chip mit Einschränkungen: Das Ryzen 4700S Desktop Kit im Test - Hardwareluxx](https://www.hardwareluxx.de/index.php/artikel/hardware/prozessoren/57076-ps5-custom-chip-mit-einschraenkungen-das-ryzen-4700s-desktop-kit-im-test.html)

IPC の計測には Nemez 氏が開発した [Nemes / IPC Tests · GitLab](https://git.nemez.net/nemes/ipc-tests) が用いられている。テストの中身としては同じ命令をレジスタ依存が発生しないように複数並べることで IPC を計測している。  

 * 参考: [命令単体の性能を計測する](http://nano.flop.jp/txt/instruction/index.html)

計測結果から *AMD 4700S* では FPU部に通常の *Zen 2 アーキテクチャ* から変更、削減された部分があると考えられるため、FPU部を中心に書いていく。  

*Zen/+/2 アーキテクチャ* では、FPU部に 4本の実行パイプラインを持ち、2x MUL + 2x ADD で構成されている。  
[Software Optimization Guide for AMD Family 17h Models 30h and Greater Processors](https://www.amd.com/en/support/tech-docs?keyword=Family+17h+Model+30h) によれば、*Zen 2 アーキテクチャ* の 各実行パイプと実行ユニットの対応関係は以下のようになっている。  

| Unit | Pipe 0 <br> (MUL) | Pipe 1 <br> (MUL) | Pipe 2 <br> (ADD) | Pipe 3 <br> (ADD) |
| :--  | :--:         | :--:         | :--:         | :--:         |
| FMUL | X | X | | |
| FADD | | | X | X |
| FCVT | | | | X |
| FDIV | | | | X |
| FMISC | X | X | X | X |
| STORE | | | X | |
| VADD | X | X | | X |
| VMUL | X | | | |
| VSHUF | | X | X | | |
| VSHIFT | | | X | |
| VMISC | X | X | X | X |
| AES | X | X | | |
| CLM[^clm] | | S | | |

{{< link >}} [Software Optimization Guide for AMD Family 17h Models 30h and Greater Processors](https://www.amd.com/en/support/tech-docs?keyword=Family+17h+Model+30h) {{< /link >}}

Nemez 氏は結果から、*AMD 4700S* では通常の *Zen 2 アーキテクチャ* から FPUパイプ/ポート が 2本に削減されていると述べている。  
自分も 2本の FPUパイプ、MUL, ADD が 1本ずつ減らされていると考えている。  

そう考えた理由をつらつらと書いていく。  
まず `{add, sub, mul, madd, div}_{avx256, sse128}_float` の性能が *AMD 4700S* ではほぼ半減している。  
`avx256, sse128` での性能が変わらないことから、*AMD 4700S* の FPU はデータ幅 256-bit であり、この点は通常の *Zen 2 アーキテクチャ* と同じである。  
`mul_{avx256, sse128}_int` の性能は変わらないが、これは通常の *Zen 2 アーキテクチャ* でも VMUL (`VPMULLD/PMULLD`) を実行可能なのは Pipe 0 のみとなっているためと思われる。  
`{add, sub}_{avx256, sse128}_int` の性能は 2/3 (-33%) となっているが、VADD (`VPADDD/PADDD`) は *Zen 2 アーキテクチャ* では Pipe0/1/3 で実行可能となっている。MUL と ADD のパイプを 1本ずつ減らし、残った 2本のどちらも VADD が実行可能とした場合 2/3 となり計測結果と一致する。  
また、MUL パイプを 1本減らしているとすると、AES の性能も半減しているものと思われる。  

`div_{avx256, sse128}_float` も元から Pipe 3 のみで実行可能となっているが、*AMD 4700S* ではこの FDIV (`VDIVPS/DIVPS`) 性能もほぼ半減している。  
これについては正直自分では分からない。  
「Family 17h Models 30h and Greater Instruction Latencies version_1.00」では IPC (Throughput) が 0.20 とされているのに、*3960X* の計測結果では 0.284-0.287 となっていることについては、  
FDIVユニットはパイプが占有中でも 2つのオペレーションを同時に行える点が関係しているのかもしれないが、これは *Zen アーキテクチャ* にも共通し、手元の *Ryzen 5 2600* で試した所、「Family 17h Instruction Latencies version_1.00」に記載されているのと近い値が出た。  
今回調べるにあたって参考とした [tanakamura (Takashi Nakamura)](https://github.com/tanakamura) 氏が開発している [instruction-bench](https://github.com/tanakamura/instruction-bench) でも、*Zen 2 アーキテクチャ* の `VDIVPS/DIVPS` は IPC= 0.29 となっている。[^inst-bench]  
それとして、*AMD 4700S* の FDIV 性能が半減していることについて、FDIVユニットが 1つのオペレーションのみをサポートするよう変更された可能性があるが、はっきりとしない。  

[^inst-bench]: [instruction-bench/znver2.log at master · tanakamura/instruction-bench](https://github.com/tanakamura/instruction-bench/blob/master/znver2.log#L98)

`zen_fpu_mix_{21, 22}` は *Zen/+/2 アーキテクチャ* のレジスタポートにある特有の仕様をテストするもので、各 FPUパイプにはレジスタポートが 2本備えられているが、FMA のような 3 ソースオペランドを必要とする命令を実行する際、Pipe 0/1 (MUL) は Pipe 3 (ADD) のレジスタポートを 1つ使う。  
そのため、FMA と ADD が混ざった命令群を実行するとき Pipe 3 (ADD) の実行が止まる可能性出てくる。*Zen 2 アーキテクチャ* の場合はピーク IPC が FPUパイプ数の 4 ではなく、3 に近くなる。  
それをテストするのが `zen_fpu_mix_{21, 22}` であり、`zen_fpu_mix_21` は 2x FMA (`VFMADD132PS`) + 1x FADD (`VADDPS`)、`zen_fpu_mix_22` は 2x FMA + 2x FADD の塊となっている。  

だが *AMD 4700S* では 2 に近く、FADD をブロックすることなく FMA が実行できるよう Pipe 0 にレジスタポートが 3本備えられている *可能性* があるが、単にうまいこと命令を並び替えて実行している可能性もある。  

また、`STORE` や `FCVT` 等、ここで計測されなかったものを処理するためのパイプが、演算メインのパイプとは別に持っている可能性もある。  

| [AMD 4700S]<br>Unit | Pipe 0 <br> (MUL ?) | Pipe 1 <br> (ADD ?) |
| :-- | :--: | :--: |
| FMUL | X | |
| FADD | | X |
| FCVT | | X ? |
| FDIV | | X ? |
| FMISC | X | X |
| STORE | | X ? |
| VADD | X | X |
| VMUL | X | X |
| VSHUF | X ? | X ? |
| VSHIFT | | X ? |
| VMISC | X | X |
| AES | X |
| CLM | S ? | |

[^clm]: Carry-less Multiply, Zen 2 supported by ucode.

*Zen/+/2 アーキテクチャ* の FPU実行パイプは 2x MUL + 2x ADD という構成だが、細かい部分で実行できる命令はそれぞれ異なり実際には非対称構成となる。  
*AMD 4700S* では FPU実行パイプを 2本に減らしたという推測が当たっているとすると、そうした部分で再設計が必要になると思われる。  
それによって CPU 全体のクロックを下げずに、FPU部に掛かる負荷とそれによる熱の発生を抑えられる等はあるかもしれないが、手間を掛けて FPU部のさらに一部とはいえ再設計、再構成をするに見合うのだろうか。  
FPU のデータ幅を 256-bit から 128-bit にするのであれば *Zen/+ アーキテクチャ (znver1)* のコンパイラにおける最適化を流用できそうだが、パイプ構成を変えたのであればコードを修正する範囲が広がる。  
謎の *AMD 4700S* が製品として出回るようになってもまだ謎は多く残っている。  

それと、やはりただの一オタクの推測であるため、ツッコミ、指摘がある場合は [Contact](/about/#contact) から連絡してくれると自分が喜ぶ。  
