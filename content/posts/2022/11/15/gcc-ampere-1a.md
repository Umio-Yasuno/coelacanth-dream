---
title: "GCC に Ampere-1A CPU のサポートを追加するパッチ"
date: 2022-11-15T05:26:31+09:00 
draft: false 
categories: [ "Hardware", "Ampere", "CPU" ]
tags: [ "Arm", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Philipp Tomsich 氏より、GCC に *Ampere-1A* CPU のサポート、最適化情報を追加するパッチが投稿されている。  

 * [[PATCH v2] aarch64: Add support for Ampere-1A (-mcpu=ampere1a) CPU](https://gcc.gnu.org/pipermail/gcc-patches/2022-November/606084.html)

Philipp Tomsich 氏は以前に *Ampere-1* に向けた同様のパッチを投稿している。[^ampere-1]  
*Ampere-1* と *Ampere-1A* の間で、Armv8.6-A ISA、SVE (Scalable Vector Extensions) 非対応、命令の発行レート (issue rate) 4命令という点は共通している。  
異なるのは *Ampere-1A* で MTE (Memory Tagging Extension) の対応追加、命令融合 (instruction fusion) のパターンを 1個追加、実行ユニットの命令タイミング、レイテンシが若干変更した点となる。  

instruction fusion では、レジスタ 2個の値と定数 1 を加算 (A+B+1)、あるいは 2個の値と定数 1 を減算 (A-B-1) するパターンが追加された。  

命令の発行レートや命令タイミング、レイテンシから、*Ampere-1A* のマイクロアーキテクチャは *Ampere-1* から性能についてはあまり変わらないように見える。  
おそらくは MTE (Memory Tagging Extension) の対応追加が主要な変更と思われる。  
MTE はセキュリティ向けの機能であり、メモリ割り当て時にタグ付けを行い、そのメモリ位置を参照するポインタと関連付け、アクセス時にタグの一致を確認することで解放後メモリの使用 (use-after-free)、バッファオーバーフローといったバグをハードウェアで検出できる。  

Arm のサーバー向け IP では、*Neoverse-N2, Demeter/Neoverse-V2, Neoverse-E2 (Cortex-A510)* から MTE に対応しており、*Ampere-1A* でその動きに追従したとも見える。  

 >          /* Ampere Computing ('\xC0') cores. */
 >          AARCH64_CORE("ampere1", ampere1, cortexa57, V8_6A, (F16, RNG, AES, SHA3), ampere1, 0xC0, 0xac3, -1)
 >         +AARCH64_CORE("ampere1a", ampere1a, cortexa57, V8_6A, (F16, RNG, AES, SHA3, MEMTAG), ampere1a, 0xC0, 0xac4, -1)
 >         
 > {{< quote >}} [[PATCH v2] aarch64: Add support for Ampere-1A (-mcpu=ampere1a) CPU](https://gcc.gnu.org/pipermail/gcc-patches/2022-November/606084.html) {{< /quote >}}

[^ampere-1]: [Ampere-1 のマイクロアーキテクチャを見る | Coelacanth's Dream](/posts/2022/05/27/ampere-1-arch/)

{{< ref >}}
 * [Arm MTE architecture: Enhancing memory safety - Architectures and Processors blog - Arm Community blogs - Arm Community](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/enhancing-memory-safety)
 * [Arm Memory Tagging 拡張機能  |  Android オープンソース プロジェクト  |  Android Open Source Project](https://source.android.com/docs/security/test/memory-safety/arm-mte)
 * [Arm Announces Neoverse V2 and E2: The Next Generation of Arm Server CPU Cores](https://www.anandtech.com/show/17575/arm-announces-neoverse-v2-and-e2-the-next-generation-of-arm-server-cpu-cores)
{{< /ref >}}
