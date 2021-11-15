---
title: "Linux Kernel に Intel Xe-HP SDV, DG2 をサポートする最初のパッチが投稿される"
date: 2021-07-02T06:26:52+09:00
draft: false
tags: [ "Linux_Kernel", "Xe-HP", "DG2", "Xe-HPG", "XE_LPD", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

intel-gfx メーリングリストに、Linux Kernel部の Intel GPU ドライバーに *{{< xe class="hp" >}} SDV (Software Development Vehicle)* 、 *DG2 ({{< xe class="hpg" >}})* プラットフォームのサポートを追加する一連のパッチが投稿、公開された。  

 * [[Intel-gfx] [PATCH 00/53] Begin enabling Xe_HP SDV and DG2 platforms](https://lists.freedesktop.org/archives/intel-gfx/2021-July/270869.html)

*{{< xe class="hp" >}}/{{< xe class="hpg" >}}* は、パッチを公開し、オープンソースソフトウェアにサポートを追加する段階まで進んだらしく、Linux Kernel 以外にも OpenCLコンパイラ [intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler) 等にもパッチが投稿され、すでにメインラインに取り込まれている。  
User Mode Driver (UMD) である Mesa3D には、 *{{< xe class="hpg" >}}* で大幅に設計が見直されたとするデータポート *LSC (Load Store Cache)* のサポートを追加するパッチが投稿されている。[^lsc]  

[^lsc]: [intel/compiler: Add support for the LSC (!11600) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/11600)

今回の一連のパッチではまだ実際に動作させることはできないが、今後 LMEM (Local Memory) や GuC (Graphics microController) ファームウェア、*{{< xe class="hp" >}}* が採るマルチタイルアーキテクチャ、それと GPUコアの外部に搭載されると思われる専用のコンピュートエンジンなどのサポートは将来的に追加する予定にあると、パッチのコメントで語られている。  

*{{< xe class="hp" >}} アーキテクチャ* は GPU IPバージョンに Gen12.5、*{{< xe class="hpg" >}} アーキテクチャ* には Gen12.55 が割り当てられている。  

*{{< xe class="hp" >}} アーキテクチャ* では Gen9アーキテクチャまでの "Slice(s)" という概念が無くなり、新たなコンセプトに GSlice (Geometry Slice), CSlice (Compute Slice), MSlice (Memory Slice) が導入されているとしている。コードを読むに *{{< xe class="hpg" >}} アーキテクチャ* も同様のコンセプトが導入している。  
Slice(s) は一部の固定ユニットと L3 Data Cache、EU (Execution Unit) をまとめた Sub-Slice を複数持つクラスタであり、AMD GPU アーキテクチャにおける Shader Engine が意味や役割としては近いだろう。  
*Gen11 アーキテクチャ* から複数の Slice が Intel GPU に搭載されることはなくなり、単一の Slice に従来の Intel GPU よりも多い Sub-Slice を搭載する形となっていたため、以前より Slice という単語が持つ意味は薄くなっていた。  
また、*Gen12/{{< xe >}} アーキテクチャ* からは Sub-Slice あたりの EU数が、従来の倍となる 16基となり、それを理由とするのか、Sub-Slice を Dual Sub-Slice と呼ぶようになった。  
*{{< xe class="hp/g" >}} アーキテクチャ* では巨大な Dual Sub-Slice のプールを持ち、それを GSlice/CSlice/MSlice に分割して割り当てる形となるようだが、まだ多くはわかっていない。  

## 2つのバリアントが存在する {{< xe class="hpg" >}} {#xe-hpg}

*DG2* ではディスプレイIP に *Alder Lake-P* と同じ *Xe_LPD (Display13)* を採用している。  
{{< link >}} [Linux Kernel に 「Intel Display13」をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/29/intel-display13/) {{< /link >}}

そして、*DG2* には *DG2-G10* と *DG2-G11* の 2つのバリアントが存在し、それらのハードウェアステッピングは一方から続くものではない、珍しい方式となっている。  
バリアント間の違いは今の所、ハードウェアに存在する細かなバグとそれに対する回避策 (ワークアラウンド) くらいしかわからない。  

 * [[Intel-gfx] [PATCH 24/53] drm/i915/dg2: add DG2 platform info](https://lists.freedesktop.org/archives/intel-gfx/2021-July/270876.html)

個人的な感覚で言えば *{{< xe class="hp/g" >}} アーキテクチャ* は、*Gen11 アーキテクチャ* から *{{< xe class="lp" >}}Gen12.[0-2] アーキテクチャ* へ進んだ時以上に、多くの機能や概念が導入されており、正直ちゃんと読めていない部分が多い。  
全体的な構成とリソースの割り当て方法が変更され、データポートも EU内の実行パイプラインも刷新されている。ディスクリート GPU 向けの *{{< xe class="hp/g" >}} アーキテクチャ* には、統合 GPU を前提としていた従来の Intel GPU アーキテクチャでは推し量れない部分が多くある。  
{{< link >}} [Intel Xe-HP EU に追加されるパイプラインと増加するスレッド/レジスタファイル | Coelacanth's Dream](/posts/2021/06/08/intel-xe_hp-thread-reg-pipe/) {{< /link >}}
そうした部分を Intel GPUアーキテクトがどういった意図で設計、搭載したのか――公式の発表、ドキュメント、あるいはパッチ、どのような形であれ明かされる時が楽しみだ。  

| Intel GPU | Graphics IP ver | Display IP ver | Media IP ver |
| :-- | :--: | :--: | :--: |
| Tiger Lake,<br>Rocket Lake? | Gen12 | Gen12 | Gen12 |
| DG1 | Gen12.1 | Gen12 | Gen12 |
| Alder Lake-S | Gen12 | Gen12 | Gen12.2[^gen12_2] |
| Alder Lake-P | Gen12 | Gen13<br>([XE_LPD](/tags/xe_lpd)) | Gen12.2[^gen12_2] |
| {{< xe class="hp" >}} | Gen12.5 | - | Gen12.5 |
| {{< xe class="hpg" >}} | Gen12.55 | Gen13<br>([XE_LPD](/tags/xe_lpd)) | Gen12.55 |

[^gen12_2]: [Add tests to detect and check MFX runtime. by uartie · Pull Request #306 · intel/vaapi-fits](https://github.com/intel/vaapi-fits/pull/306)

| {{< xe >}} EU |  |  | Issue Port 0 | IP 1 | IP 2 | IP 3 | IP 4? |
| :-- | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| {{< xe class="lp" >}} | Branch<br>(Control Flow) | Send | Short<br>(INT /FP) | Math | - | - | - |
| {{< xe class="hp" >}} |  | Send | INT? (int8/16/32)<br> /Branch? | Math? | DPAS? | Long?<br>(Int64 /FP64) | FP?<br>(FP16/32, BF16) |
|                       | In-Order | Out-of-Order | In-Order | Out-of-Order | Out-of-Order | In-Order | In-Order |
{{< link >}} [intel-graphics-compiler/RegDeps.cpp at f5e9e3dcda319339b82f1171c7e55add46884036 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/f5e9e3dcda319339b82f1171c7e55add46884036/visa/iga/IGALibrary/IR/RegDeps.cpp#L100) {{< /link >}}

