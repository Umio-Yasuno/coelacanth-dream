---
title: "Variable Register Thread をサポートする Intel Xe3"
date: 2024-12-22T09:08:57+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Xe3", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel GPU 向けのオープンソースドライバーでは、既に *Xe3 アーキテクチャ* と *Panther Lake* に採用される *Xe3 LPG アーキテクチャ* のサポートが進められている。  
その中で *Xe3 アーキテクチャ* で実装される新機能 "Variable Register Thread" の情報が公開された。  

 * [intel/xe3+: Take advantage of Variable Register Thread on Xe3+. (!32664) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/32664)
 * [Add support for Panther Lake devices · intel/intel-graphics-compiler@f45f621](https://github.com/intel/intel-graphics-compiler/commit/f45f621c4b52705ee14c7e52a7110948ce072b9f)

## VRT {#vrt}
Variable Register Thread (VRT) は *Xe3 アーキテクチャ* でサポートされる、スレッドごとに割り当てるレジスタファイル (GRF, general-purpose register) を柔軟に変更する機能となる。  

従来の Intel GPU アーキテクチャではスレッドあたり 128 GRF が割り当てられ、EU あたり 7スレッドをサポートしていた。  
*Xe-HP アーキテクチャ* からは XVE (EU) あたり 8スレッドに変わり、4スレッドに制限されるがスレッドあたり 256 GRF が割り当てられる Large GRF モードをサポートした。  
レジスタファイルサイズは従来の Intel GPU アーキテクチャでは 28KiB (128 [grf] * 32 [bytes] * 7 [threads])、*Xe-HP アーキテクチャ* からは 32KiB、*Xe-HPC, Xe2 アーキテクチャ* からは 64KiB (128 [grf] * 64 [bytes] * 8 [threads]) となっている。  

*Xe3 アーキテクチャ* では VRT によって、スレッドあたり最大 256 GRF を割り当て可能になると同時に、128 GRF より小さい数を割り当て可能になった。  
VRT のサポートを追加するマージリクエストを投稿した Francisco Jerez 氏は、VRT によってレジスタ溢れ (Register Spills) の削減とほとんどのシェーダーを SIMD32 でコンパイルすることができると語っている。  

レジスタ溢れはコンパイラが割り当てるレジスタファイルがハードウェアがサポートする数より大きい場合に発生し、その場合コンパイラはローカルメモリやキャッシュに一部データを退避させる必要がある。  
それらの退避先はレジスタファイルと比べれば低速であるため、実行性能の低下や SIMD ユニットの使用率低下といった影響が出てしまう。  
VRT のサポートによって、オフラインで実行するシェーダーコンパイラのテストにてレジスタ溢れのサイズを大幅に減らすことができたとしている。  

 * [Small Register Mode vs. Large Register Mode](https://www.intel.com/content/www/us/en/docs/oneapi/optimization-guide-gpu/2023-1/small-register-mode-vs-large-register-mode.html)
 * [Registerization and Avoid Register Spills](https://www.intel.com/content/www/us/en/docs/oneapi/optimization-guide-gpu/2023-0/registerization-and-avoid-register-spills.html)

SIMD32、というより EU 内部の SIMDユニットより広い幅の命令にコンパイルすることの利点は SIMDユニットの使用率を向上させられることにある。  
SIMD 幅によって割り当てるレジスタファイルは増え、実行も内部的に SIMDユニットに合わせて分割されて実行されるが、それは完全にパイプライン化されているため、計算スループットを最大化できる。  

また、Intel Graphics Compiler (IGC) の変更から、*Xe3 アーキテクチャ* では XVE (EU) あたり 10スレッドをサポートするが明らかになっている。これによりレジスタファイルサイズも XVE (EU) あたり 80KiB になると考えられる。  
VRT により、使用するレジスタファイル数が少ないスレッドが複数あってもレジスタの無駄は小さく、多くのスレッドをサポートすることのメリットはより大きくなる。  

 * [Add support for Panther Lake devices · intel/intel-graphics-compiler@f45f621](https://github.com/intel/intel-graphics-compiler/commit/f45f621c4b52705ee14c7e52a7110948ce072b9f)
