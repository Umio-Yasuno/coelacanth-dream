---
title: "一部の AMD GPU で実装、有効化されている RB+ とは何か"
date: 2020-11-10T20:09:50+09:00
draft: false
tags: [ "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU", "Note" ]
noindex: false
# summary: ""
toc: false
---

解説記事というよりはメモ、ノート的な記事。  

まず、AMD GPU には RB (Render Backend) というユニットがあり、RB は MSAA や EQAA 等のアンチエイリアス処理、深度テストを行ない、各ピクセルが最終的にフレームに表示されるかの判定を処理し、結果をメモリに書き出す役割を持つ。  
RB はクロックあたり 4ピクセルを処理することができ (4 pixel/cycle)、ROP で言えば 4基相当となる。  
{{< link >}} [ROP は何の略 | Coelacanth's Dream](/posts/2020/05/31/what-does-rop-stand-for/) {{< /link >}}

そして、AMD GPU ドライバーのコードを読むに *RB+* というものが存在し、一部のチップに実装、有効化されている。  
*RB+* の何が + なのかを調べたのがこの記事。  

## 一部の APU/GPU に実装、有効化 {#some-gpu}

*RB+ / RB Plus* は、オープンソースドライバーの [src/amd/common/ac_gpu_info.c](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/common/ac_gpu_info.c) によると、*Stoney APU (gfx810)* と GFX9 とそれ以降世代の APU/GPU に実装されているが、すべてが有効にできるのではなく、一部のみとなっている。  
有効可能なものには、APU だと *Stoney* 、[Raven/Picasso (gfx902)](/tags/raven)、[Raven2 (gfx909)](/tags/raven2)、[Renoir (gfx90c)](/tags/renoir)、[VanGogh](/tags/vangogh) が、  
dGPU では *Vega12 (gfx904)* 、*RDNA 2 / GFX10.3* 世代である [Sienna Cichlid](/tags/sienna_cichlid) 、[Navy Flounder](/tags/navy_flounder) 、[Dimgrey Cavefish](/tags/dimgrey_cavefish) が該当する。  

機能的には持っているはずだが、無効化されている GPU には、*Vega10 (gfx900)* 、 *Vega20 (gfx906)* 、*Navi10 (gfx1010)* 、*Navi12 (gfx1011)* 、 *Navi14 (gfx1012)* がある。  
これらの GPU で無効化されている理由は不明。  
APU では基本有効化されていることから、メモリ帯域が関係していると考えられる。  
ただ *GFX9 / Vega* 世代以上の dGPU では、HBM2 2048-bit を備える *Vega10* 、HBM2 4096-bit の *Vega20* では無効化され、HBM2 1024-bit である *Vega12* では有効化され、  
*RDNA / GFX10* 世代の *Navi10* GDDR6 256-bit、*Navi14* GDDR6 128-bit では無効化されているあたり、単純にメモリ帯域で決めているようにも思えない。  

 >       info->has_rbplus = info->family == CHIP_STONEY || info->chip_class >= GFX9;
 >    
 >       /* Some chips have RB+ registers, but don't support RB+. Those must
 >        * always disable it.
 >        */
 >       info->rbplus_allowed =
 >          info->has_rbplus &&
 >          (info->family == CHIP_STONEY || info->family == CHIP_VEGA12 || info->family == CHIP_RAVEN ||
 >           info->family == CHIP_RAVEN2 || info->family == CHIP_RENOIR || info->chip_class >= GFX10_3);
 >
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 3c2489d2e45b3013361c7284ed9de14fe40554cc · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/3c2489d2e45b3013361c7284ed9de14fe40554cc/src/amd/common/ac_gpu_info.c) {{< /quote >}}

## RB+ の機能 {#rbplus}

*RB+* がどういった機能を持つかは、結論からさっさと言えば、クロックあたりに処理可能な最大ピクセル数を 8ピクセルに拡張したものと思われる。  

