---
title: "Intel Grand Ridge のマイクロアーキテクチャは Crestmont に"
date: 2023-03-22T02:35:47+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Grand_Ridge", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日、Intel が公開する電力管理ツール Pepc (Power, Energy, and Performance Configurator) に追加されたコミットにより、*Grand Ridge* CPU のマイクロアーキテクチャに *Crestmont* が採用されていることが明らかにされた。  
また、Pepc では *Grand Ridge (Logansville プラットフォーム)* を、*Snow Ridge (Tremont)* や *Denverton (Goldmont)* のようなマイクロサーバー向けの CPU としている。  

 >         +CRESTMONTS =    (INTEL_FAM6_GRANDRIDGE,)
 >
 > {{< quote >}} [pepclibs: fix spelling of "crestmont" · intel/pepc@578ac4c](https://github.com/intel/pepc/commit/578ac4c7669748873d9cd867aa099d88a0ee6e20) {{< /quote >}}

 * [pepclibs: fix spelling of "crestmont" · intel/pepc@578ac4c](https://github.com/intel/pepc/commit/578ac4c7669748873d9cd867aa099d88a0ee6e20)

*Grand Ridge* はまだ Intel の公式ロードマップには追加されていないが、ISA ドキュメントや OSS 内ではすでに名前が出ている。  
対応する命令範囲も公開されており、*Gracemont* と比較すると `AVX-{IFMA, VNNI-INT8, NE-CONVERT}, CMPCCXADD, RAOINT` が追加されている。  
*Grand Ridge* が *Crestmont* だとすると、*Grand Ridge* と *Sierra Forest* の対応命令範囲が完全には一致しないことが少し不思議に思えてくる。  
現状 `RAOINT (Remote Atomic Operation)` に対応する CPU は *Grand Ridge* のみとされ、*Sierra Forest* は含まれていない。  
*Sierra Forest* が採用するマイクロアーキテクチャを Intel は正式には公開していないが、Intel 3 プロセスで製造され、ロードマップ上では 2024年に位置していることは発表されている。  
*Sierra Forest* では *Crestmont* とは若干異なるマイクロアーキテクチャを採用するのか、それとも `RAOINT` を無効化する理由が何かあるのか。  

*Crestmont* は *Meteor Lake* の Atom 系コア (Small, E-Core) にも採用されていることが明らかになっているが、*Meteor Lake* の対応する命令範囲に `AVX-{IFMA, VNNI-INT8, NE-CONVERT}, CMPCCXADD` はない。  
これは Core 系 (Big, P-Core) の *Redwood Cove* がそれら命令に対応しておらず、ハイブリッドアーキテクチャの関係上無効化されているのだと思われる。  

{{< ref >}}
 * [intel/pepc: Pepc - Power, Energy, and Performance Configurator](https://github.com/intel/pepc)
 * [x86 Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)
 * [Intel Sierra Forest, Grand Ridge, Granite Rapids でサポートされる新命令 | Coelacanth's Dream](/posts/2022/10/04/intel-ise-rev_46/#rao-int)
{{< /ref >}}
