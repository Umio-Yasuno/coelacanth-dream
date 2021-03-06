---
title: "AMD Pollock APU データベース"
date: 2020-06-14T00:07:27+09:00
draft: false
tags: [ "Raven2", "Pollock", "gfx909", "Zen" ]
keywords: [ "", ]
categories: [ "Hardware", "APU", "AMD", "Database" ]
noindex: false
# weight: 100
---

あるハードウェアに関する情報を集め続ける内に、情報それぞれが孤立気味になってきたため、随時更新を行なう情報が集約されたページを作ることとした。  
そのため、作成時に限って目新しい情報はないことを留意していただきたい。  

今後他のハードウェアに関するページも作っていくつもり。  

{{< pindex >}}

 * [Pollock 概要](#plk-summary)
 * [Pollock 仕様(推測)](#plk-spec)
 * [FT5 パッケージ](#ft5-pack)
 * [ボード](#board)

{{< /pindex >}}

## Pollock 概要 {#plk-summary}
コードネーム *Pollock* は[Raven2](/tags/raven2)をベースとする省電力APU。  
アーキテクチャと規模は、CPU は Zenアーキテクチャ、2-Core/4-Thread、CPU L3cache 4MB、  
GPU は Vega/GCN5アーキテクチャ [(gfx909)](/tags/gfx909)、総CU数 3基、総RB(RenderBackend)数 1基(4-ROP相当)、GPU L2cache 512KB。  
製造プロセスは Global Foundries 14nm FinFet (14LP)。  

パッケージには現状、BGAタイプの *FT5 パッケージ* が使われることが確認されている。[^7]  
TDP は *FT5 パッケージ* で 4.8W。[^1]  
ブースト時の電力リミットの仕様は、デフォルトで 4.8W、ピーク時で 9W、ピークからデフォルト移行時で 6W。[^1]  

[^7]: [soc/amd/picasso: Add Kconfig option for chip footprint (Ia4663d38) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051509)
[^1]: [mb/google/zork: update power parameters to 4.8w for dalboz (I711d1109) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2135098)

コードネーム *Pollock* の略称は *PLK* 。[^5][^6]  
Linux Kernel へのパッチに初めて *Pollock* という名が出てきたのは、2020/01/08。[^13]

[^13]: [[PATCH] drm/amd/display: add Pollock IDs, fix Pollock & Dali clk mgr construct](https://lists.freedesktop.org/archives/amd-gfx/2020-January/044548.html)

[^5]: [Jun4 Benchmarks [2006047-AR-JUN47027921] - OpenBenchmarking.org](https://openbenchmarking.org/result/2006047-AR-JUN47027921)
[^6]: [Details for Computer/Device AMD Cereme-PLK Pollock : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_system.php?q=cea598ab93a493a79eb8dfe2c4b68badc4f9dfb78aacd4e9cfaacff2c2e497aa92&l=en)

双子の兄的な存在に、コードネーム [Dali](/tags/dali) がいるが、それぞれの i2cコントローラ数が異なり、  
*Dali* は 3基、*Pollock* は 5基持つとされている。[^12]  
また、*Dali* は *FP5 /AM4 パッケージ* を採用し、*Pollock* は *FT5 パッケージ* を採用したものと分けられ、そこで *Dali* と *Pollock* とを区別することができる。  

[^12]: [soc/amd/common: Determine # of i2c controllers at runtime (I397b074e) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2057468)

*Dali* と *Pollock* は *Raven2* の別リビジョンとなるが、それらを分ける決定的なもの、*Dali* を *Dali* 足らしめるものはまだはっきりとしていない。  
ただ、*Dali, Pollock* が登場してからは *Raven2* GPU に新たな DeviceID が割り振られることはなく、*Raven2* が元いた位置は完全に *Dali* へ引き継がれたものと認識している。  

`x86_model` は `20h` となる。  

## Pollock 仕様 (推測) {#plk-spec}

| AMD Pollock | |
| :-- | :--: |
| Socket | FT5 |
| Memory Interface | 64-bit (1ch) |
| CPU | *Zen* <br>(Family 17h, Model 20h) |
| &emsp;Max CPU Core/Thread | 2/4 |
| &emsp;CPU L3cache | 4 MB |
| &emsp;CPU Base Clock | (1.0 ~ 1.2 GHz) |
| &emsp;CPU Boost Clock | 2.3-(2.35 ~ 2.4 GHz) |
| GPU | *Vega (gfx909)* |
| &emsp;Max GPU CU | 3 |
| &emsp;Max GPU SP | 192 |
| &emsp;Max RB+ | 1<br>(== 8-ROP) |
| &emsp;GPU L2cache | 128 KB |
| &emsp;GPU Clock | 600-800 MHz<br>[^6] |
| Process | GF 14nm |
| TDP | 4.8-6W |

CPUのクロック仕様は主に Geekbenchの結果を元にしている。  
末尾に `.gb5` を付けることで詳細を見ることができ、そこで大体の最大クロックがわかる。  
{{< link >}}[Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2301114){{< /link >}}
{{< link >}}<https://browser.geekbench.com/v5/cpu/2301114.gb5>{{< /link >}}


## FT5 パッケージ {#ft5-pack}
*FT5 パッケージ* は 2-in-1の小型フォームファクター向けで、電力効率を重視したクラスのパッケージとなる。[^11]  
特徴的な仕様として、*Pollock* のベースとなる *Raven2* は DDR4 デュアルチャネル分のインターフェイスを備えているが、*FT5 パッケージ* はそれを制限し、DDR4 シングルチャネルまでの対応となる点があげられる。[^9]  
初期のパッチセットでは SATAインターフェイスも無いという記述があったが[^10]、後のパッチセットでその部分は削除されている。[^9]  

[^9]: [soc/amd/picasso: Add Kconfig option for chip footprint (Ia4663d38) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051509)
[^10]: <https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051509/1>
[^11]: [Tech Docs | AMD](https://www.amd.com/en/support/tech-docs)

**Dalboz** の仕様から、NVMe SSD の PCIe Gen3 x2 接続には対応しているものと思われる。[^8]  

[^8]: [Dalboz: tuning PCIe topology make DUT can boot up from SSD (I9113b54d) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2066389)

## ボード {#board}
AMD が提供する *Pollock* 搭載のリファレンスボードのコードネームは **Cereme** 。[^2]  
*Pollock* を搭載する Chromebook ボードは 2020/06/14 現在、**Dalboz** [^3] とそれをベースとする **Vilboz** [^4] の 2種類が確認されている。  

[^2]: [mainboard/amd/cereme : Create Initial Cereme Platform (I593d661f) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2024459)
[^3]: [mb/google/zork: add dalboz baseboard option (I646d9ad1) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2055651)
[^4]: [vilboz: Initial EC image (I511548fb) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2224680)

## Pollock SKU {#plk-sku}

| Pollock SKU | Core/Thread | CPU Base/Boost | GPU CU | GPU Clock | TDP |
| :-- | :--: | :--: | :--: | :--: | :--: |
| AMD 3015e[^amd-3015e] | 2/4 | 1.2GHz/2.3GHz | 3 | 0.6GHz | 6W |
| AMD 3015Ce[^amd-3015ce-gb5] | 2/4 | 1.2GHz/2.3GHz | 3 | 0.6GHz | |

[^amd-3015e]: [AMD 3015e | AMD](https://www.amd.com/en/products/apu/amd-3015e#product-specs)
[^amd-3015ce-gb5]: [Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/3664531)