以下は GPUOpen Driver を構成する一つである [PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal) の一部コードだが、  
クロックあたりに処理可能なピクセル数を、*Shader Engine数 \* SEあたりの RB数 \* (RB+ が有効なら 8pixel、そうでないなら 4pixel)* で求めている。  

 >       // Pixels per clock follows the following calculation:
 >       // GPU__GC__NUM_SE * GPU__GC__NUM_RB_PER_SE * (RBPlus ? 8 : 4)
 >       pInfo->pixelsPerClock = (pInfo->gfx6.numShaderEngines * pInfo->gfx6.maxNumRbPerSe * (pInfo->gfx6.rbPlus ? 8 : 4));
 >
 > {{< quote >}} [pal/gfx6Device.cpp at 007816f4bd2fd3ace30f8af9629cff7ec1dcb998 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/007816f4bd2fd3ace30f8af9629cff7ec1dcb998/src/core/hw/gfxip/gfx6/gfx6Device.cpp#L3260) {{< /quote >}}

また、*Raven APU* は RB を 2基 (8ROP相当) 持つが、AMD公式のスライドでは *16 Pixels Units* となっている。[^raven-slide]  

[^raven-slide]: [Delivering a new level of visual performance in an SoC AMD "Raven Rid…](https://www.slideshare.net/AMD/delivering-a-new-level-of-visual-performance-in-an-soc-amd-raven-rdige-apu)

ただ RB の規模を倍に拡張した訳ではないようで、*RB+* が有効な場合、通常の倍の性能を得るため、入力するシェーダーのピクセルフォーマットを特定のものに指定する必要があるとされている。[^rbplus-format]  

[^rbplus-format]: [ac,radv: use better export formats for 8-bit when RB+ isn't allowed (!7512) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7512/diffs) <br> [pal/gfx9RsrcProcMgr.cpp at 4ae736bdbc5d5dee59851ac564c5e21d807b44b0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/4ae736bdbc5d5dee59851ac564c5e21d807b44b0/src/core/hw/gfxip/rpm/gfx9/gfx9RsrcProcMgr.cpp#L276)

そして、{{< comple >}} 調べるきっかけともなった {{< /comple >}} 気になる点は、*RDNA 2 / GFX10.3* 世代の GPU、**Radeon RX 6000シリーズ** ではどうなっているかである。  
**Radeon RX 6000シリーズ** は ROP数が AMD公式の製品ページに記載されており、**RX 6900 XT** は 128ROP となっている。[^rx-6900-xt]  
この 128ROP という規模を、*RB+* を 16基搭載することで実現しているのか、それとも *RB* を 32基搭載しているのか。  
上述したように、*RDNA 2 / GFX10.3* 世代の APU/GPU は *RB+* が実装、有効化されているため、前者の可能性が高いように思われる。  
また、AMD が公開している **Radeon RX 6000シリーズ** のダイショット {{< comple >}} に近い CG {{< /comple >}} を見るに、RB と思われるユニットは 16基確認でき、*RB+* として相当する ROP数をカウントしていると考えれば辻褄が合う。  
{{< link >}} [AMD Navi10のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/01/22/navi10-dieshot-and-guess/) {{< /link >}}
{{< link >}} [AMD Radeon™ RX 6000 Series die shot](https://www.globenewswire.com/NewsRoom/AttachmentNg/56b9f51f-b313-41a3-9fc1-0f1bf766c3d4/en) {{< /link >}}

以前はオープンソースドライバー ( *RadeonSI (OpenGL), RADV (Vulkan)* ) に*RB+* の機能を無効化するオプションが存在していたが、現在では削除されている。  
そして、判定部に *"RDNA 2 / GFX10.3 以上の世代"* とあることから、これからの AMD GPU は *RB+* がデフォルトとなるのではないかと思われる。  

[^rx-6900-xt]: [AMD Radeon RX 6900 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6900-xt#product-specs)
