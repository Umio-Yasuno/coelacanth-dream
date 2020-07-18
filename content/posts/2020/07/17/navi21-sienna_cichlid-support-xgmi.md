---
title: "Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート"
date: 2020-07-17T12:46:06+09:00
draft: false
tags: [ "RDNA_2", "Sienna_Cichlid", "gfx1030" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMD の次世代 [RDNA 2](/tags/rdna_2) GPU、*Navi21 /Sienna Cichlid* が高速なGPU間通信、**XGMI** をサポートする。  
**XGMI** は [Vega20](/tags/vega20) と [Arcturus](/tags/arcturus) もサポートしており[^xgmi-vega20-arcturus]、**Infinity Fabric Link** とも言い換えられるだろう。  
{{< link >}} [[PATCH] drm/amdgpu: enable xgmi support for sienna cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051633.html) {{< /link >}}

[^xgmi-vega20-arcturus]: [drm/amdgpu: Enable xgmi support for Arcturus](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=eb39aff7e0e33870319dd7c70a293c80e2df26c7)

## 増やされた SDMAコントローラは結局 XGMI のためか {#sdma-for-xgmi}

従来の AMD GPU と比べて多い *Sienna Cichlid* の SDMAコントローラについて、疑うようなことばかり書いてきたが、*Sienna Cichlid* が **XGMI** をサポートすることが明らかになった以上、つまるところ SDMAコントローラは **XGMI** のために増やされたのだろう。  

[The amd-gfx Archives](https://lists.freedesktop.org/archives/amd-gfx/) でのレビューを通らずに(見つけられなかった) 組み込まれたパッチを見るに、*Sienna Cichlid* は最大 4基の GPUとの接続を可能とするようだ。  
{{< link >}}[drm/amdgpu: add XGMI support for sienna cichlid](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdgpu/gfxhub_v2_1.c?h=amd-staging-drm-next&id=57bde5ec6ffbeb984981ab1a38d85b07df651bd2){{< /link >}}

 >       +	switch (adev->asic_type) {
 >       +	case CHIP_SIENNA_CICHLID:
 >       +		max_num_physical_nodes   = 4;
 >       +		max_physical_node_id     = 3;
 >       +		break;
 > 引用元: <cite>[drm/amdgpu: add XGMI support for sienna cichlid](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdgpu/gfxhub_v2_1.c?h=amd-staging-drm-next&id=57bde5ec6ffbeb984981ab1a38d85b07df651bd2)</cite>

この最大 4基というのは *Vega20* と同数であり[^vega20-max-nodes]、*Vega20* の場合は 3GPU以上での接続はリングバスの構成を取ることになり、それによって GPU間のデータ転送に幾許かの遅延が発生しうるものとなっていた。  

[^vega20-max-nodes]: [drm/amdgpu: Added ASIC specific checks in gfxhub V1.1 get XGMI info](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdgpu/gfxhub_v1_1.c?h=amd-staging-drm-next&id=f0312f45a0540a1551ca4644ff2461250520111a)

だが *Sienna Cichlid* は、SDMAコントローラ 2基である *Vega20* に対し[^vega20-sdma-controllers]、倍の 4基持つ[^sienna_cichlid-sdma-controllers]。  

[^vega20-sdma-controllers]: [kfd_device.c\amdkfd\amd\drm\gpu\drivers - ~agd5f/linux - Unnamed repository; edit this file 'description' to name the repository.](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdkfd/kfd_device.c?h=amd-staging-drm-next&id=3ace943f8f8158ee2b0fed8482edb37245b28f45#n356)
[^sienna_cichlid-sdma-controllers]: [[PATCH 021/207] drm/amdgpu: add sdma ip block for sienna_cichlid (v5)](https://lists.freedesktop.org/archives/amd-gfx/2020-June/049985.html)

もしかしたらリングバス構成を取らずに 4GPU との接続が可能 *かもしれない* 。未来のことだから、本当に断言できることなんてない。  

## 詳細はまだまだ不明 {#detail-unknown}
高速なGPU間通信 **XGMI** をサポートするといっても、帯域はどうなるかや、*Arcturus* のように、増やした 2基の SDMAコントローラを **XGMI** に最適化されたものにしなかったのは何故か等、不明な点はある。  
想像するならば、**XGMI** に最適化しなかったのは *Arcturus* 程、マルチGPUを前提にした構成ではなく、1GPUでの構成を普通に取るためだろうか。*Navi21 /Sienna Cichlid* は *Arcturus* のように GPGPU へ振り切ったものではなく、コンシューマ向けとして出ることが考えられる。  

マルチGPU構成を取るとして、Apple Mac Pro向けの **Radeon Pro Vega II Duo** のように、1カード上で 2GPUを *Infinity Fabric Link* で接続する形を取るのか[^radeon-pro-vega-ii-duo]、 **Radeon Pro VII** のように 2カードを専用ブリッジで接続する形を取るのかもまだわからない[^radeon-pro-vii]。  
ただこれは、どちらも *Vega20* ベースであるように、*Navi21 /Sienna Cichlid* もまた両方の形が可能と考えられる。  

[^radeon-pro-vega-ii-duo]: [Radeon™ Pro Vega II Graphics | AMD](https://www.amd.com/en/graphics/workstations-radeon-pro-vega-ii)
[^radeon-pro-vii]: [AMD Radeon™ Pro VII Graphics | AMD](https://www.amd.com/en/products/professional-graphics/radeon-pro-vii)

*Navi21 /Sienna Cichlid* は、まだしばらくは何かしら夢を持ちながら付き合う存在だ。  

{{< ref >}}

 * [深掘り！「AMD Next Horizon」 - Vega 7nm Deep Dive。7nmのVegaはコンシューマに来ない？ (3) Interconnect - GPU同士の接続がInfinityFabricに | マイナビニュース](https://news.mynavi.jp/article/20181227-20181227-amd_next_horizon/3)
 * [AMD，プロフェッショナル用途向け新型GPU「Radeon Pro VII」を発表。Radeon VIIベースの性能強化版](https://www.4gamer.net/games/133/G013322/20200513097/)
 * [西川善司の3DGE：PCIe Gen.4対応，そしてメモリバス帯域幅1TB/s到達。Vega 7nmは見るべきポイントの多いGPUだ](https://www.4gamer.net/games/133/G013322/20181110005/)

{{< /ref >}}
