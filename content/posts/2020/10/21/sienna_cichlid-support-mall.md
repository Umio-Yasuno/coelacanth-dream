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

{{< ins >}}

Phoronix の Michael Larabel 氏によるとディスプレイコントローラーの省電力機能であるとのこと。  
確かによく読んでみると `dcn30_apply_idle_power_optimizations` 関数にコードが追加されている。  
{{< link >}} [Radeon Linux Driver Seeing "MALL" Feature For Big Navi - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Radeon-Big-Navi-MALL-Linux) {{< /link >}}

氏も指摘しているが、DCN3.0 に向けた機能であるのに、サポートするのは *Sienna Cichlid* のみであり、同じく DCN3.0 を搭載する *Navy Flounder* は対象となっていない。  
ただこれもパッチのコメントでそう述べられているだけで、*Sienna Cichlid* のみとする判定文は追加されていないため、実際は *Navy Flounder* でも MALL 機能のサポートが為されていると考えられる。  

{{< /ins >}}

