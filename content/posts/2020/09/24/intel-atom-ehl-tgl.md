---
title: "Intel Elkhart Lake 発表、その詳細"
date: 2020-09-24T04:20:23+09:00
draft: false
tags: [ "Tremont", "Elkhart_Lake" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

Intel より、組み込み向けに *Tiger Lake* をベースとする Coreプロセッサと、*Elkhart Lake* ベースの Atomプロセッサが発表された。  

 * [IoT-Enhanced Processors Increase Performance, AI, Security | Intel Newsroom](https://newsroom.intel.com/news/iot-processors-industrial-edge/)
 * [Elkhart Lake: Overview and Technical Documentation](https://www.intel.com/content/www/us/en/design/products-and-solutions/processors-and-chipsets/elkhart-lake/overview.html)

## Elkhart Lake 詳細 {#ehl-detail}

*Tremontアーキテクチャ* では 4コア + L2キャッシュ でクラスタを形成し、単位としている。  
L2キャッシュサイズは 1MB から 4.5MB の範囲で選択可能であり、マイクロサーバー向けの *Snow Ridge* は 4.5MB、モバイル向けのハイブリッドプロセッサ *Lakefield* は 1.5MB、そして組み込み向けの *Elkhart Lake* も 1.5MB となっている。  
他にモバイル向けの *Jasper Lake* がいるが、L2キャッシュサイズを含めて詳細はまだ不明。  
*Snow Ridge* は 6クラスタ、計 24コアの構成だが、*Lakefield / Elkhart Lake* は 1クラスタ、計 4コアとされる。  

| Intel Tremont | Snow Ridge | Lakefield | Elkhart Lake | Jasper Lake |
| :-- | :--: | :--: | :--: | :--: |
| Max CPU Clock | 2.2 GHz | ? | 3.0 GHz | ? |
| Cluster LLC | 4.5 MB | 1.5 MB | 1.5 MB | ? |
| GPU EU | N/A | 64 EU | 32 EU | 32 EU |


パッケージは複数のチップで構成される MCP(Multi-Chip Package)。CPU、GPU、メモリインターフェイス等が統合されている Compute Die は 10nmプロセス、主に I/O 部を担当している PCH(Platform Control Hub) は 14nmプロセスで製造される。  

*Elkhart Lake* は組み込み向けとして I/O を多く搭載する必要があり、パッケージレベルで統合される PCH を用意したと思われるが、Linux Kenel のドキュメントからモバイル向けの *Jasper Lake* は PCH を必要としない SoC だと思われる。[^jsl-soc]  

[^jsl-soc]: [Kernel driver i2c-i801 — The Linux Kernel documentation](https://01.org/linuxgraphics/gfx-docs/drm/i2c/busses/i2c-i801.html)

*Elkhart Lake* はメモリインターフェイスを 128-bit 分持ち、最大で LPDDR4/x-4267(4x 32-bit) または DDR4-3200(2x 64-bit) に対応しているが、今回発表されたモデルとプラットフォームでは LPDDR4/x-3200 までとなっている。  

Hyper-Threading は有効にされておらず、2コアのSKUであっても 2C/2T とされている。  
*Jasper Lake* でも同様に無効にされたままなのか、有効にされるのかはやはり不明。  

TDP(PL1) は 4.5W-12W の範囲に対応し、Burst(Turbo) Mode の最大クロックは共通して 3.0GHz。  
GPU EU数は 16EU または 32EU となり、最大クロックは 850MHz。  
