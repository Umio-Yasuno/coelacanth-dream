---
title: "Linux Kernel に RDNA 2 GPU 「Dimgrey Cavefish」 をサポートするパッチが投稿される"
date: 2020-10-08T01:47:35+09:00
draft: false
tags: [ "Dimgrey_Cavefish", "RDNA_2", "Navi23" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

次世代 *RDNA 2 /GFX10.3* 世代の GPU、*Dimgrey Cavefish* をサポートするパッチが Linux Kernel (amd-gfx) に投稿された。  
{{< link >}} [[PATCH 00/50] Add support for Dimgrey Cavefish](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054538.html) {{< /link >}}

## DCN3.02 {#dcn3_02}
[Sienna Cichlid](/tags/sienna_cichlid)、[Navy Flounder](/tags/navy_flounder) が DCN3 を、[VanGogh APU](/tags/vangogh) はマイナーバージョンとなる DCN3.01 と搭載するのに対し、*Dimgrey Cavefish* はまたびみょうにバージョンが進んだ DCN3.02 を搭載する。  
{{< link >}} [[PATCH 49/50] drm/amd/display: Add support for DCN302 (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054588.html) {{< /link >}}

*Dimgrey Cavefish* の最大画面出力数は *Sienna Cichlid, Navy Flounder* よりも 1画面少なくなり、DCN3 を搭載する *Sienna Cichlid, Navy Flounder* は最大 6画面出力だが、DCN3.02 の *Dimgrey Cavefish* では 最大 5画面となる。  
{{< comple >}} GPU の規模に関する推測を何かしら立てるのは苦手、嫌いとも言えるが {{< /comple >}}最大画面出力数を 1画面減らすのは、*Polaris10* に対する *Polaris11/12* 、 *Navi10* に対する *Navi14* 等、ある GPU よりも規模を抑えた GPU によく見られる設計である。  

 >      +static const struct resource_caps res_cap_dcn302 = {
 >      +		.num_timing_generator = 5,
 >      +		.num_opp = 5,
 >      +		.num_video_plane = 5,
 >      +		.num_audio = 5,
 >      +		.num_stream_encoder = 5,
 >      +		.num_dwb = 1,
 >      +		.num_ddc = 5,
 >      +		.num_vmid = 16,
 >      +		.num_mpc_3dlut = 2,
 >      +		.num_dsc = 5,
 >      +};
 >      +
 >
 > {{< quote >}} [[PATCH 49/50] drm/amd/display: Add support for DCN302 (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054588.html) {{< /quote >}}

<br>
他は概ね *Sienna Cichlid, Navy Flounder* と同じとなっており、マルチメディアエンジンには VCN3 1基、SDMAエンジンは 2基となっている[^kfd]。
なお、XGMI をサポートする *Sienna Cichlid* は SDMAエンジンを 4基搭載している。[^sienna_cichlid-xgmi]  

[^kfd]: [[PATCH 29/50] drm/amdkfd: Support dimgrey_cavefish KFD (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054567.html)
[^sienna_cichlid-xgmi]: [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/)
