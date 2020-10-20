---
title: "Zen_3 Support New Inst"
date: 2020-10-20T08:54:39+09:00
draft: true
tags: [ "Zen_3", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
toc: false
---

自信が無いので没。  

バイナリやオブジェクトファイルの操作に使われるツール [binutils](https://www.gnu.org/software/binutils/) に *znver3 / Zen 3 アーキテクチャ* をサポートするパッチが投稿され、この世代で新たにサポートされる x86命令が明かされた。  
{{< link >}} [[PATCH] Add znver3 processor](https://sourceware.org/pipermail/binutils/2020-October/113674.html) {{< /link >}}

## INVLPGB / TLBSYNC

`INVLPGB (Invalidate TLB Entry(s) with Broadcast)` は TLBエントリを無効化する命令、`TLBSYNC (Synchronize TLB Invalidations)` はそれを各コアで実行、応答したことを保証する同期命令となる。  

## SNP (Secure Nested Paging)

## INVPCID (Invalidate TLB Entry(s) in a Specified PCID)
指定した `PCID (Process Context Identifier)` 

## OSPKE (OS Memory Protection Key)

## VAES 
`VAES (Vector AES)` は AES暗号化のエンコード/デコードをベクトルで実行する命令であり、Intel CPU では *Sunny Cove (Ice Lake Client/Server)* からサポートされている。  
ソフトウェアでは、オープンソースで開発される圧縮/展開ソフト、7-zip で使われている。  





{{< ref >}}

 * [AMD Sends Out Patches Adding "Znver3" Support To GNU Binutils With New Instructions - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Znver3-Binutils-Support)
 * [x86 Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)
 * [Introduction to Intel® Advanced Vector Extensions](https://software.intel.com/content/www/us/en/develop/articles/introduction-to-intel-advanced-vector-extensions.html)
 * [AMD64 Architecture Programmer’s Manual, Volume 3: General-Purpose and System Instructions - 24594.pdf](https://www.amd.com/system/files/TechDocs/24594.pdf)
 * [つながる時代のものづくりｘセキュリティを実現する人材が育まれるセキュリティ勉強会（後編） : 富士通](https://www.fujitsu.com/jp/innovation/security/column/10-2/)

{{< /ref >}}
