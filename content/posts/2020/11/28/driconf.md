---
title: "Mesa3Dドライバーの DriConf について"
date: 2020-11-28T04:03:52+09:00
draft: false
tags: [ "RadeonSI", ]
# keywords: [ "", ]
categories: [ "Software", "GPU" ]
noindex: false
# summary: ""
toc: false
---

DriConf は OpenGL に実装された、性能向上や描画品質の変更を目的としたオプションであり、ドライバー/スクリーン/アプリ―ケーションごとに設定できる。  
OpenGL API を用いるゲーム/アプリケーションで何らかの問題があった場合、その回避策がオプションとして実装、そのアプリケーションに対してはデフォルトで有効とすることがある。性能が明らかに向上する場合でもデフォルトで有効とされることも。  

デフォルトの設定がどうなっているかは、以下のファイルから確認できる。  

 * [src/util/00-mesa-defaults.conf · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/master/src/util/00-mesa-defaults.conf) 

可変リフレッシュレート (Adaptive Sync) 有効無効の設定もドライバー/スクリーン/アプリケーションごとに設定が可能。  

以前はオプションの切り替えに使うツールとして [DriConf](https://dri.freedesktop.org/wiki/DriConf/) が標準だったが、今は [adriconf](https://gitlab.freedesktop.org/mesa/adriconf) がそのポジションにいる。  

AMD GPU向けに実装されているオプションには以下のようなものがある。  
また、特に性能に影響するものとしてディスパッチをマルチスレッド化する `mesa_glthread` オプションがある。  
これらを試しに有効化し、性能をチューニングすることはユーザーとして 1つの楽しみとも言える。  


 >        #define DRI_CONF_ALLOW_DRAW_OUT_OF_ORDER(def) \
 >           DRI_CONF_OPT_B(allow_draw_out_of_order, def, \
 >                          "Allow out-of-order draw optimizations. Set when Z fighting doesn't have to be accurate.")
 > {{< quote >}} [src/util/driconf.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/master/src/util/driconf.h) {{< /quote >}}

 >        /**
 >         * \brief radeonsi specific configuration options
 >         */
 >        
 >        #define DRI_CONF_RADEONSI_ASSUME_NO_Z_FIGHTS(def) \
 >           DRI_CONF_OPT_B(radeonsi_assume_no_z_fights, def, \
 >                          "Assume no Z fights (enables aggressive out-of-order rasterization to improve performance; may cause rendering errors)")
 >        
 >        #define DRI_CONF_RADEONSI_COMMUTATIVE_BLEND_ADD(def) \
 >           DRI_CONF_OPT_B(radeonsi_commutative_blend_add, def, \
 >                          "Commutative additive blending optimizations (may cause rendering errors)")
 >        
 >        #define DRI_CONF_RADEONSI_ZERO_ALL_VRAM_ALLOCS(def) \
 >           DRI_CONF_OPT_B(radeonsi_zerovram, def, \
 >                          "Zero all vram allocations")
 >
 > {{< quote >}} [src/util/driconf.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/master/src/util/driconf.h) {{< /quote >}}
