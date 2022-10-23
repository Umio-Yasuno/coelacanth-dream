---
title: "Intel Meteor Lake GPU は Xe-LPG に、Xe-LPM+ では引き続き AV1 エンコードをサポートか"
date: 2022-10-22T05:47:41+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Xe-HPG", "Meteor_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

## {{< xe >}}-LPG {#xe-lpg}
Intel GPU ドライバー (`i915`) 等で *Meteor Lake* の Graphics IP を指す名前として *{{< xe >}}-LPG* が使われ始めている。  

 * [[Intel-gfx] [PATCH v3 13/14] drm/i915/xelpg: Add multicast steering](https://lists.freedesktop.org/archives/intel-gfx/2022-October/309132.html)
 * [Introducing MTL Support · intel/gmmlib@e7d5cb0](https://github.com/intel/gmmlib/commit/e7d5cb083f6ff3a606d056303f0f7802e97cbca7)
 * [Meteor Lake platform support · intel/cm-compiler@e1b0daa](https://github.com/intel/cm-compiler/commit/e1b0daaa90f5ca215500a89a1cb281dec8966bbc)

 >         MTL's graphics IP (Xe_LPG) once again changes the multicast register
 >         types and steering details.  Key changes from past platforms:
 >          * The number of instances of some MCR types (NODE, OAAL2, and GAM) vary
 >            according to the MTL subplatform and cannot be read from fuse
 >            registers.  However steering to instance #0 will always provided a
 >            non-terminated value, so we can lump these all into a single
 >            "instance0" table.
 >          * The MCR steering register (and its bitfields) has changed.
 >
 > {{< quote >}} [[Intel-gfx] [PATCH v3 13/14] drm/i915/xelpg: Add multicast steering](https://lists.freedesktop.org/archives/intel-gfx/2022-October/309132.html) {{< /quote >}}

 >         diff --git a/clang/compute_sdk/docs/cmcuserguide/cmcuserguide.rst b/clang/compute_sdk/docs/cmcuserguide/cmcuserguide.rst
 >         index 7836235111b..efa0e34bb83 100644
 >         --- a/clang/compute_sdk/docs/cmcuserguide/cmcuserguide.rst
 >         +++ b/clang/compute_sdk/docs/cmcuserguide/cmcuserguide.rst
 >         @@ -79,6 +79,7 @@ GEN12     TGLLP    CM_GEN12    1200          0
 >          ...       ADLN     CM_GEN12    1240          0
 >          XEHP_SDV  XEHP_SDV CM_XEHP     1270          0
 >          XeHPG     DG2      CM_XEHPG    1271          0
 >         +XeLPG     MTL      CM_XELPG    1275          0
 >          XeHPC     PVC      CM_XEHPC    1280          0
 >          ...       PVCXT    CM_XEHPC    1280          5
 >          ========= ======== =========== ============= ===================
 >         
 >
 > {{< quote >}} [Meteor Lake platform support · intel/cm-compiler@e1b0daa](https://github.com/intel/cm-compiler/commit/e1b0daaa90f5ca215500a89a1cb281dec8966bbc) {{< /quote >}}

*Meteor Lake GPU* の Display IP には *{{< xe >}}-LPD+ (ver14)*、Media IP には *{{< xe >}}-LPM+ (ver13)* という IP 名が付けられているため、単に統一感を持たせたとも取れる。  
一方で、*{{< xe >}}-LPG* は今後 *{{< xe >}}-HPG 系列* をベースにしながらも別の系列として更新していくが考えられる。  

IGC (intel-graphics-compiler) 等へのパッチから、*Meteor Lake GPU ({{< xe >}}-LPG)* は *DG2/Alchemist*、*{{< xe >}}-HPG* と基本共通した特徴を持ち、レイトレーシングやメッシュシェーダーをサポートするとされているが、XMX ユニットを持たない (`DPAS (Dot Product Accumulate Systolic)` 命令をサポートしない) こととハードウェア的な倍精度演算のサポートが一部復活する点で異なっている。[^igc]  
また、Intel は *{{< xe >}}-HPG 系列* は今後 *{{< xe gen="2" >}}-HPG (Battlemage)*、*{{< xe gen="3" >}}-HPG (Celestial)*、*{{< xe >}}Next (Druid)* として更新していくことを発表している。[^xe-roadmap]  

[^igc]: [Intel Graphics Compiler で Meteor Lake のサポートが進み始める ―― Xe-HPG、XMXユニットは非搭載か、再度 FP64 に対応 | Coelacanth's Dream](/posts/2022/07/06/igc-mtl/)
[^xe-roadmap]: [Intel、ゲーマー向け次世代GPU「Arc」をチラ見せ。ハイエンドまで対応可能な伸縮アーキテクチャ - PC Watch](https://pc.watch.impress.co.jp/docs/news/1345044.html)

今後はデスクトップ向けに *{{< xe >}}-HPG*、内蔵 GPU 向けに *{{< xe >}}-LPG* として分岐していくのだろう。  

| Intel GPU ({ver}.{release_ver}) | Graphics IP ver | Display IP ver | Media IP ver |
| :-- | :--: | :--: | :--: |
| Tiger Lake, Rocket Lake? | 12 | 12 | 12 |
| DG1 | 12.1 | 12 | 12 |
| Alder Lake-S | 12 | 12 | 12.2 |
| Alder Lake-P | 12 | 13 (XE_LPD) | 12.2 |
| {{< xe >}}-HP | 12.5 | - | 12.5 |
| {{< xe >}}-HPG (DG2/Alchemist) | 12.55 | 13 (XE_LPD) | 12.55 |
| {{< xe >}}-HPC | 12.6 | - | 12.6 |
| Meteor Lake | 12.7 (XE_LPG) | 14 (XE_LPD+) | 13 (XE_LPM+) |

## {{< xe >}}-LPM+ は引き続き AV1 エンコードをサポートか {#xe-lpm}
oneAPI Video Processing Library (oneVPL) の GPU ランタイム (oneVPL-intel-gpu) にて、*Meteor Lake* の Media Engine、*{{< xe >}}-LPM+* のサポートを追加するコミットが取り込まれている。  
なお [intel/media-driver](https://github.com/intel/media-driver) では、コミット内のコメントで *{{< xe >}}-LPM+* に触れてはいるが、まだ実際のソースファイルやコミットは公開されていない。  

 * [[Common] MTL open source (#3635) · oneapi-src/oneVPL-intel-gpu@dc395df](https://github.com/oneapi-src/oneVPL-intel-gpu/commit/dc395df5185741cd49f9d12617a968f29fbf1605)
 * [[Encode] Upstream MTL Encoders (#3752) · oneapi-src/oneVPL-intel-gpu@9f74297](https://github.com/oneapi-src/oneVPL-intel-gpu/commit/9f74297329365b8bafed91c4031215dce12b38b5)

oneVPL-intel-gpu へのコミットでは、*{{< xe >}}-LPM+* 向けに HEVC エンコードと AV1 エンコードに関するファイルが追加されており、*DG2/Alchemist* から引き続いて AV1 エンコードがサポートされると推測される。  
それ以外では、*{{< xe >}}-LPM+* から AV1 デコードのポストプロセッシングがサポートされている。  

ただ Linux Kernel における Intel GPU ドライバー (`i915`) へのパッチから、*{{< xe >}}-LPM+* のエンコードエンジン (VEBox, Video Enhancement Engine) 数は 1基と、*DG2/Alchemist* の 2基よりも小規模となっている。[^xe-lpmp]  
メモリ帯域も影響するだろうが、*{{< xe >}}-LPM+* と *DG2/Alchemist* とでメディアエンジンの性能を比較した時、*{{< xe >}}-LPM+* の方がエンコード性能が低くなると思われる。  

| Intel Media Engine | VEBox | VDBox |
| :--                | :--:  | :--:  |
| TGL/DG1/ADL        | 1     | 2     |
| {{< xe >}}-HP SDV  | 4     | 4     |
| DG2/Alchemist      | 2     | 2     |
| {{< xe >}}-LPM+    | 1?    | 2?    |

[^xe-lpmp]: [GPU と Media Engine が別々のタイルに搭載される Meteor Lake | Coelacanth's Dream](/posts/2022/08/30/intel-mtl-media-engine/)

{{< ref >}}
 * [Intel GFX ドライバーに Meteor Lake をサポートする最初のパッチが投稿される ―― Gen12.7, Xe_LPD+, Xe_LPM+ | Coelacanth's Dream](/posts/2022/07/07/i915-mtl/)
{{< /ref >}}
