---
title: "Sienna Cichlid は MALL 機能をサポート"
date: 2020-10-21T05:54:59+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", GPU ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel (amd-gfx) に *RDNA 2 /GFX10.3* 世代の GPU、*Sienna Cichlid* が持つ MALL 機能を有効化するパッチが投稿された。  
{{< link >}} [[PATCH 2/3] drm/amdgpu: add support to configure MALL for sienna_cichlid (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/055006.html) {{< /link >}}
{{< link >}} [[PATCH 3/3] drm/amdgpu/display: add MALL support](https://lists.freedesktop.org/archives/amd-gfx/2020-October/055007.html) {{< /link >}}

## 詳細は不明な MALL 機能
MALL は *Memory Access at Last Level* の略とされているが、その機能の詳細はパッチに記述されていない。  
パッチの中身を読んでも、NOALLOC を示すレジスタの値の追加や PTE (Page Table Entry) へのビットの追加、ディスプレイコントローラーが MALL 機能を使えるようにしてあることぐらいしか {{< comple >}} 自分には {{< /comple >}} 分からない。  

推測、というよりは妄想に近いことを前置きすると、まず *at Last Level* とのキーワードから通常の GPU が持つ VRAM より下の階層と考える。  
そして MALL 機能は現状 *Sienna Cichlid* だけがサポートするとされており、*RDNA 2 /GFX10.3* 世代の dGPU 全てが持つという書き方はされていない。  
*Sienna Cichlid* だけがサポートする機能では他に、高速な GPU間通信 *XGMI /Infinity Fabric* が思い当たる。[^xgmi]  
結果、MALL 機能はマルチGPUを意識した機能であり、マルチGPU構成において分散するメモリに関する機能という考えが浮かんだが、あくまでも妄想であるし、妄想にしてもパッとしない。  

[^xgmi]: [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/)

後は今後 AMD がその内明かしてくれるだろう情報を待つ。  
