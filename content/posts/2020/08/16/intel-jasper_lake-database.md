---
title: "Intel Jasper Lake Database"
date: 2020-08-16T07:18:46+09:00
draft: true
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
| Total EU | 16EU /20EU<br>24EU /32EU |
| &emsp;Slice | 1 |
| &emsp;Sub Slice | 2 /4  |
| &emsp;&emsp;EU per Sub Slice | 4 /5 /6 /8 |
| &emsp;GPU L3cache | 1280KB (4 Banks) |
| &emsp;Max Fill-Rate | 8 pixel/clk? |
| TDP | 6 ~ ?W<br>[^dedede-pl1] |

[^intel-family]: [linux/intel-family.h at master · torvalds/linux](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h)
[^dedede-pl1]: <https://review.coreboot.org/c/coreboot/+/41668/6/src/mainboard/google/dedede/variants/baseboard/devicetree.cb>

## Chromebook {#chromebook}

 * Dedede (Main Board)
   * Boten
   * Drawcia
   * Madoo
   * Waddledoo
   * Wheelie
   * Magolor
