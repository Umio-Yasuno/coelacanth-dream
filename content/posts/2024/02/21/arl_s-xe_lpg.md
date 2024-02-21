---
title: "Arrow Lake-S は Xe-LPG, Arrow Lake-H は Xe-LPG Plus に"
date: 2024-02-21T17:34:29+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU"]
tags: [ "Arrow_Lake", "Xe-LPG"]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

以前に Intel Arrow Lake では GPU アーキテクチャが *Xe-LPG Plus* となり、行列積和演算命令 `DPAS (Dot Product Accumulate Systolic)` 命令をサポートすることについて取り上げたが、  
これは Arrow Lake-H に限定され、Arrow Lake-S は Meteor Lake と同じ *Xe-LPG* となる可能性が出てきた。  

 * [GPU が Xe-LPG Plus となり DPAS 命令をサポートする Intel Arrow Lake | Coelacanth's Dream](/posts/2023/11/24/intel-arl-xe-lpg-plus/)

関連する変更は [VC Intrinsics](https://github.com/intel/vc-intrinsics) と [Intel(R) C for Metal Compiler](https://github.com/intel/cm-compiler/blob/cmc_monorepo_110/clang/Readme.md) に見られる。  
VC Intrinsics は Intel GPU 向けの LLVM IR 組み込み関数を使用するためのライブラリ、Intel(R) C for Metal Compiler (cm-compiler) は GPU カーネルのプログラミング言語、C for Metal の Intel GPU 向けコンパイラとなる。  

 * [Enable Arrow Lake and Lunar Lake platforms support · intel/cm-compiler@4147c5d](https://github.com/intel/cm-compiler/commit/4147c5d2b715f9aec26a4674b415ffc35895d652)

 >         +    if (Product == IGFX_ARROWLAKE) {
 >         +      if (GFX_IS_ARL_S(DevId))
 >         +        return {"XeLPG", RevId};
 >         +      return {"XeLPGPlus", RevId};
 >         +    }
 >              break;
 > 
 > {{< quote >}} [Add VC support for Arrow Lake and Lunar Lake platforms · intel/intel-graphics-compiler@e3333c9](https://github.com/intel/intel-graphics-compiler/commit/e3333c9e81383996d9d2cf72e16128cfeb173bf6) {{< /quote >}}
 >
 >         +  case IGFX_ARROWLAKE:
 >         +    if (GFX_IS_ARL_S(DeviceId))
 >         +      return "arl-s";
 >         +    return "arl-h";
 >         +  case IGFX_LUNARLAKE:
 >         +    return "lnl";
 >
 > {{< quote >}} [Add VC support for Arrow Lake and Lunar Lake platforms · intel/intel-graphics-compiler@e3333c9](https://github.com/intel/intel-graphics-compiler/commit/e3333c9e81383996d9d2cf72e16128cfeb173bf6) {{< /quote >}}

Arrow Lake の GPU が *Xe-LPG* と *Xe-LPG Plus* に分かれることについては Intel GPU 向けの Kernel Mode Driver (KMD) と User Mode Driver (UMD) でも触れられており、やはり UMD では `DPAS` 命令を Arrow Lake-H (*Xe-LPG Plus*) の場合のみ発行するようになっている。  

 * <https://patchwork.freedesktop.org/series/128322/>
 * [intel/compiler: Lower DPAS instructions on ARL except ARL-H (c3a0483f) · コミット · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/c3a0483f5bcb01aa74946069415618ebcf897cb3?merge_request_iid=27352)

 >               Some SKUs of Arrow Lake use a slightly newer Xe_LPG+ graphics
 >         IP (version 12.74).  Add some additional PCI IDs and extend the
 >         code to support this newer IP version.  The general code flow
 >         should continue to match existing MTL and Xe_LPG code paths.
 >
 > {{< quote >}} <https://patchwork.freedesktop.org/series/128322/> {{< /quote >}}

Arrow Lake については GPU 部だけでなく、CPU 部においてもサポートする拡張命令が `arrowlake (06_C5H)` と `arrowlake-s (06_C6H)` で異なることが既にドキュメントで公開されており、`arrowlake-s` は `arrowlake` のサポート範囲に加えて `AVXVNNIINT16, SHA512, SM3, SM4` といった命令をサポートしている。  
ここでの接尾辞 `-S` が CPU 部と GPU 部で同じ意味だとすると、規模だけでなくアーキテクチャレベルで Arrow Lake-S は CPU 重視、Arrow Lake-H は GPU 重視と言える傾向となっているのかもしれない。  
