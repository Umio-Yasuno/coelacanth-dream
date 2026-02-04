---
title: "Nova Lake-P は GPU に Xe3p LPG を搭載"
date: 2026-02-04T20:14:33+09:00
draft: false
categories: [ "GPU", "Intel", "Hardware" ]
tags: [ "Nova_Lake", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

以前に、Intel の次世代プロセッサ *Nova Lake* の GPU コアアーキテクチャは *Panther Lake* や *Wildcat Lake* と同じ *Xe3 (Xe3_LPG)* だとする記事を公開した。  

 * [Nova Lake の GPU コアアーキテクチャは Xe3 | Coelacanth's Dream](/posts/2025/11/20/nova-lake-xe3/)

だが、先日公開された Linux Kernel における Intel GPU ドライバーへのパッチにおいて、*Nova Lake-P* の存在とその GPU コアアーキテクチャが *Xe3p (Xe3p_LPG)* であることが明らかにされた。  

 * [[PATCH 00/16] Basic enabling patches for Xe3p_LPG and NVL-P](https://lore.kernel.org/intel-gfx/20260202221104.GI458797@mdroper-desk1.amr.corp.intel.com/T/#m5bebb5831c5b3065e9a69091c66041cac99e8075)

Intel プロセッサにおける "P" バリアントは主にモバイルプラットフォーム向けとされ、"U" バリアントより上位の性能を持つプロセッサに位置付けられる。  

*Nova Lake-P* は GPU コア部は *Xe3p_LPG* だが、メディアエンジン部とディスプレイエンジン部は *Xe3p_LPM* と *Xe3p_LPD* を採用し、*Nova Lake-S (NVL-S/HX/UL/U/H)* と同じとなっている。  

 >         NVL-P is a new Intel platform that comes with the following IPs:
 >         
 >         - Xe3p_LPG graphics;
 >         - Xe3p_LPM media;
 >         - Xe3p_LPD display.
 >         
 >         Enabling patches for Xe3p_LPM and Xe3p_LPD are already integrated in our
 >         driver.  In this series we add patches enabling Xe3p_LPG and then follow
 >         up with patches enabling NVL-P as a platform in our driver.
 >
 > {{< quote >}} [[PATCH 00/16] Basic enabling patches for Xe3p_LPG and NVL-P](https://lore.kernel.org/intel-gfx/20260202221104.GI458797@mdroper-desk1.amr.corp.intel.com/T/#m5bebb5831c5b3065e9a69091c66041cac99e8075) {{< /quote >}}

*Xe3p_LPG* の詳細はまだ不明だが、Crescent Island (*Xe3p_XPC*) と同様に、ページテーブルの再利用に関するハードウェア支援機能をサポートすることは明かされている。  

*Xe3p* では新たに FP8/FP4/MXFP4/MXFP8 に関する行列積和演算命令が追加される。  
そのため、*Nova Lake-S* と *Nova Lake-P* では推論性能や AI 関連の機能において大きな違いが生まれることが考えられる。  
*Xe_LPG (Meteor Lake/Arrow Lake-S)* と *Xe_LPG+ (Arrow Lake-H)* においても、XMX (Xe Matrix eXtention) の有無という違いがあった。  
同じような世代とコードネームでも、GPU 性能と機能の差別化のためか、実質的な GPU コアアーキテクチャを変えるという傾向が見られる。  

さらに言えば、同じ *Xe3* でも、*Panther Lake, Nova Lake-U/H* はハードウェアレイトレーシングをサポートするが、*Wildcat Lake, Nova Lake-S/HX/UL* はサポートしないという違いが存在する。  
プラットフォームごとの最適化が重要なのはわかるが、結果としてユーザーが GPU の機能について把握しにくくなっている気がしなくもない。  

 * [src/intel/dev/intel_device_info.c · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/intel/dev/intel_device_info.c?ref_type=heads)
