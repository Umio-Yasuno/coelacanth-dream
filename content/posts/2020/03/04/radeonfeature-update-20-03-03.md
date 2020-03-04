---
title: "Xorg RadeonFeatureアップデート (2020-03-03)"
date: 2020-03-04T17:30:43+09:00
draft: false
tags: [ "Radeon", ]
keywords: [ "", ]
categories: [ "AMD", "GPU" ]
noindex: false
---

AMD公式のドキュメントである[Xorg RadeonFeature](https://www.x.org/wiki/RadeonFeature/)が更新された。  
更新内容は、[Renoir](/tags/renoir)の情報追加となる。  

ディスプレイコントローラー数は4基であることが明記され、最大画面出力数は前世代APUの[Raven](/tags/raven) /[Picasso](/tags/picasso)から変わらないことがわかった。  
また4基というのはLinux Kernelの記述とも一致する。[^1]  

[^1]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/dcn21/dcn21_resource.c?h=amd-staging-drm-next#n792>

そしてGFX Coreの世代は、Vega系、Ravenと同じである[GFX9](/tags/gfx9)とも明記された。  
商業的な理由か、AMDはRenoirのGPUに関しては性能をアピールし、どの世代にあたるかは微妙に濁してきた節がある。  
一応メディア等から質問された場合ははっきりとVegaだと回答してはいるが[^2][^3]、AMDの公式サイトの仕様ページでは、  
Graphics Model の項目が、Ryzen 2000/3000シリーズ APUでは Radeon RX Vega [CU数] Graphicsという形式であったのに対し、[^4]  
Ryzen Mobile 4000シリーズでは AMD Radeon Graphics となっている。[^5]  

[^4]: [AMD Ryzen™ 7 3700U Mobile Processor | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-3700u#product-specs)
[^5]: [AMD Ryzen™ 7 4700U | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-4700u#product-specs)

Ryzen Mobile 4000シリーズと同時に発表された、Raven2ベース (Zen APU) であるRyzen 3 3250U、Athlon Gold 3150U、Athlon Silver 3050Uでも同様の記述となっているため、自分の邪推と言ってしまえばそれまでだが、  
発表直後にRenoirのGPU部はNavi (GFX10)という *勘違い* が一部広まっていたのは否めないはずだ。  

[^2]: [AMD at CES 2020: Q&A with Dr. Lisa Su - AnadTech](https://www.anandtech.com/show/15344/amd-at-ces-2020-qa-with-dr-lisa-su)
[^3]: <https://twitter.com/Thracks/status/1215137876922396672>

知名度はそこまで高くはないものの、公開されているAMD公式のドキュメントに、  
*RenoirのGPUはGFX9 /Vegaと同世代* と記載されたことで、勘違いが{{< comple >}}発表からそれなりに時間が経ったため、ほぼなくなったとは思うが{{< /comple>}}完全になくなることを願う。  

<br>
今回の更新内容とは関係ないが、DCN 2.0行のディスプレイコントローラー数が6基のままとなっている。  
Navi10は6基搭載されているが、Navi14は5基であるため、 5-6 と記載するのが正確なはずだ。  

 > Navi10 has 6 PHY, but Navi14 only has 5 PHY, that is
 > because there is no ENGINE_ID_DIGD in Navi14.

 > 引用元: <cite>[drm/amd/display: Add ENGINE_ID_DIGD condition check for Navi14](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/display/dc/dcn20/dcn20_resource.c?h=amd-staging-drm-next&id=9fd4c2d712377f5fb9d3a1ad4f3106bf7833ccad)</cite>

まあそこまで重要ではない。  

