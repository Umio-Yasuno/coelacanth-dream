---
title: "Intel Arc Aシリーズ/ATS-M 個人的まとめ"
date: 2022-03-31T03:11:42+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Xe-HPG", "DG2" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel より、Intel Arc Aシリーズ GPU とその詳細が正式発表された。  
その個人的なまとめ。  

* [Intel’s Discrete Mobile Graphics Family Arrives](https://www.intel.com/content/www/us/en/newsroom/opinion/intel-discrete-mobile-graphics-family-arrives.html)
    * [Intel Arc Graphics](https://www.intel.com/content/www/us/en/newsroom/resources/intel-arc-graphics.html)
    * [Intel-Arc-Keynote-Presentation-Slides.pdf](https://download.intel.com/newsroom/2022/client-computing/Intel-Arc-Keynote-Presentation-Slides.pdf)
* [Intel、ノートPC向け単体GPU「Arc Aシリーズ」を正式発表。ローエンドからハイエンドまで4月から順次投入 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1399221.html)
* [Intel® Iris® Xe GPU Architecture](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-gpu-optimization-guide/top/xe-arch.html)

{{< pindex >}}
 * [ACM-G10 (DG2-G10), ACM-G11 (DG2-G11)](#dg2-g10-g11)
    * [ACM-G12 (DG2-G12)](#acm-g12)
    * [メディアエンジン](#media)
 * [ATS-M](#ats-m)
{{< /pindex >}}

## ACM-G10 (DG2-G10), ACM-G11 (DG2-G11) {#dg2-g10-g11}
一般に公開されているリリース、資料には記述されていないが、メディア向けの資料には Intel Arc Aシリーズのベースとなる 2種類のダイの詳細がある。  

* [Intel、ノートPC向け単体GPU「Arc Aシリーズ」を正式発表。ローエンドからハイエンドまで4月から順次投入 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1399221.html)

*ACM-G1x* という名称については、[IGC (intel-graphics-compiler)](https://github.com/intel/intel-graphics-compiler) で使われているが、指すものは Linux Kernel ドライバーや Mesa3D で使われている *DG2-G1x* と同じ。  

*ACM-G10* と *ACM-G11* はどちらも TSMC 6nmプロセスで製造され、インターフェイスは PCIe Gen4、メモリには GDDR6 を採用する。  
ダイ間の違いは規模が主となり、*ACM-G10* は {{< xe >}}-Core 32基 (EU 512基)、L2キャッシュ 16MB、GDDR6 256-bit、PCIe Gen4 x16、  
対して *ACM-G11* は {{< xe >}}-Core 8基 (EU 128基)、L2キャッシュ 4MB、GDDR6 96-bit、PCIe Gen4 x8 となる。  
L1キャッシュ/SLM (Shared Local Memory) 192KB、メディアエンジン 2基、ディスプレイエンジン (パイプ) 4基という点は共通。  

ダイサイズとトランジスタ数は、*ACM-G10* が 406mm2 と 21.7B、*ACM-G11* が 157mm2 と 7.2B となっている。  

### ACM-G12 (DG2-G12) {#acm-g12}

オープンソース・ドライバーでは、*ACM/DG2-G10/G11* に加え *ACM/DG2-G12* のサポートが進められているが、今回の発表では触れられていない。  
IGC 内の記述から、*ACM/DG2-G12* は {{< xe >}}-Core 16基 (EU 256基) なると見られ、SKU としては *Intel Arc 5 A550M* と一致する。  
また、Mesa3D 内の記述から、L2キャッシュは 8MB になると見られる。  
{{< link >}} [DG2-G12, DG2 L3 banks, SIMD width | Coelacanth's Dream](/posts/2022/01/16/xe_hpg-hpc-eu-inst/#dg2-l3-banks) {{< /link >}}

 > 		         case IGFX_DG2:
 > 		+            /* 128 */
 > 		             if (TRUE == GFX_IS_DG2_G11_CONFIG(platform->getPlatformInfo().usDeviceID))
 > 		             {
 > 		                 InitAcm_G11HwWaTable(&waTable, pSkuFeatureTable, &stWaInitParam);
 > 		                 InitAcm_G11SwWaTable(&waTable, pSkuFeatureTable, &stWaInitParam);
 > 		             }
 > 		+            /* 256 */
 > 		+            else if (TRUE == GFX_IS_DG2_G12_CONFIG(platform->getPlatformInfo().usDeviceID))
 > 		+            {
 > 		+                InitAcm_G12HwWaTable(&waTable,pSkuFeatureTable, &stWaInitParam);
 > 		+                InitAcm_G12SwWaTable(&waTable,pSkuFeatureTable, &stWaInitParam);
 > 		+            }
 > 		+            /* 512 */
 >
 > {{< quote >}} [Add support for DG2 256 · intel/intel-graphics-compiler@44fe175](https://github.com/intel/intel-graphics-compiler/commit/44fe17526d180c1500e7038e025b82b027bddd9b) {{< /quote >}}

### メディアエンジン {#media}
*ACM/DG2-G10/G11* が AV1 HWエンコードに対応することは、データセンター向けとなる *Arctic Sound-Mainstream (ATS-M)* の発表時に明らかにされていたが、一般向け SKU でも対応することが今回発表された。  

発表内では、デコードは最大 8k60 12-bit HDR、エンコードは 8k 10-bit HDR に対応するとしている。  
一方、[intel/media-driver](https://github.com/intel/media-driver) の直近の変更によれば、デコードは HEVC/VP9/AV1 の場合、最大 16k、エンコードは HEVC では 16k、VP9 と AV1 は 8k に対応するとしている。  

* [media-driver/media_features.md at fcb8d5744a3c4ccc802e40c5d087ed4749764269 · intel/media-driver](https://github.com/intel/media-driver/blob/fcb8d5744a3c4ccc802e40c5d087ed4749764269/docs/media_features.md)

## ATS-M {#ats-m}

*ATS-M* のサポートを追加するパッチが、Intel の Matt Roper 氏より、Linux Kernel における Intel GPUドライバー、i915 のメーリングリストに投稿された。  
Matt Roper 氏は *ATS-M* を、(DG2 と同じ) `XE_HPG (Graphics ver: 12.55)` と `XE_HPM (Media ver: 12.55)` をベースするが、ディスプレイ出力はサポートしない、サーバー向けプラットフォームだと説明している。  
*ATS-M150* 、*ATS-M75* という名称とその DeviceID も追加されている。  
DeviceID と、*ATS-M150* が *ACM/DG2-G10* を、*ATS-M75* が *ACM/DG2-G11* をベースにする点は、Mesa3D の内容と一致する。  
{{< link >}} [DG2 と Arctic Sound-M の関係性 | Coelacanth's Dream](/posts/2022/02/05/dg2-ats_m/#m75-m150) {{< /link >}}

* [[Intel-gfx] [PATCH 1/2] drm/i915/ats-m: add ATS-M platform info](https://lists.freedesktop.org/archives/intel-gfx/2022-March/294043.html)
    * [[Intel-gfx] [PATCH 2/2] topic/core-for-CI: Add ATS-M PCI IDs](https://lists.freedesktop.org/archives/intel-gfx/2022-March/294044.html)
