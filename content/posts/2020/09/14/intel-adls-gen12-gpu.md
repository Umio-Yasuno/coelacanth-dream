---
title: "Intel Alder Lake-S の GPU は Gen12アーキテクチャ"
date: 2020-09-14T21:07:41+09:00
draft: false
tags: [ "Alder_Lake", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

CPU部は *Golden Coce (Core)* と *Gracemont (Atom)* のハイブリッド構成を取ることが明かされている *Alder Lake* だが、[intel/gmmlib](https://github.com/intel/gmmlib) に組み込まれた変更から、  
デスクトップ向けとされる *Alder Lake-S* の GPU部は [Tiger Lake](/tags/tiger_lake)、[Rocket Lake](/tags/rocket_lake)、[DG1](/tags/dg1)等と同じ [Gen12アーキテクチャ](/tags/gen12) であることが分かった。  
{{< link >}} [Added the ADL-S device ID's and phyAddr support · intel/gmmlib@2072b0d](https://github.com/intel/gmmlib/commit/2072b0d1e8ba2cba2f94bc2c1fda89d6e457a50b) {{< /link >}}

*Alder Lake-S* に関するコードが `gmmlib/GmmGen12Platform.cpp` に追加されている。  

 >          if(GFX_GET_CURRENT_PRODUCT(Data.Platform) == IGFX_ALDERLAKE_S)
 >          {
 >              Data.NoOfBitsSupported                = 46;
 >              Data.HighestAcceptablePhysicalAddress = GFX_MASK_LARGE(0, 45);
 >          }
 >
 > {{< quote>}} [gmmlib/GmmGen12Platform.cpp at 2072b0d1e8ba2cba2f94bc2c1fda89d6e457a50b · intel/gmmlib](https://github.com/intel/gmmlib/blob/2072b0d1e8ba2cba2f94bc2c1fda89d6e457a50b/Source/GmmLib/Platform/GmmGen12Platform.cpp) {{< /quote >}}

{{< ins datetime="2020-10-01" >}}

また、[Intel-Media-SDK/MediaSDK](https://github.com/Intel-Media-SDK/MediaSDK) でも *Alder Lake-S* は Gen12LP に関連付けられている。  
{{< link >}} [[ADL-S] Enable ADL-S platform support by aidan2020sh · Pull Request #2381 · Intel-Media-SDK/MediaSDK](https://github.com/Intel-Media-SDK/MediaSDK/pull/2381/files) {{< /link >}}

{{< /ins >}}


省電力、モバイル向けとされる *Alder Lake-P* に関してはまだ確定していないが、同様に *Gen12アーキテクチャ* である可能性が高いだろう。  

{{< pindex >}}

 * [Alder Lake-S iGPU](#adls-igpu)
   * [Alder Lake-S は最大 GT1、Alder Lake-P は最大 GT2、それと DDR5](#adls-gt1_adlp-gt2)

{{< /pindex >}}

## Alder Lake-S iGPU {#adls-igpu}

まず、*Alde Lake-S* の GPU に関しては、以前に SiSoftware Official Live Ranker に投稿された **Intel Alder Lake Client Platform Alder Lake Client System (Intel AlderLake-S ADP-S DRR4 CRB)** の結果から、256SP (32EU)、L2キャッシュ 512KB という規模が判明していた。[^sissoftware-adls]  

[^sissoftware-adls]: [Details for Computer/Device Intel Alder Lake Client Platform Alder Lake Client System (Intel AlderLake-S ADP-S DRR4 CRB) : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_system.php?q=cea598ab93aa98a193b5d2efc9bb86a0c9f4d2ba87a1d9e4c2a7c2ffcfe99aa79f&l=en)

この L2キャッシュというのはそのままの意味ではなく、恐らくソフトウェアから検出された URB (Unified Return Buffer) のサイズを表していると思われる。そして URB は GPU部に搭載された L3キャッシュの一部を割り当てる形を取るため、搭載された GPUキャッシュのある階層の規模を表すのでもないことを留意したい。  

この時点ではまだ、*Gen12 GT1* の規模と概ね一致するという認識だった。  
それと、オープンソースドライバーのコードを読むに、ある Intel GPU の最大規模はアーキテクチャとグレード(GTx) で判定されるため、例えば同じ *Gen12アーキテクチャ* である *Tiger Lake GT1* と *Rocket Lake GT1* は同規模となる。  
ただ L3キャッシュバンクあたりの容量やピクセルバックエンドの規模はプロセッサで異なる場合がある。  

### Alder Lake-S は最大 GT1、Alder Lake-P は最大 GT2、それと DDR5 {#adls-gt1_adlp-gt2}

以前 Coreboot に投稿されたパッチから、デスクトップ向けの *Alder Lake-S* は最大で
 GT1 、省電力、モバイル向けの *Alder Lake-P* は最大 GT2 になると見られる。  
 {{< link >}} [Coreboot に Alder Lake のデバイスIDが追加　―― ADL-S の GPU は最大で GT1、ADL-P は GT2 か | Coelacanth's Dream](/posts/2020/08/04/adl-coreboot/) {{< /link >}}

前述したように、GPU 規模はグレード(GTx) で判定されているため、GPU に限って見れば、デスクトップ向けとしては *RKL GT1* と、モバイル向けとしては *TGL GT2* と規模は変わらないだろう。  
しかし、*Alder Lake-S/P* は DDR5 に対応するため、デスクトップ向けとしてはそれなりの性能向上が見込めるかもしれない。  

*Alder Lake-S/P* は 2基のメモリコントローラを持ち、DDR4/LPDDR4/LPDDR5/DDR5 に対応する。[^adl-mc]  

 >       +-------------+--------+---------+
 >       | Memory Type | Max Ch | Max DPC |
 >       +-------------+--------+---------+
 >       | DDR4        |      1 |       2 |
 >       | LPDDR4(X)   |      4 |       1 |
 >       | LPDDR5      |      4 |       1 |
 >       | DDR5        |      2 |       1 |
 >       +-------------+--------+---------+
 >
 >    These numbers are for a single memory controller. Since ADL has two memory controllers, there's twice as many channels in total. DPC stays the same.
 >
 >
 > {{< quote>}} <https://review.coreboot.org/c/coreboot/+/45192/7/src/soc/intel/alderlake/meminit.c#109> {{< /quote >}}

*Tiger Lake-U* も LPDDR5 に対応しているため、*Tiger Lake-U* から *Alder Lake-P* にかけては目立った GPU性能の向上は難しいかもしれないが、デスクトップ向けとしては DDR5 対応により実感できるものとなるだろう。そうなると GPU の規模が少し寂しく思えてくるが。  
それと DDR5 では 1DPC(DIMM per channel) となるため、メモリ容量を求めて 2DPC が可能な DDR4 でのシステム構成が出てくるかもしれない。  
このあたりは DDR5メモリと対応プラットフォームの及の勢い次第だが。  

[^adl-mc]: <https://review.coreboot.org/c/coreboot/+/45192/7/src/soc/intel/alderlake/meminit.c#109>

*Alder Lake* は 2021年の後半に市場へ投入されると Intel は発表している。  
それまでは *Gen12アーキテクチャ* 、より正確に言えば *{{< xe class="lp" >}}アーキテクチャ* がクライアント向けプロセッサの iGPU として継続される。  
期間で言えば、*Gen9アーキテクチャ* は *Skylake* から *Comet Lake* まで続けて採用され、その間には *Kaby Lake (Amber Lake)* 、*Coffee Lake* 、*Whiskey Lake* 、Atom系プロセッサでは *Apollo Lake* 、*Gemini Lake* が存在する。  
Core系はプロセス更新失敗とそれに伴う *Cannon Lake* のキャンセルといった事情が絡み、*Skylakeアーキテクチャ* と共に *Gen9アーキテクチャ* がここまで続くこととなった。  
しかし、*Gen12アーキテクチャ /{{< xe class="lp" >}}* はプロセスも CPUアーキテクチャも異なる 3種のプロセッサ (*Tiger Lake /Rocket Lake /Alder Lake-S*) での採用となり、*Gen9アーキテクチャ* とは理由を全く異にする。  
これは *Gen12アーキテクチャ /{{< xe class="lp" >}}* の優秀さ {{< comple >}} 将来を見据えた細部に渡る拡張、メディア部の強化 (AV1 HWデコード)…… {{< /comple >}} を示していると言える。  

<!--

*Alder Lake* は 2021年の後半に市場へ投入されると Intel は発表している。  
それまでは *Gen12アーキテクチャ* 、より正確に言えば *{{< xe class="lp" >}}アーキテクチャ* が継続される訳だが、{{< xe class="lp" >}} は性能向上と用途拡大のため前世代から細部に渡って拡張が施され、メディア部では AV1 HWデコードに対応と、将来を見据えた設計になっている。  
そう陳腐化してしまうこともないだろう。  

-->

| Intel Gen12 | GT0.5 | GT1 | GT2 | GT2 (DG1) |
| :-- | :--: | :--: | :--: | :--: |
| Dual-Sub Slices | 1 | 2 | 6 | 6 |
| &ensp;EUs | 16 | 32 | 96 | 96 |
| &ensp;&ensp;SPs | 128 | 256 | 768 | 768 |
| GPU L3$ | 1536KB? | 1536KB? | 3072KB | 16384KB[^dg1-l3] |
| | RKL | TGL /RKL<br>/ADL-S | TGL /ADL-P? | DG1 |

[^dg1-l3]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)
