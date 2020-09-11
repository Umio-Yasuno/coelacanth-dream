---
title: "Intel Jasper Lake Database"
date: 2020-08-16T07:18:46+09:00
draft: false
tags: [ "Database", "Tremont", "Jasper_Lake", "Gen11" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

[Intel Tremont](/tags/tremont)アーキテクチャを採用するモバイル向けプロセッサ。[^jsl-for-mobile]  

[^jsl-for-mobile]: [x86/cpu: Add Jasper Lake to Intel family · torvalds/linux@b2d32af](https://github.com/torvalds/linux/commit/b2d32af0bff402b4c1fce28311759dd1f6af058a)

## Jasper Lake Spec {#spec}
| Intel JSL | |
| :-- | :--: |
| Memory Interface | LP/DDR4 128-bit? |
| CPU | *Tremont*<br>(Family: 0x6, Model 0x9C)<br>[^intel-family] |
| &emsp;CPU Core/Thread | (2/4?)<br>(4/4?) |
| GPU | *Gen11* |
| Total EU<br>(Sub-Slice x EU) | 16EU(2x8) /20EU(4x5)<br>24EU(4x6) /32EU(4x8) |
| &emsp;GPU L3cache | 1280KB (4 Banks) |
| TDP | 6 ~ ?W<br>[^dedede-pl1] |

[^intel-family]: [linux/intel-family.h at master · torvalds/linux](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h)
[^dedede-pl1]: <https://review.coreboot.org/c/coreboot/+/41668/6/src/mainboard/google/dedede/variants/baseboard/devicetree.cb>

## CPU {#jsl-cpu}
CPUアーキテクチャに [Tremont](/tags/tremont) を採用することは判明しているが、最大コア数、CPUコア間で共有する L2キャッシュ構成等は不明。  

<details>
<summary>Tremont Architecture</summary>
   {{< figure src="/image/2020/09/11/tremont-arch-diagram.webp" caption="画像出典: [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)" >}}
</details>

### GPU {#jsl-gpu}
GPU の仕様は、同様に *Tremontアーキテクチャ* を採用する [Elkhart Lake](/tags/elkhart_lake) とほとんど共通している。そのため、GPUドライバーの記述では *Elkhart Lake* とされている場合もある。  
規模やディスプレイ周りの仕様は多少異なるとされる。  

GPUアーキテクチャは [Gen11](/tags/gen11)。[^jsl-gen11]  
*Ice Lake* と同じだが、[intel/media-driver](https://github.com/intel/media-driver)の記述を読む限り、ハードウェアの固定機能部に合わせてシェーダーを用いるエンコードや映像の後処理(フィルター、デノイズ) 機能が *Ice Lake* とは違いサポートされていない。  
だがソフトウェアの将来的なアップデートでサポートされる可能性はある。  

[^jsl-gen11]: [intel: Add device info for 1x4x6 Jasper Lake (11fdd5f5) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/11fdd5f52c3db070f33f7ef82d41acf14b1a2670)

GPUのシェーダー部、Slice/Sub-Slice/EU の構成には 5種類が存在する。EU数で言えば、16EU/20EU/24EU/32EU の 4種類となる。  

 >      CHIPSET(0x4E51, ehl_4x4, "JSL", "Intel(R) UHD Graphics")
 >      CHIPSET(0x4E55, ehl_2x8, "JSL", "Intel(R) UHD Graphics")
 >      CHIPSET(0x4E57, ehl_4x5, "JSL", "Intel(R) UHD Graphics")
 >      CHIPSET(0x4E61, ehl_4x6, "JSL", "Intel(R) UHD Graphics")
 >      CHIPSET(0x4E71, ehl_4x8, "JSL", "Intel(R) UHD Graphics")
 >
 > 引用元: [include/pci_ids/i965_pci_ids.h · 559b26b7ee093c2cbe446bf8023876642deadcee · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/559b26b7ee093c2cbe446bf8023876642deadcee/include/pci_ids/i965_pci_ids.h)

Slice数は 1基で固定、Sub-Slice数は 2基または 4基、Sub-Slice あたりの EU数は 4/5/6/8基のどれかとなっている。  
総 16EUとなる構成には、Sub-Slice数 2基、それぞれの EU数は 8基という構成と、Sub-Slice数 4基、それぞれの EU数は 4基という構成の 2種類がある。  
Sub-Slice 内には、テクスチャサンプラーやローカルメモリ等のリソースが含まれるため、性能では後者の Sub-Slice数 4基の構成が高くなるのではないかと考えられる。  

## Chromebook {#chromebook}

*Jasper Lake* を搭載するChromebookボードは、メインボードがコードネーム **Dedede** であり、実際に製品に採用される派生ボードにも星のカービィに関連したコードネームが付けられている。  

 * Dedede (Main Board)
   * Boten
   * Drawcia
   * Madoo
   * Waddledoo
   * Wheelie
   * Magolor


{{< ref >}}

 * [introducing-intel-tremont-microarchiture.pdf](https://newsroom.intel.com/wp-content/uploads/sites/11/2019/10/introducing-intel-tremont-microarchiture.pdf)

{{< /ref >}}
