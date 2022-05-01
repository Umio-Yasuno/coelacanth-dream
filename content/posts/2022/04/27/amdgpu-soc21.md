---
title: "AMDGPUドライバーに SoC21 をサポートするパッチ"
date: 2022-04-27T14:56:49+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "Linux_Kernel", "GFX11" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD のソフトウェアエンジニアである Alex Deucher 氏により、AMDGPUドライバーに *SoC21* のサポートを追加するパッチが amd-gfx メーリングリストに投稿されている。  

 * [[PATCH 0/5] Add new SoC21 infrastructure](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078231.html)
    * [[PATCH 3/5] drm/amdgpu: add nbio callback to query rom offset](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078232.html)
    * [[PATCH 4/5] drm/amdgpu: add new write field for soc21](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078233.html)
    * [[PATCH 5/5] drm/amdgpu: add soc21 common ip block v2](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078234.html)

*SoC21* は GPU SoC におけるインフラストラクチャであり、各種 IPブロックのレジスタ設定、初期化、状態や省電力機能の管理といった、まさにベースとなる機能を担っている。  
*Vega/GFX9* 世代とそれ以降の AMD GPU は *SoC15* を採用している。  

パッチのサイズが大きすぎるとの理由でメーリングリストには投稿されていないが、`GC (Graphics and Compute) v11.0.0` IPブロックのレジスタヘッダーを追加するパッチもスレッドには含まれている。  
*RDNA 2* 世代の `GC v10.3.x` よりバージョンが進んでいることから、次世代の AMD APU/GPU、*RDNA 3* 世代をターゲットにしたパッチではないかと思われる。  

 > 		This adds GPU SoC infrastructure for asics which
 > 		use the soc21 design.  The first two patches are
 > 		register headers which are too big for the mailing
 > 		list so I have omitted them.
 > 		
 > 		Hawking Zhang (3):
 > 		  drm/amdgpu: add mp v13_0_0 ip headers v7
 > 		  drm/amdgpu: add gc v11_0_0 ip headers v11
 > 		  drm/amdgpu: add nbio callback to query rom offset
 > 		
 > 		Likun Gao (1):
 > 		  drm/amdgpu: add new write field for soc21
 > 		
 > 		Stanley.Yang (1):
 > 		  drm/amdgpu: add soc21 common ip block v2
 >
 > {{< quote >}} [[PATCH 0/5] Add new SoC21 infrastructure](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078231.html) {{< /quote >}}

AMDGPUドライバーのレジスタヘッダーには、サイズが大きなものだと約 5-10MB あり、同じく AMD のソフトウェアエンジニアである Christian König 氏はそれらをうまく圧縮する方法を見つけなければならないとしている。[^register-header]  

[^register-header]: [[PATCH 0/5] Add new SoC21 infrastructure](https://lists.freedesktop.org/archives/amd-gfx/2022-April/078255.html)
