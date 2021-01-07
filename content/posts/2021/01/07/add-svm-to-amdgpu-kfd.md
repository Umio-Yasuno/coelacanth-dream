---
title: "Linux Kernel に CPU + GPU の統合メモリ空間をサポートする最初のパッチが投稿される"
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

最初のパッチということで、まだ IOMMU が有効だと動作しなかったり、バグや性能への問題があるとしており、正式なサポートがメインラインに組み込まれるまでは時間が掛かると見られる。  

## SVM

*Vega/GFX9* から搭載されている HBCC (High-Bandwidth Cache Controller) は 49-bit アドレッシングをサポートしており、それにより最大 512TB の仮想アドレス空間の確保が可能となっている。  
最新の CPU でも、仮想アドレス空間は 48-bit までというのがほとんどであり、HBCC はそれを十分にカバーできる。  
*GFX8* とそれ以前の AMD GPU では HBCC も無く、49-bit アドレッシングもサポートしていないため、今回のパッチの対象からは外れる。  

 * [[PATCH 06/35] drm/amdkfd: register svm range](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057897.html)

仮想メモリ空間には、複数の GPU も含まれており、GPU間での移行もサポートされているが、ある GPU から別の GPU にアクセスできない場合があるとして、システムメモリをブリッジとして使用して移行する方法を採っている。  
ただコード内のコメントから、移行先と移行元の GPU 両方が *PCIe large bar* {{< comple >}} Above 4G Decoding、最近 Smart Access Memory を切っ掛けに話題となった Resizable PCI BAR のことと思われる {{< /comple >}} を有効にされている場合、または *XGMI / Infinity Fabric Link* で接続され、同じトポロジ内の Hive にある場合は、  
ブリッジにシステムメモリを使うことなく移行するように今後変更されると思われる。  

 >        +	 * TODO: for both devices with PCIe large bar or on same xgmi hive, skip
 >        +	 * system memory as migration bridge
 >        +	 */
 >
 > {{< quote >}} [[PATCH 35/35] drm/amdkfd: multiple gpu migrate vram to vram](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057927.html) {{< /quote >}}

### XNACK

SVM の実装には `XNACK` という機能が活用されている。  
`XNACK` はページフォールトが発生した時の、物理メモリを仮想メモリに割り当てる Demand Paging (デマンドページング) やページの移行を行なう Page Migration に使われる機能。[^xnack]  
APU では *GFX8* 世代から、dGPU では *Vega/GFX9* 世代からサポートされている。  

*Vega/GFX9* でサポートされていた、広帯域な HBM2メモリをキャッシュとし、CPU の持つシステムメモリの一部も VRAM として効率的に扱う、機能としての HBCC にも `XNACK` が使われていた。  

[^xnack]: [llvm-project/AMDGPUUsage.rst at master · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/master/llvm/docs/AMDGPUUsage.rst#target-features)

 * [[PATCH 15/35] drm/amdkfd: add xnack enabled flag to kfd_process](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057908.html)
 * [[PATCH 16/35] drm/amdkfd: add ioctl to configure and query xnack retries](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057907.html)

`XNACK` を有効としてコンパイルされたコードを `XNACK` が無効になっている GPU で実行すると、正しく実行されない、実行されたとしても性能が低下する可能性があることから、コンパイラバックエンドである LLVM 等では、無効時は *Vega10* の GFX ID を *gfx900* 、有効時は *gfx901* として区別していた。  
だがこうした命名規則は途中で廃され、有効時でも *Vega10* は *gfx900* とされるようになった。[^gfxid-xnack]  
途中で廃した (最初はサポートするつもりだった) ことから、HBCC を Linux環境でもサポートすることや、あるいは 49-bitアドレッシングのサポートも合わせ、*Vega10* の頃に今回のパッチのような CPU と GPU の共有メモリを実装する計画があったのかもしれないが、その真実は AMD のみぞ知る。  

[^gfxid-xnack]: [AMDGPU: Remove remnants of gfx901 (it was deprecated some time ago) · llvm/llvm-project@1501af4](https://github.com/llvm/llvm-project/commit/1501af4846791c3b52b812c41ec540081343ba38)


## 3rd Gen AMD Infinity Architecture

AMD は **3rd Gen AMD Infinity Architecture** で CPU と GPU のメモリ空間を統合する構想を明らかにしており、今回のパッチはそれに向けたものと思われる。  
*CDNA 1 アーキテクチャ* を採用する *Arcturus/MI100* は 2nd Gen となり、*CDNA 2 アーキテクチャ* から 3rd Gen AMD Infinity Architecture が実装される。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_6.webp" title="3rd Gen AMD Infinity Architecture" >}}

メモリ空間が統合されることで、メモリ管理やデータの同期等をプログラム側で意識する必要性が減り、より広い GPU の活用が期待される。  
3rd Gen ではメモリ空間の統合だけでなく、CPU と GPU 間の帯域が向上し、レイテンシも削減されるとしている。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_10.webp" title="AMD 3rd Gen Infinity Architecture Enables Accelerated Computing" >}}

[[PATCH 05/35] drm/amdkfd: Add SVM API support capability bits](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057896.html)

{{< ref >}}

 * [Heterogeneous Memory Management (HMM) — The Linux Kernel documentation](https://www.kernel.org/doc/html/v4.18/vm/hmm.html)

{{< /ref >}}
