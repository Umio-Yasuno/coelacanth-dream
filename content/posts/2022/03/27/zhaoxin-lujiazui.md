---
title: "Zhaoxin Lujiazui のマイクロアーキテクチャ構成"
date: 2022-03-27T17:37:29+09:00
draft: false
categories: [ "Hardware", "Zhaoxin", "CPU" ]
tags: [ "Lujiazui", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Zhaoxin の MayShao 氏より、GCC のメーリングリストに同社のマイクロアーキテクチャ、コードネーム *Lujiazui (陆家嘴)* の最適化情報を追加するパッチが投稿されている。  
Zhaoxin は中国に本社を置く x86互換CPUの開発、製造企業であり、VIA Technologies の x86-64 のライセンスを保持している。  

*Lujiazui* はデスクトップ/モバイル向けの KX-6000 シリーズと サーバー向けの KH-30000 シリーズに採用されている。  
コアあたり 1スレッド、コア数は両シリーズ共通して最大 8コア。  
キャッシュ構成は、コアあたり L1 Data 32KB、L1 Instruction 32KB、複数コアで共有する L2キャッシュが最大 8MB。  
KX-6640A/6640MA のみ 4C/4T、L2キャッシュ 4MB という仕様になっており、またブーストクロック機能は KX-6640MA のみ有効化されている。  

CPUID における Family, Model, Stepping は `Family: 0x7, Model: 0x3B (59)` となっている。`(CPUID EAX: 0x000307B0)`  

メモリは DDR4-2666、デュアルチャネルをサポート。  
他 I/O には、PCIe 3.0 x16、USB 3.1 x2、USB 2.0 x4、SATA 3.2 x2 を備える。  

* [[PATCH] [x86_64] Zhaoxin lujiazui enablement](https://gcc.gnu.org/pipermail/gcc-patches/2022-March/592269.html)

{{< pindex >}}
 * [対応命令](#isa)
 * [フロントエンド部](#frontend)
 * [バックエンド部](#backend)
{{< /pindex >}}

## 対応命令 {#isa}

*Lujiazui* はベクトル命令に SSE4.2 /AVX、ビット操作命令に ABM (LZCNT) /BMI /BMI2 /POPCNT をサポートしている。  
下引用部には記述されていないが、公式サイトの仕様と [Linux Hardware Database](https://linux-hardware.org/) にある `lscpu` 等の実行結果によれば SHA_NI 命令もサポートしているはずである。[^db]  

 > 		+@item lujiazui
 > 		+ZHAOXIN lujiazui CPU with x86-64, MOVBE, MMX, SSE, SSE2, SSE3, SSSE3, SSE4.1,
 > 		+SSE4.2, AVX, POPCNT, AES, PCLMUL, RDRND, XSAVE, XSAVEOPT, FSGSBASE, CX16,
 > 		+ABM, BMI, BMI2, F16C, FXSR, RDSEED instruction set support.
 > 		+
 >
 > {{< quote >}} [[PATCH] [x86_64] Zhaoxin lujiazui enablement](https://gcc.gnu.org/pipermail/gcc-patches/2022-March/592269.html) {{< /quote >}}

[^db]: [HW probe of Shanghai Zhaoxin Semiconductor ZXE CRB #7fe4a3390b: cpuinfo](https://linux-hardware.org/?probe=7fe4a3390b&log=cpuinfo)

## フロントエンド部 {#frontend}

*Lujiazui* マイクロアーキテクチャは 3-wide デコーダー (decoder 0/1/2) を持ち、1個の uOp として実行される命令はすべてのデコーダーで 1サイクルで処理できる。  
ロード/ストア命令と組み合わされるような、2個の uOp に分解、実行される命令は decoder 0/1 のみで処理可能という制限があるが、1サイクルという点は変わらない。  
マイクロコードで実装されている一部の複雑な命令は decoder 0 のみで処理可能であり、不特定数のサイクルを必要とする。  

 > 		+;; The rules for the decoder are simple:
 > 		+;;  - an instruction with 1 uop can be decoded by any of the three
 > 		+;;    decoders in one cycle.
 > 		+;;  - an instruction with 2 uops can be decoded by decoder 0 or decoder 1
 > 		+;;    but still in only one cycle.
 > 		+;;  - a complex (microcode) instruction can only be decoded by
 > 		+;;    decoder 0, and this takes an unspecified number of cycles.
 > 		+;;
 > 		+;; The goal is to schedule such that we have a few-one-two uops sequence
 > 		+;; in each cycle, to decode as many instructions per cycle as possible.
 > 		+(define_cpu_unit "lua_decoder0" "lujiazui_decoder")
 > 		+(define_cpu_unit "lua_decoder1" "lujiazui_decoder")
 > 		+(define_cpu_unit "lua_decoder2" "lujiazui_decoder")
 >
 > {{< quote >}} [[PATCH] [x86_64] Zhaoxin lujiazui enablement](https://gcc.gnu.org/pipermail/gcc-patches/2022-March/592269.html) {{< /quote >}}

## バックエンド部 {#backend}

*Lujiazui* マイクロアーキテクチャは Out-of-Order 実行が実装されている。  
6本の実行パイプラインを備え、内 2本 (Port 4/5) は AGU (Address Generation Unit)、Load/Store Unit となっている。  
他 4本 (Port0/1/2/3) は汎用となっている。  

 > 		+;; The out-of-order core has six pipelines.
 > 		+;; Port 4, 5 are responsible for address calculations, load or store.
 > 		+;; Port 0, 1, 2, 3 for everything else.
 > 		+
 > 		+(define_cpu_unit "lua_p0,lua_p1,lua_p2,lua_p3" "lujiazui_core")
 > 		+(define_cpu_unit "lua_p4,lua_p5" "lujiazui_agu")
 > 		+
 > 		+(define_reservation "lua_p03" "lua_p0|lua_p3")
 > 		+(define_reservation "lua_p12" "lua_p1|lua_p2")
 > 		+(define_reservation "lua_p1p2" "lua_p1+lua_p2")
 > 		+(define_reservation "lua_p45" "lua_p4|lua_p5")
 > 		+(define_reservation "lua_p4p5" "lua_p4+lua_p5")
 > 		+(define_reservation "lua_p0p1p2p3" "lua_p0+lua_p1+lua_p2+lua_p3")
 >
 > {{< quote >}} [[PATCH] [x86_64] Zhaoxin lujiazui enablement](https://gcc.gnu.org/pipermail/gcc-patches/2022-March/592269.html) {{< /quote >}}

`lujiazui_cost` には以下のような記述があるが、4コアごとに L2キャッシュ 4MB を持つ構成になっているのかもしれない。AMD Zen の CCX よりかは、Intel Atom のモジュール構成が近いと言える。  

 > 		+  32,				  	/* size of l1 cache.  */
 > 		+  4096,					/* size of l2 cache.  */
 >
 > {{< quote >}} [[PATCH] [x86_64] Zhaoxin lujiazui enablement](https://gcc.gnu.org/pipermail/gcc-patches/2022-March/592269.html) {{< /quote >}}

また、3-wide デコーダー、実行パイプライン 6本、その内 2本が AGU というマイクロアーキテクチャ規模だけで言えば、Intel Atom Goldmont Plus あたりが近いかもしれない。AVX命令や各ビット操作命令をサポートするという違いはあるが。  

{{< ref >}}
 * [兆芯 - Wikipedia](https://ja.wikipedia.org/wiki/%E5%85%86%E8%8A%AF)
 * [Zhaoxin Displays x86-Compatible KaiXian KX-6000: 8 Cores, 3 GHz, 16 nm FinFET](https://www.anandtech.com/show/13388/zhaoxin-shows-x86-compatible-kaixian-kx6000)
 * [开先® KX-6000系列处理器 - 开先®KX-6000系列处理器 - 好用的电脑芯！](https://www.zhaoxin.com/prod_view.aspx?nid=3&typeid=129&id=327)
 * [开胜® KH-30000系列处理器 - 开胜®KH-30000处理器 - 好用的电脑芯！](https://www.zhaoxin.com/prod_view.aspx?nid=3&typeid=95&id=322)
 * [HW probe of Shanghai Zhaoxin Semiconductor ZXE CRB #7fe4a3390b: cpuinfo](https://linux-hardware.org/?probe=7fe4a3390b&log=cpuinfo)
{{< /ref >}}
