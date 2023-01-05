---
title: "AMD、モバイル向け Ryzen 7000 シリーズを正式発表"
date: 2023-01-05T17:25:02+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Phoenix", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD は 2023/01/04 付、CES 2023 にてモバイル向け Ryzen 7000 シリーズを正式発表した。  

 * [AMD Extends its Leadership with the Introduction of its Broadest Portfolio of High-Performance PC Products for Mobile and Desktop :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1111/amdextends-its-leadership-with-the-introduction-of-its)
 * [AMD Highlights Future of High-Performance and Adaptive Computing During Opening Keynote of CES 2023 :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1109/amd-highlights-future-of-high-performance-and-adaptive)
 
モバイル向け Ryzen 7000 シリーズは複数のセグメントで構成されており、基本的にそれぞれ別のシリコンダイをベースとしている。  
Ryzen 7045HX シリーズは *Dragon Range (Zen 4, RDNA 2, 5nm + 6nm)*、
Ryzen 7040/HS シリーズは *Phoenix (Zen 4, RDNA 3, 4nm)*、
Ryzen 7035 シリーズは *Rembrandt-R (Zen 3+, RDNA 2, 6nm)*、
Ryzen 7030 シリーズは *Barcelo-R (Zen 3, Vega, 7nm)* ベースとなっている。  

## Dragon Range
*Dragon Range* はチップレット構成であり、CCD と IOD のダイサイズがデスクトップ向け Ryzen 7000 シリーズ、*Raphael* と一致することから、*Raphael* を TDP 55-75W に抑えたモデルと考えていいだろう。  
*Dragon Range* の CPU ソケット、パッケージは *Raphael* の AM5 と異なり、FL1 とされている。  

 * [AMD Ryzen™ 9 7945HX | AMD](https://www.amd.com/en/product/13016)

以前、AMDGPU ドライバーに *GC 11.0.4 APU* のサポートを追加するパッチが投稿された際、それが *Dragon Range* に向けたものではないかと推測したが、実際は異なり、*Dragon Range* の GPU 部は *RDNA 2 アーキテクチャ* となっている。[^gc_11_0_4]  
*GC 11.0.4* は *Phoenix APU (GC 11.0.3)* のリビジョン違いか、今回の CES 2023 では発表されなかった APU を指すのだろう。  

[^gc_11_0_4]: [もう 1つの RDNA 3 APU IP、GC 11.0.4 | Coelacanth's Dream](/posts/2022/11/22/rdna_3-apu-gc_11_0_4/)

## Phoenix
まずニュースリリースでは Ryzen 7040HS シリーズの最大合計キャッシュサイズが 40MB、SKU ごとのページでは 24MB (8x L2 1MB + L3 16MB)、各ニュースサイトで公開されているメディア向け資料では 20MB となっており統一されていない。  
おそらく SKU ごとのページの 24MB (8x L2 1MB + L3 16MB) が正しいキャッシュサイズと思われるが、確証が得られない。  

*Phoenix* の GPU部は *RDNA 3 アーキテクチャ* となり、CU 数は最大 12基。  
**Radeon 780M** は GPUクロック 2900/3000MHz、CU 数 12基、
**Radeon 760M** は GPUクロック 2800MHz、CU 数 8基の構成となっている。
GPU キャッシュサイズや RB (RenderBackend) 数等の規模は公開されていない。  

ダイサイズは公称 178mm2。CPU と GPU 共にアーキテクチャが更新され、**Ryzen AI** と呼ぶ AI エンジンが追加されたが、TSMC 4nmプロセスの採用により、(実際の製造コストはともかく) *Rembrandt* より小さいダイサイズに収まっている。  

 * [AMD Ryzen™ 9 7940HS | AMD](https://www.amd.com/en/product/13036)

| AMD APU | Die Size | Process |
| :--     | :--:     | :--:    |
| Renoir/Lucienne | 156mm^2 | 7nm |
| Cezanne/Barcelo | 180mm^2 | 7nm |
| Rembrandt | 210mm^2 | 6nm    |
| Mendocino | 100mm^2 | 6nm    |
| Phoenix   | 178mm^2 | 4nm    |

## Rembrandt-R
*Rembrandt* ベースの Ryzen 6000 シリーズと *Rembrandt-R* ベースの Ryzen 7035 シリーズで異なるのは、同コア同 TDP で比較したときに一部モデルのブーストクロックが若干 (+50MHz) 向上しているくらいで、大きな違いはない。  

Ryzen 6000 シリーズには無かった 4-Core/8-Thread の SKU が、Ryzen 7035 シリーズでは **Ryzen 3 7335U** として追加されている。  
*Rembrandt/-R* の最大構成から、CPU と L3キャッシュを半分無効化し、GPU CU 数を 4基に減らした SKU となる。  
これまでは **Radeon 660M** のモデル名は *Rembrandt* ベースの GPUクロック 1800MHz、GPU CU数 6基の構成に割り当てられていたが、それより小規模の **Ryzen 3 7335U** の GPU部にも割り当てられており、APU に GPUモデル名を導入した意義が薄れているように思う。  

 * [AMD Ryzen™ 3 7335U | AMD](https://www.amd.com/en/product/12961)
