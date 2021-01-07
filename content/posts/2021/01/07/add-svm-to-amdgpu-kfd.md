---
title: "Linux Kernel に CPU + GPU の共有メモリをサポートするパッチが投稿される"
date: 2021-01-07T17:51:16+09:00
draft: true
tags: [ "Arcturus", ]
# keywords: [ "", ]
categories: [ "AMD", "CPU", "GPU", "Software" ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel (amd-gfx) に、HMM (Heterogeneous Memory Management) をベースとする SVM (Share Virtual Memory) のサポートを追加する最初のパッチが投稿された。  

 * [[PATCH 00/35] Add HMM-based SVM memory manager to KFD](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057892.html)

*Vega/GFX9* から搭載されている HBCC (High-Bandwidth Cache Controller) は 49-bit アドレッシングをサポートしており、それにより最大 512TB の仮想アドレス空間の確保が可能となっている。  
最新の CPU でも、仮想アドレス空間は 48-bit までというのがほとんどであり、HBCC はそれを十分にカバーできる。  
*GFX8* とそれ以前の AMD GPU では HBCC も 49-bit アドレッシングもサポートしていないため、今回のパッチの対象からは外れる。  

[[PATCH 06/35] drm/amdkfd: register svm range](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057897.html)

AMD は **3rd Gen AMD Infinity Architecture** で CPU と GPU のメモリ空間を統合する構想を明らかにしており、今回のパッチはそれに向けたものと思われる。  
*CDNA 1 アーキテクチャ* を採用する *Arcturus/MI100* は **2nd Gen**

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_6.webp" title="3rd Gen AMD Infinity Architecture" >}}

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_10.webp" title="AMD 3rd Gen Infinity Architecture Enables Accelerated Computing" >}}

[[PATCH 05/35] drm/amdkfd: Add SVM API support capability bits](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057896.html)

仮想メモリ空間には、複数の GPU も含まれており、GPU間での移行もサポートされているが、ある GPU から別の GPU にアクセスできない場合があるとして、システムメモリをブリッジとして使用して移行する方法を採っている。  
ただコード内のコメントから、移行先と移行元の GPU 両方が *PCIe large bar* {{< comple >}} Above 4G Decoding、最近では Smart Access Memory と呼ばれるようになった Resizable PCI BAR のことと思われる {{< /comple >}} を有効にされている場合、または *XGMI/Infinity Fabric Link*

 >        +	 * TODO: for both devices with PCIe large bar or on same xgmi hive, skip
 >        +	 * system memory as migration bridge
 >        +	 */

[[PATCH 35/35] drm/amdkfd: multiple gpu migrate vram to vram](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057927.html)

{{< ref >}}

 * [Heterogeneous Memory Management (HMM) — The Linux Kernel documentation](https://www.kernel.org/doc/html/v4.18/vm/hmm.html)

{{< /ref >}}
