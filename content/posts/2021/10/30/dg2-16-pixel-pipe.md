---
title: "最大 16本の Pixel Pipe を持つ Intel DG2"
date: 2021-10-30T08:55:27+09:00
draft: false
tags: [ "DG2", "Gen12", "Xe-HPG" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

まだドラフトで、マージリクエストの段階ではあるが、Intel *{{< xe class="hp/hpg" >}}* の Pixel pipe を最適化するパッチがオープンソースドライバー Mesa3D に投稿されている。  
GFXバージョン (`GFX_VER`) では Gfx12.5、*{{< xe class="hp" >}}* も対象に入っているが、パッチのコメントでは主に *DG2/{{< xe class="hpg" >}}* を想定している。  

 * [Draft: intel: Pixel pipeline optimizations for XeHP hardware. (!13569) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13569)

以前にも *{{< xe class="lp" >}}* 向けに Pixel pipe を最適化するパッチが投稿されていた。  
Intel Gen12/{{< xe >}}プラットフォームでは事前に設定された Pixel pipe のハッシュテーブルを用いるが、それはすべての Pixel pipe に等しく EU あるいは Sub-Slice が割り当てられていることを前提としていた。それにより *{{< xe class="lp" >}}* の最大構成 6 Sub-Slice (96 EU) より小さい規模の構成を採る SKU では、アンバランスな割り当てがボトルネックとなり性能が低下することとなっていた。  
パッチではそれを修正し、適切に EU を Pixel pipe に割り当てることを目的としたもので、80 EU (5 Sub-Slice ?) 構成の *Tiger Lake* において 10%~63% の性能改善を果たしている。  

 * [intel: Fix pixel-pipe performance bottlenecks on fused Gen12 parts. (!8749) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/8749)

*{{< xe class="lp" >}}* は最大で 3本の Pixel pipe、それぞれ ROP 8基相当の Pixel Backend を 3基持っている。  

 > 		+#define GEN_DEVICE_MAX_PIXEL_PIPES      (3)  /* Maximum on gen12 */
 >
 > {{< quote >}} [intel: Fix pixel-pipe performance bottlenecks on fused Gen12 parts. (!8749) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/8749/diffs?commit_id=e2ef1c46760ba6996fd49f5fc56d56e1af8d2220#diff-content-777d253fd71902d8f51ea827069054d0bf1d731a) {{< /quote >}}

今回の *DG2/{{< xe class="hpg" >}}* に向けたパッチも同様のものと思われ、*{{< xe class="lp" >}}* から大幅に増えた Pixel pipe、EU、Sub-Slice に対応している。  

## 最大で 16本の Pixel pipe {#16-pipes}

*DG2/{{< xe class="hpg" >}}* においても、最大構成から一部 EU、Sub-Slice を無効化した構成を予定しているらしく、マージリクエストのコメントでは今回のパッチで **DG2-448** の場合、20%-40% の性能改善を確認できたとしている。  
デモシーンやベンチマーク以外にゲームでも確かな性能向上を果たしており、規模が大きくなった分 *{{< xe class="lp" >}}* よりも影響と恩恵が大きいのかもしれない。  

 > This series is part of the XeHP enabling effort.  It's not strictly required for functional correctness, but it improves performance significantly for most non-trivial workloads on all DG2 platforms it's been tested on so far, particularly on fused configurations -- E.g. on DG2-448 it gives us a 20%-40% performance improvement on most interesting workloads:
 >
 > {{< quote >}} [Draft: intel: Pixel pipeline optimizations for XeHP hardware. (!13569) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13569) {{< /quote >}}

*DG2/{{< xe class="hpg" >}}* については、各 Sub-Slice ({{< xe >}}-Core) が EU 16基 (Vector Engine + Matrix Engine) とレイトレーシングユニット、テクスチャサンプラーを持ち、その Sub-Slice 4基と固定機能ユニット、Pixel Backend 2基で *Render Slice* を構成する図が Intel Architecture Day 2021 で示されていた。  
*Render Slice* あたり Sub-Slice 4基、EU 64基、Pixel Backend 2基 (ROP 16基相当?) となる。  
巨大な L2キャッシュとそこに *Render Slice* 8基 (Sub-Slice 32基、EU 512基、Pixel Backend 16基) を搭載した構成も示されており、低消費電力ソリューションからエンスージアストクラスのゲーミングソリューションまでスケールアップ可能としている。  

パッチでは *DG2* は最大 16本の Pixel pipe を持つとしており、これは Intel が示した図の Pixel Backend 数とも一致する。  
また、Sub-Slice 2基 (EU 32基) に対して Pixel Backend 1基というバランスは *{{< xe class="lp" >}}* と同じ。  

 > 		+#define INTEL_DEVICE_MAX_PIXEL_PIPES      (16) /* Maximum on DG2 */
 >
 > {{< quote >}} [Draft: intel: Pixel pipeline optimizations for XeHP hardware. (!13569) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13569/diffs?commit_id=d6f4f19155321dd0895f6c3cb14fc22318585421#diff-content-712a8df50d1196c189650dc493dacd43b7afed44) {{< /quote >}}

*DG2* が最大 EU 512基とすると、コメントで触れられた **DG2-448** は *Render Slice* 1基を無効化して 7基 (Sub-Slice 28基、EU 448基、Pixel Backend 14基) という構成と思われるが、それだと Sub-Slice ごとに持つレイトレーシングユニット、テクスチャサンプラー、*Render Slice* ごとに持つ Pixel Backend も無効化することになるため、*Render Slice* 8基はそのままに、Sub-Slice あたりの EU を 56基とする構成も考えられる。  
また、**DG2-128** という構成にも触れており、こちらは *Render Slice* 2基で構成した低消費電力向けの構成と思われる。  

 > On DG2-128 this seems to improve SynMark2/OglDrvRes by 16.0% ±0.1%,
 > n=8.
 >
 > {{< quote >}} [Draft: intel: Pixel pipeline optimizations for XeHP hardware. (!13569) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13569/diffs?commit_id=a392fccccfed48400993948d6e8b2b35c86af930) {{< /quote >}}

| | DG2-128 | DG2-448 | DG2-512 |
| :-- | :--: | :--: | :--: |
| Total Render Slice | 2 ? | 7/8 ? | 8 ? |
| Total Sub-Slice | 8 | 28/32 ? | 32 ? |
| EU per Sub-Slice | 16 ? | 14/16 ? | 16 ? |
| Total Pixel backend | 4 ? | 14/16 ? | 16 ? |

{{< ref >}}
  * [Xe-HPG Microarchitecture](https://www.intel.com/content/www/us/en/architecture-and-technology/visual-technology/arc-discrete-graphics/xe-hpg-microarchitecture.html)
{{< /ref >}}
