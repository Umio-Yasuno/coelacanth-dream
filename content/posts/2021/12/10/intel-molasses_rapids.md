---
title: "謎？のコードネーム、Intel Molasses Rapids"
date: 2021-12-10T10:54:29+09:00
draft: false
tags: [ "Unknown" ]
# keywords: [ "", ]
categories: [ "Note", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Intel のソフトウェアエンジニア、[kilobyte (Adam Borowski)](https://github.com/kilobyte) 氏よりわずかに語られた謎のコードネームを持つ CPU *Molasses Rapids* についてのメモ。  

## Molasses Rapids {#molasses-rapids}

まず、自分が *Molasses Rapids* の名を最初に見たのは、オープンソースで開発される `x86/x86-64` の逆アセンブラライブラリ [zydis](https://zydis.re/) への Issue だった。  

 * [Question: supported instructions · Issue #252 · zyantific/zydis](https://github.com/zyantific/zydis/issues/252)
 * [#995993 - RFS: zydis/3.1.0-1 [ITP] -- fast and lightweight x86/x86-64 disassembler library - Debian Bug report logs](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=995993#12)

Github での Issue の主内容は、zydis は特徴としてすべての x86/x86-64 (AMD64) 命令とその拡張をサポートしている (*Supports all x86 and x86-64 (AMD64) instructions and extensions*) ことを挙げているが、拡張命令は追加され続けており、バージョンによってはその特徴が正しくないことになる、ということについて。  

そして、発端となったのが [kilobyte (Adam Borowski)](https://github.com/kilobyte) 氏による以下のコメント。  

 > 		There's an issue in the long desc that would be nice to fix in -2:
 > 		> [...] It supports all x86 and AMD64 instructions and extensions [...]
 > 		which is a lie.  I don't see Sapphire Rapids stuff, for example.
 > 		The desc should limit that claim to something like: "supports all x86
 > 		instructions and extensions up to Molasses Rapids/Intel and Hubris
 > 		Ridge/AMD".
 >
 > {{< quote >}} [#995993 - RFS: zydis/3.1.0-1 [ITP] -- fast and lightweight x86/x86-64 disassembler library - Debian Bug report logs](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=995993#12) {{< /quote >}}

コメントでは、*すべての* 命令と拡張命令をサポートしているというのは間違っており、 *Molasses Rapids/Intel, Hubris Ridge/AMD* までの範囲、と限る必要があるとしている。  
上記コメント時点 (2021/10/12) では *Sapphire Rapids* に実装されているいくつかの命令を zydis ではサポートしていない、といったようなコメントがあるが、実際にはその少し前に `AMX`、`AVX512_FP16` といった命令をサポートするコミットが取り込まれている。  
それとさり気なく *Hubris Ridge/AMD* というコードネームも出てきているが、とりあえず置いておく。  

もう1つ [kilobyte (Adam Borowski)](https://github.com/kilobyte) 氏が *Molasses Rapids* について言及しているのが、[PMDK: Persistent Memory Development Kit](https://github.com/pmem/pmdk) でのある Issue/Pull  request。  
日付的には、上記コメントよりこちらの方が先となる。(2021/08/23)  

 * [Make Check failed on Arm64 · Issue #5255 · pmem/pmdk](https://github.com/pmem/pmdk/issues/5255)
 * [Add Arm64 specified geting cache line size by kevinzs2048 · Pull Request #5271 · pmem/pmdk](https://github.com/pmem/pmdk/pull/5271)
    * <https://github.com/pmem/pmdk/pull/5271#issuecomment-903468247>

ユニットテストにおいてキャッシュラインサイズの取得に、まず `/sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size` を確認し、存在した場合はその値を用いる。  
`arm64 (aarch64)` の場合はその sysfsファイルが存在しないため、別の取得方法に変更するという Pull request だが、シェルスクリプト内の該当部分では、取得に失敗した場合 `|| (OR)` 演算子によって、`64` をキャッシュラインサイズとして用いている。  
それでも実際のサイズとは異なる場合が出てくるが、テストであるため性能に最適化する必要性は薄い。  

 > 		This code started as x86-only, and yet it reads the cacheline size (and only for CPU 0!). It seems to anticipate enlarging the cacheline size in a future processor. Let's call that one "Molasses Rapids".
 > 		
 > 		If the initiator is a Cascade Lake, target is Molasses Rapids — we flush twice as often as required, which is correct and at most slightly unoptimal.
 > 		If the initiator is a Molasses Rapids yet the target is Cascade Lake, we flush every 128 bytes, leaving every other cacheline unflushed.
 >
 > {{< quote >}} <https://github.com/pmem/pmdk/pull/5271#issuecomment-903468247> {{< /quote >}}

[kilobyte (Adam Borowski)](https://github.com/kilobyte) 氏は、最近の x86/x86-64 CPU はキャッシュラインサイズを 64 (Byte) とする中、最初に `.../cpu0/cache/index0/coherency_line_size` を確認、取得する理由を、将来のプロセッサでキャッシュラインサイズを拡大することを見込んでのものではないか、としている。  
そしてそのプロセッサを *Molasses Rapids*、キャッシュラインサイズを 128 Byte として例えに用いている。  

### 安易な推測 {#easy-guess}

*Intel Molasses Rapids* と一緒に名前だけ出てきた *AMD Hubris Ridge* だが、こうして自分でネタに取り上げながら言うのもなんだが、単に [kilobyte (Adam Borowski)](https://github.com/kilobyte) 氏が説明する時に用いる仮のコードネームであるように自分は思う。  
zydis でのコメントで目的としているのは、命令と拡張命令のサポートが、実際にはある範囲までに限られることを伝えることと考えられ、PMDK においてもイニシエータとターゲットでキャッシュラインサイズが異なる場合の操作を説明することが目的と考えられる。  
*Molasses Rapids, Hubris Rapids* で検索した時、これまでに挙げたページ以外に出てこなかったことも後押しする。あまりに情報が無さすぎる。  

ちなみに *Molasses* は糖蜜、サトウキビの糖液という意味だが、そこからひどくのろいものを指すのにも使われるらしい。*Hubris* は破滅へと至る慢心あるいは傲慢の意。  

それとキャッシュラインを拡大することの利点として、メモリとキャッシュ間のやり取りはキャッシュライン単位であるため、メモリ帯域を活用しやすくなることが挙げられる。  

