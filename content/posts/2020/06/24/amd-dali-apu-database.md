---
title: "AMD Dali APU Database"
date: 2020-06-24T19:52:05+09:00
draft: false
tags: [ "Database", "Dali", "gfx909", "Raven2", "Zen" ]
keywords: [ "", ]
categories: [ "AMD", "APU", "Hardware" ]
noindex: false
---

ベースとなるシリコンが同じ [Raven2](/tags/raven2) であるため、[AMD Pollock APU Database](/posts/2020/06/14/amd-pollock-apu-database/)と内容はいくつか被るが、それ故相互補完する関係にあると言える。  

{{< pindex >}}

 * [Dali 概要](#dali-summay)
 * [Dali 仕様 (推測)](#dali-spec)
 * [Dali 製品](#dali-product)
 * [x86\_Model について ――Picasso と呼ばれる Dali](#dali-x86model)

{{< /pindex >}}

## Dali 概要 {#dali-summay}
コードネーム *Dali* は [Raven2](/tags/raven2) をベースとする省電力APU。  
アーキテクチャと規模は、CPU は Zenアーキテクチャ、2-Core/4-Thread、CPU L3cache 4MB、  
GPU は Vega/GCN5アーキテクチャ [(gfx909)](/tags/gfx909)、総CU数 3基、総RB(RenderBackend)数 1基(4-ROP相当)、GPU L2cache 512KB。  
製造プロセスは Global Foundries 14nm FinFet (14LP)。  

パッケージには現状、*Raven /Picasso* 同様にBGAタイプの *FP5 パッケージ* が使われるものが確認されている。  
対応する TDP は、デフォルトで **AMD 3020e** [^4] の 6W から **Ryzen 3 3250U** [^5] の 15W、cTDP では 25W までと幅広い。  

[^4]: [AMD 3020e | AMD](https://www.amd.com/en/products/apu/amd-3020e#product-specs)
[^5]: [AMD Ryzen™ 3 3250U | AMD](https://www.amd.com/en/products/apu/amd-ryzen-3-3250u#product-specs)

コードネーム *Dali* の略称は *DAL* 。[^1]  
Linux Kernel へのパッチに初めて *Dali* という名前が出てきたのは、2019/09/09 とされる。[^2]  

[^1]: [[PATCH 0/2] Enable Dali for DC](https://lists.freedesktop.org/archives/amd-gfx/2019-September/039775.html)
[^2]: [AMD Announces World’s Highest Performance Desktop and Ultrathin Laptop Processors at CES 2020 | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-announces-worlds-highest-performance-desktop-and-ultrathin)

双子の弟的な存在に、コードネーム [Pollock](/tags/pollock) がいるが、それぞれの i2cコントローラ数が異なり、  
*Dali* は 3基、*Pollock* は 5基持つとされている。[^12]  
他に違いとしては、ターゲットとする TDP帯が考えられ、*Dali* が 6〜15(25)W、*Pollock* が (4.5〜) 4.8W となる。  

*Dali* と *Pollock* は *Raven2* の別リビジョンとなるが、それらを分ける決定的なもの、*Dali* を *Dali* 足らしめるものはまだはっきりとしていない。  

[^12]: [soc/amd/common: Determine # of i2c controllers at runtime (I397b074e) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2057468)

一部製品で `x86_model` が異なり、*Picasso* と同じ `18h` と、*Pollock* と同じ `20h` に分かれる。  

## Dali 仕様 (推測) {#dali-spec}

| AMD Dali | |
| :-- | :--: |
| Socket | FP5 <!-- /AM4 --> |
| Memory Interface | 128-bit (2ch) |
| CPU | *Zen* <br>(Family 17h, Model 18h/20h) |
| &emsp;Max CPU Core/Thread | 2/4 |
| &emsp;CPU L3cache | 4 MB |
| &emsp;CPU Base Clock | (1.2 ~ 2.6 GHz) |
| &emsp;CPU Boost Clock | (2.6 ~ 3.5 GHz) |
| GPU | *Vega (gfx909)* |
| &emsp;Max GPU CU | 3 |
| &emsp;Max GPU SP | 192 |
| &emsp;Max ROP | 4<br>(== 1-RB) |
| &emsp;GPU L2cache | 512 KB |
| &emsp;GPU Clock | 800 ~ 1200 MHz |
| Process | GF 14nm |
| TDP | 6 ~ 25W |

## Dali 製品 {#dali-product}

<!--

 * Athlon Silver 3050U
 * Athlon Gold 3150U
 * Ryzen 3 3250U
 * AMD 3020e 
 * Athlon Silver 3050e 
 * Athlon Silver 3050C
 * Athlon Gold 3150C
 * Ryzen 3 3250C

-->

| Product | Core/Thread | CPU Base/Boost | GPU CU | GPU Clock | TDP |
| :-- | :--: | :--: | :--: | :--: | :--: |
| Athlon Silver 3050U [^t3050u] | 2/2 | 2.3GHz/3.2GHz | 2 | 1.1GHz | 15(12-25)W |
| Athlon Gold 3150U [^t3150u] | 2/4 | 2.4GHz/3.3GHz | 3 | 1.0GHz | 15(12-25)W |
| Ryzen 3 3250U [^t3250u] | 2/4 | 2.6GHz/3.5GHz | 3 | 1.2GHz | 15(12-25)W |
| AMD 3020e [^t3020e] | 2/2 | 1.2GHz/2.6GHz | 3 | 1.0GHz | 6W |
| Athlon Silver 3050e [^t3050e] | 2/4 | (1.4GHz?)/2.8GHz | 3 | 1.0GHz | 6W |
| Athlon Silver 3050C | |
| Athlon Gold 3150C | |
| Ryzen 3 3250C | |
| Ryzen Embedded R1102G [^t1102g] | 2/2 | 1.2GHz/2.6GHz | 3 | 1.0GHz | 6W |
| Ryzen Embedded R1305G [^t1305g] | 2/4 | 1.5GHz/2.8GHz | 3 | 1.0GHz | 8-10W |

[^t3050u]: [AMD Athlon™ Silver 3050U | AMD](https://www.amd.com/en/products/apu/amd-athlon-silver-3050u#product-specs)
[^t3150u]: [AMD Athlon™ Gold 3150U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u#product-specs)
[^t3250u]: [AMD Ryzen™ 3 3250U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u#product-specs)
[^t3050e]: [AMD Athlon™ Silver 3050e | AMD](https://www.amd.com/en/product/9896)
[^t3020e]: [AMD 3020e | AMD](https://www.amd.com/en/products/apu/amd-3020e#product-specs)
[^t1102g]: [AMD Ryzen™ Embedded R1102G with Radeon™ Vega 3 Graphics | AMD](https://www.amd.com/en/product/9226)
[^t1305g]: [AMD Ryzen™ Embedded R1305G with Radeon™ Vega 3 Graphics | AMD](https://www.amd.com/en/product/9221)

## x86\_Model について ――Picasso と呼ばれる Dali {#dali-x86model}

まず、GF 12nmプロセスで製造される Zen+ APU、*Picasso* はプロセッサの判別にも使われる `x86_Model` の値が `18h (24)` となっている。  

 >     switch (boot_cpu_data.x86_model) { 
 >     case 0x1:	/* Zen */
 >     case 0x8:	/* Zen+ */
 >     case 0x11:	/* Zen APU */
 >     case 0x18:	/* Zen+ APU */
 >
 > 引用元: <cite>[linux/k10temp.c at b02c6857389da66b09e447103bdb247ccd182456 · torvalds/linux](https://github.com/torvalds/linux/blob/b02c6857389da66b09e447103bdb247ccd182456/drivers/hwmon/k10temp.c#L587)</cite>

*Raven2* とその別リビジョンである *Dali* 、*Pollock* は当然違う `x86_Model` が与えられるべきなのだが、*Raven2* は *Picasso* と同じ `18h` となり、CPUID も共有する。  
それだけでなく、*Dali* は製品によって `18h` 、または `20h` と分かれており、Chromebook向けでない製品の、またその一部が `18h` となる。[^16]  

 > some non-Chrome OPN Dali APU use the same CPUID with Picasso.
 > to identify those APU via reading the opn spare fuse from smu.
 >
 > 引用元: <cite>[soc/amd/picasso: load RV2 VBIOS with rv2 family OPN (I21f317e1) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2033051)</cite>

*Picasso* とは違う `20h` が本来 *Raven2* に与えられるべき値と考えているのだが、何故そうなったのかは不明。  
*Dali* がすべて `20h` となるならば、それが *Dali* が *Raven2* の別リビジョンとされることの理由としてまだ考えられたのだが。  
*Pollock* はまだ実際の製品が確認できていないが、すべて `20h` となることを願う。  

[^16]: [soc/amd/picasso: load RV2 VBIOS with rv2 family OPN (I21f317e1) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2033051)

`18h` であり、*Picasso* と認識される製品には、以下の 3つが確認されている。  

 * Athlon Silver 3050U [^18h3050u]
 * Athlon Gold 3150U [^18h3150u]
 * Ryzen 3 3250U [^18h3250u]

`20h` となる製品には以下 2つが確認されており、  

 * AMD 3020e [^20h3020e]
 * Athlon Silver 3050e [^20h3050e]

またChromebook向けである以下 3つが `20h` になると考えられる。  

 * Athlon Silver 3050C
 * Athlon Gold 3150C
 * Ryzen 3 3250C


[^18h3050u]: [HP HP All-in-One 22-df0xxx - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2340721)
[^18h3150u]: [HP HP Slim Desktop S01-aF0XXX - Geekbench Browser](https://browser.geekbench.com/v5/cpu/910983)
[^18h3250u]: [HP HP Laptop 15s-eq1xxx - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2445614)

[^20h3020e]: [HP HP Laptop 15-ef1xxx - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15465129)
[^20h3050e]: [HC HCAR357-MI - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15554063)
