---
title: "エンコード機能を持たない VCN v5.0.1"
date: 2025-01-02T18:26:46+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU", "APU" ]
tags: [ "Linux_Kernel", "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

2024-12-05、AMD の Alex Deucher 氏によって VCN (Video Core/Codec Next) IP v5.0.1 のサポートを AMDGPU ドライバーに追加するパッチがメーリングリストに公開された。  
同時に JPEG (JPEG コーデック用のデコードエンジン) IP v5.0.1 のサポートに関するパッチも公開されている。  
v5.0.1 というのは現時点で VCN IP としては最新のバージョンとなる。  

 * [[PATCH 05/10] drm/amdgpu: Add VCN_5_0_1 firmware](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117613.html)
 * [[PATCH 06/10] drm/amdgpu: Add VCN_5_0_1 codec query](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117618.html)
 * [[PATCH 07/10] drm/amdgpu: Add JPEG5_0_1 support](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117619.html)
 * [[PATCH 08/10] drm/amdgpu: enable JPEG5_0_1 ip block](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117616.html)
 * [[PATCH 09/10] drm/amdgpu: Add VCN_5_0_1 support](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117620.html)
 * [[PATCH 10/10] drm/amdgpu: Enable VCN_5_0_1 IP block](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117617.html)

## エンコード機能は持たず {#enc}
AMDGPU ドライバーには VCN IP がサポートするコーデック情報がハードコードされており、User Mode Driver が API 経由でその情報を取得できるようになっている。  
そして、今回追加された VCN IP v5.0.1 のコーデック情報によれば、VCN v5.0.1 はメディアエンジンとしてエンコード機能を持たず、デコード機能のみを持つ。  

 >         diff --git a/drivers/gpu/drm/amd/amdgpu/soc15.c b/drivers/gpu/drm/amd/amdgpu/soc15.c
 >         index 3bb4a573e07b2..a59b4c36cad73 100644
 >         --- a/drivers/gpu/drm/amd/amdgpu/soc15.c
 >         +++ b/drivers/gpu/drm/amd/amdgpu/soc15.c
 >         @@ -171,6 +171,24 @@ static const struct amdgpu_video_codecs vcn_4_0_3_video_codecs_encode = {
 >          	.codec_array = NULL,
 >          };
 >         
 >         +static const struct amdgpu_video_codecs vcn_5_0_1_video_codecs_encode_vcn0 = {
 >         +	.codec_count = 0,
 >         +	.codec_array = NULL,
 >         +};
 >         +
 >         +static const struct amdgpu_video_codec_info vcn_5_0_1_video_codecs_decode_array_vcn0[] = {
 >         +	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_MPEG4_AVC, 4096, 4096, 52)},
 >         +	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_HEVC, 8192, 4352, 186)},
 >         +	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_JPEG, 16384, 16384, 0)},
 >         +	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_VP9, 8192, 4352, 0)},
 >         +	{codec_info_build(AMDGPU_INFO_VIDEO_CAPS_CODEC_IDX_AV1, 8192, 4352, 0)},
 >         +};
 >         +
 >         +static const struct amdgpu_video_codecs vcn_5_0_1_video_codecs_decode_vcn0 = {
 >         +	.codec_count = ARRAY_SIZE(vcn_5_0_1_video_codecs_decode_array_vcn0),
 >         +	.codec_array = vcn_5_0_1_video_codecs_decode_array_vcn0,
 >         +};
 >         +
 >
 > {{< quote >}} [[PATCH 06/10] drm/amdgpu: Add VCN_5_0_1 codec query](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117618.html) {{< /quote >}}

VCN v5.0.1 が何の AMD GPU に搭載されるかについては、現在 AMDGPU ドライバーは IPブロックごとにサポートする方式であり、他の IPブロックと結び付けられている情報は少ない。  
しかし、完全に IPブロックのサポートが独立している訳ではないため、ある程度は推測可能である。  

VCN v5.0.1 のコーデック情報は `drivers/gpu/drm/amd/amdgpu/soc15.c` に追加されており、`soc15` は *Vega* 系の APU/GPU や *CDNA* 系の GPU のサポートに関係している。  
また、*MI300/Aqua Vanjaram/CDNA 3* はメディアエンジンとして VCN v4.0.3 と JPEG v4.0.3 を搭載しており、こちらもエンコード機能は持たない。  
ということで VCN IP v5.0.1 は次世代の *CDNA* 系 GPU にメディアエンジンとして搭載される IPブロックと推測できる。  

ちなみに、VCN IP v5.0.0 は `drivers/gpu/drm/amd/amdgpu/soc24.c` にコーデック情報が記述されており、そして `soc24` は *RDNA 4* 系の GPU のサポートに関係している。  
