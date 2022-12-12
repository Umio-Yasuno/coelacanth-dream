---
title: "Meteor Lake GPU GT2/GT3 の EU数"
date: 2022-12-12T18:22:56+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Meteor_Lake", "Xe-HPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Lionel Landwerlin 氏により、Mesa3D ライブラリ、ドライバーに *Meteor Lake GPU* の EU数と GPU構成に関するパッチ、マージリクエストが投稿されている。  

 * <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/20228>

パッチは Intel GPU がハードウェア内部に持つ Performance Counter Unit を用いた性能測定において、*Meteor Lake GPU* のサポートを追加するものとなる。  
ハードウェア内部のユニットを用いる関係上、パッチには *Meteor Lake GPU* の EU数や GPU構成の情報が含まれる。  

## Meteor Lake GT2/GT3 {#gt2_gt3}
パッチでは *Meteor Lake-M/P* 共通して、EU数が 64基以下は GT2、それに当てはまらず、かつ 128基以下の場合は GT3 の設定ファイルを適用している。  
*Meteor Lake* からは CPU/GPU/SoC 部をそれぞれタイル (ダイ) に分けて搭載するタイルアーキテクチャが採用されるが、恐らく現状では GT2 (EU 最大 64基)、GT3 (EU 最大 128基) の GPUタイルの構成が用意されているのではないかと思われる。  

 >         +   case INTEL_PLATFORM_MTL_M:
 >         +   case INTEL_PLATFORM_MTL_P:
 >         +      if (intel_device_info_eu_total(devinfo) <= 64)
 >         +         return intel_oa_register_queries_mtlgt2;
 >         +      if (intel_device_info_eu_total(devinfo) <= 128)
 >         +         return intel_oa_register_queries_mtlgt3;
 >         +      return NULL;
 >
 > {{< quote >}} [intel: add MTL performance metrics (!20228) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/20228) {{< /quote >}}

また、設定ファイルを見ると、GT2 は `Slice 0 Xe Core 3` までの、GT3 は `Slice 1 Xe Core 3` までの設定が用意されている。  
それらの記述から、*Meteor Lake GPU ({{< xe >}}-LPG)* でも *{{< xe >}}-HPG* と同じ Render Slice あたり {{< xe >}}-Core 4基 (4x EU 16基) の構成を採り、そして GT2 は Render Slice 1基、GT3 は Render Slice 2基という規模であることが読み取れる。  

 >         +    <counter name="Slice1 Xe Core3 Input Available"
 >         +             symbol_name="Sampler13InputAvailable"
 >         +             underscore_name="sampler13_input_available"
 >         +             description="The percentage of time in which slice1 Xe core3 sampler input is available"
 >         +             data_type="float"
 >         +             max_equation="100"
 >         +             units="percent"
 >         +             semantic_type="duration"
 >         +             equation="C 4 READ 100 UMUL $GpuCoreClocks FDIV"
 >         +             availability="$GtSlice1XeCore3"
 >         +             mdapi_group="GPU/Sampler"
 >         +             mdapi_usage_flags="Tier3 Overview Frame Batch Draw"
 >         +             mdapi_supported_apis=""
 >         +             mdapi_hw_unit_type="dualsubslice"
 >         +             />
 >
 > {{< quote >}} [intel: add MTL performance metrics (!20228) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/20228) {{< /quote >}}

*Meteor Lake* における、製品情報やドキュメント中では L2、ドライバー中では L3 とされているキャッシュの詳細は不明だが、キャッシュバンク数については *{{< xe >}}-HPG* と同じ設定だとすると GT2 はバンク数 6基、GT3 はバンク数 8基となる。  
キャッシュバンクあたりのサイズも同じ 512KiB である場合、GT2 は 3072KiB、GT3 は 4096KiB となる。  

 >         static void
 >         update_l3_banks(struct intel_device_info *devinfo)
 >         {
 >            if (devinfo->ver != 12)
 >               return;
 >         
 >            if (devinfo->verx10 >= 125) {
 >               if (devinfo->subslice_total > 16) {
 >                  assert(devinfo->subslice_total <= 32);
 >                  devinfo->l3_banks = 32;
 >               } else if (devinfo->subslice_total > 8) {
 >                  devinfo->l3_banks = 16;
 >               } else {
 >                  devinfo->l3_banks = 8;
 >               }
 >            } else {
 >               assert(devinfo->num_slices == 1);
 >               if (devinfo->subslice_total >= 6) {
 >                  assert(devinfo->subslice_total == 6);
 >                  devinfo->l3_banks = 8;
 >               } else if (devinfo->subslice_total > 2) {
 >                  devinfo->l3_banks = 6;
 >               } else {
 >                  devinfo->l3_banks = 4;
 >               }
 >            }
 >         }
 >         
 > {{< quote >}} [src/intel/dev/intel_device_info.c · a099d6ae4d133fdef4a81fc2fbcfb8c7b9b5a440 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/a099d6ae4d133fdef4a81fc2fbcfb8c7b9b5a440/src/intel/dev/intel_device_info.c#L1185-1211) {{< /quote >}}

今回のパッチから、*Meteor Lake GPU ({{< xe >}}-LPG) GT3* では *DG2-G11/ACM-G11* と EU数では同規模の EU 128基になると見られるが、*Tiger Lake GPU GT2* の EU 96基、L3キャッシュ 3840KiB と比べると向上幅が小さいように感じるかもしれない。  
しかし、*Meteor Lake GPU ({{< xe >}}-LPG)* ではレイトレーシングや Mesh Shader のサポート、ハードウェア的な FP64 データタイプのサポートが復活する等、機能の追加や改良が施されている。  
あとはより高速なメモリのサポートによるメモリ帯域の向上が期待できる。  

{{< ref >}}
 * [Introduction to the Xe-HPG Architecture](https://www.intel.com/content/www/us/en/developer/articles/technical/introduction-to-the-xe-hpg-architecture.html)
{{< /ref >}}
