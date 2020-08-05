---
title: "Coreboot に Alder Lake のデバイスIDが追加　―― ADL-S の GPU は最大で GT1、ADL-P は GT2 か"
date: 2020-08-04T11:19:26+09:00
draft: false
tags: [ "Alder_Lake" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

オープンソースなBIOS、ファームウェアを開発するプロジェクト [Coreboot](https://www.coreboot.org/) に *Alder Lake-S/P* の DeviceID を追加するパッチが投稿された。  
{{< link >}} [soc/intel/common: Include Alderlake device IDs (I17ce56a2) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/44108) {{< /link >}}

{{< pindex >}}


 * [Alder Lake iGPU ( GT0/GT1/GT2 )](#adl-igpu)
 * [Alder Lake PCH PCIe](#adl-pcie)
 * [Alder Lake 推測](#adl-guess)

{{< /pindex >}}

## Alder Lake iGPU ( GT0/GT1/GT2 ) {#adl-igpu}

*Alder Lake* の GPU部に付けられたマクロ名から、*Alder Lake* GPU には **GT0、GT1、GT2** の 3種類が存在し、*Alder Lake-S* は最大で **GT1** 、*Alder Lake-P* は最大で **GT2** の規模を持つことがわかる。[^adl-gpu-deviceid]  

[^adl-gpu-deviceid]: <https://review.coreboot.org/c/coreboot/+/44108/1/src/include/device/pci_ids.h#3607>

Intel 次世代のデスクトップ向けプロセッサと見込まれている [Rocket Lake](/tags/rocket_lake) もまた GPU部の規模は最大で **GT1** までとなっている。  
そしてベンチマーク結果から、*Alder Lake-S* の GPU構成は *Gen12 GT1* の規模と一致することがわかっている。[^adl-s-sisoftware]  
{{< link >}} [Intel、オープンソースドライバーに DG1 と Rocket Lake の関するコードを追加 ――DG1 は 96EU、RKL は 16EU または 32EU | Coelacanth's Dream](/posts/2020/05/08/intel-add-dg1-rkl-oss-driver/) {{< /link >}}

[^adl-s-sisoftware]: [Details for Computer/Device Intel Alder Lake Client Platform Alder Lake Client System (Intel AlderLake-S ADP-S DRR4 CRB) : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_system.php?q=cea598ab93aa98a193b5d2efc9bb86a0c9f4d2ba87a1d9e4c2a7c2ffcfe99aa79f&l=en)

**GT0** については Coreboot 内のファイルには記載がないが、[Intel(R) Graphics Memory Management Library](https://github.com/intel/gmmlib) 内の *TGL LP GT0* に対する記載から[^tgl-lp-gt0]、GFX(Slice/Sub-Slice/EU?) 機能は持たない、もしくは無効化されており、ディスプレイ出力のみに対応したバリアントと考えられる。  

[^tgl-lp-gt0]: [gmmlib/igfxfmid.h at master · intel/gmmlib](https://github.com/intel/gmmlib/blob/master/Source/inc/common/igfxfmid.h)

## Alder Lake PCH PCIe {#adl-pcie}
*Alder Lake-S* と *Alder Lake-P* で PCH(Platform Controller Hub) が持つ PCIe RP(Root Port) の DeviceID 数が異なり、  
*ADP_S (Alder Lake PCH S /Alder Point S?)* は 28-RP、*ADP_P (Alder Lake PCH P /Alder Point?)* は 12-RP となっている。  

*Comet Lake* 世代のものと比較すると、デスクトップ向けである *Comet Lake-S* の PCH、 *CMP_H* は 24-RP、モバイル向けの *CMP_LP* は 16-RP であり、  
*Alder Lake-S* は末尾、GPU部規模からも推測されるようにデスクトップ向けのプロセッサと考えられ、*Alder Lake-P* はモバイル向け、または組み込み向けといった小規模なシステム向けのプロセッサと考えられる。  

次世代 Atom系プロセッサにおいて、モバイル向けとなる *JSP (Jasper Lake PCH /Jasper Point?)* は 8-RP、組み込み向けの *MCC (Mule Creek Canyon, Elkhart Lake PCH)* は 7-RP であり、*ADP_P* は Core系 と Atom系の中間の PCIe RP(Root Port) を持つと言える。  
しかし、Linux Kernel へのあるパッチで、*Alder Lake(-S)* のモデルナンバー(`x86_Model`) が明らかになると共に、[Lakefield](/tags/lakefiled)同様に Core系と Atom系の両方のコアを持つハイブリッド構成を取ることが確かとなったが、  
*Alder Lake-P* もまたハイブリッド構成を取るかどうかは朧気である。  
{{< link >}} [Intel Alder Lake が ハイブリッドコア構成を取ることが確かに | Coelacanth's Dream](/posts/2020/07/21/intel-adl-hybrid-core/) {{< /link >}}

## Alder Lake 推測 {#adl-guess}
まず *Alder Lake-S* はデスクトップ向けと考えて問題ないのではないかと思う。  
*Alder Lake-P* はモバイル向けか組み込み向けかはっきりしないが、**GT2** の DeviceID を持つことから、モバイル向けにおいて GPU性能を重視している Intel の最近の傾向からモバイル向けである色が濃いように思う。  
*Alder Lake-S* は最大 **GT1** 、*Alder Lake-P* は **GT2** というのは、10th Gen プロセッサに存在する、CPU性能は *Comet Lake* 、GPU性能は *Ice Lake* という棲み分けを引き継いだ結果であるようにも思える。  

| Intel | CPU | GPU |
| :-- | :--: | :--: |
| 10th Gen | Comet Lake | Ice Lake |
| 11th Gen | Rocket Lake? | Tiger Lake? |
| 12th Gen? | Alder Lake-S?? | Alder Lake-P?? |

ただ、*Alder Lake-S/P* の GPU が *Tiger Lake /Rocket Lake* より進んだアーキテクチャではなく、同じ [Gen12アーキテクチャ](/tags/gen12) を取るのであれば、規模もまた変わらない可能性がある。  
オープンソース・ドライバーの記述から、*Gen12アーキテクチャ* はプロセッサごとではなく、**GT** の後に続く数字からその規模を判定しているからだ。[^gen12-dev-info]  
もしそれが合っているとして、*Alder Lake-P GT2* が *Tiger Lake LP GT2* より上の GPU性能を出すにはクロックを引き上げるかメモリ帯域を増やす必要があると思われる。  

[^gen12-dev-info]: [src/intel/dev/gen_device_info.c · 559b26b7ee093c2cbe446bf8023876642deadcee · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/559b26b7ee093c2cbe446bf8023876642deadcee/src/intel/dev/gen_device_info.c)

| Intel Gen12 | GT0.5 | GT1 | GT2 | GT2 (DG1) |
| :-- | :--: | :--: | :--: | :--: |
| Dual-Sub Slices | 1 | 2 | 6 | 6 |
| &ensp;EUs | 16 | 32 | 96 | 96 |
| &ensp;&ensp;SPs | 128 | 256 | 768 | 768 |
| GPU L3$ | 1536KB? | 1536KB? | 3072KB | 16384KB[^dg1-l3] |
| | RKL | TGL /RKL<br>/ADL-S? | TGL /ADL-P? | DG1 |

[^dg1-l3]: [Intel、DG1 において OpenCL と oneAPI Level Zero をサポート　―― 巨大なキャッシュを持つ DG1 | Coelacanth's Dream](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)
