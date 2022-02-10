---
title: "DG2 と Arctic Sound-M の関係性"
date: 2022-02-05T06:58:31+09:00
draft: false
tags: [ "DG2", "Xe-HPG", "Xe-HP" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

個人的に気になった *DG2/Alchemist* と *ATS-M (Arctic Sound Mainstream)* の関係を一旦整理するだけの記事。  

## Arctic Sound-P/M {#ats-p_m}
Intel のエンジニアは早く (2018年) から開発中の、Intel 初 (?) となる Discrete GPU に *Arctic Sound* というコードネームを付けていることを明らかにしていた。[^ats-codename]  
当初 *Arctic Sound* は動画ストリーミング処理を担当するようなデータセンター向けの dGPU のみを対象にしたコードネームだったが、後にゲーミング向けにも対象を拡大した。  

[^ats-codename]: [Intel Is Developing A Desktop Gaming GPU To Fight Nvidia, AMD](https://www.forbes.com/sites/jasonevangelho/2018/04/11/intel-is-developing-a-desktop-gaming-gpu-to-fight-nvidia-amd/)

それからさらなる Intel dGPU の情報が公開されていくにつれ、その内容から *XeHP = Arctic Sound* とされるようになった。あくまでも自分の認識ではあるが。  

そして Intel oneAPIプロジェクトの 1つ、[oneVPL (oneAPI Video Processing Library)](https://github.com/oneapi-src/oneVPL/) にも *Arctic Sound* のサポートが追加された。  
それによると *Arctic Sound* には *ATS-P (Premium) / ATS-M (Mainstream)* バリアントが存在し、*XeHP SDV* が *ATS-P* 、*DG2/Alchemist* が *ATS-M* に対応している。  
oneVPL でサポートする Intel GPU プラットフォームは列挙体 (enum) で管理され、`MFX_PLATFORM_ARCTICSOUND_P` と `MFX_PLATFORM_XEHP_SDV` 、`MFX_PLATFORM_ATS_M` と `MFX_PLATFORM_DG2` には同じ値が割り当てられている。  
コメント部で、*ATS-M* は *DG2* と同じメディア機能を持つとしているが、*DG2* と分けた上で同じ値を割り当てた意図は不明。  

{{< bq cite="[oneVPL-intel-gpu/mfxcommon.h at 5d0987b9b3ebe60844402dda532909075b5b7d02 · oneapi-src/oneVPL-intel-gpu](https://github.com/oneapi-src/oneVPL-intel-gpu/blob/5d0987b9b3ebe60844402dda532909075b5b7d02/api/vpl/mfxcommon.h#L200-L207)" >}}
 > 		    MFX_PLATFORM_ARCTICSOUND_P  = 45,
 > 		    MFX_PLATFORM_XEHP_SDV       = 45, /*!< Code name XeHP SDV. */
 > 		    MFX_PLATFORM_DG2            = 46, /*!< Code name DG2. */
 > 		    MFX_PLATFORM_ATS_M          = 46, /*!< Code name ATS-M, same media functionality as DG2. */
 > 		#ifdef ONEVPL_EXPERIMENTAL
 > 		    MFX_PLATFORM_ALDERLAKE_N    = 55, /*!< Code name Alder Lake N. */
 > 		#endif
 > 		    MFX_PLATFORM_KEEMBAY        = 50, /*!< Code name Keem Bay. */
{{< /bq >}}

### ATS-M75/150 {#m75-m150}
Mesa3D の *DG2* サポートは進められているが、今は一旦関連するコード部が無効化されている。  
そうした中で、*DG2-G10/G11* の DeviceID をさらに追加するパッチが投稿された。  
その追加された DeviceID は他と異なり、*DG2* ではなく *ATS-M75/M150* という名前が付けられている。  

* [intel/dev: Add more DG2 pci-ids (still disabled, waiting on i915) (!14884) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14884) 

{{< bq cite="[intel/dev: Add more DG2 pci-ids (still disabled, waiting on i915) (!14884) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14884/)" >}}
 > 		 CHIPSET(0x56a6, dg2_g11, "DG2", "Intel(R) Graphics")
 > 		 CHIPSET(0x56b0, dg2_g11, "DG2", "Intel(R) Graphics")
 > 		 CHIPSET(0x56b1, dg2_g11, "DG2", "Intel(R) Graphics")
 > 		+CHIPSET(0x56c0, dg2_g10, "ATS-M150", "Intel(R) Graphics")
 > 		+CHIPSET(0x56c1, dg2_g11, "ATS-M75", "Intel(R) Graphics")
{{< /bq >}}

関数形式のマクロ `CHIPSET` の引数は `(_id, _family, _fam_str, _name)` という並びになっている。 [^chipset]  
`_id` (DeviceID) から GPU を検出、`_family` を元にその GPUファミリーが持つとしている機能の情報を取得する。`_fam_str, _name` はデバイス名として使われ、今回追加されたものだと実際には *Intel(R) Graphics (ATS-M75)* といったように表示される。`_fam_str` の部分には通常、マーケティング的な、SKU名が入る。  
そのため、異なるのはデバイス名の補足的な部分のみとなる。  
*ATS-M* が上と oneVPL でサポートされているものと同じだとして、末尾の数字は消費電力とかを表しているのだろうか？  

[^chipset]: [src/intel/dev/intel_device_info.c · 4f9141607f40f0be9cee38ff6b006a05bba72e88 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/d9416cd8bd437bd877b21c685ccd28bbb425d7eb/src/intel/dev/intel_device_info.c#L1317-1341)

`_family` が `dg2_g10/dg2_g11` となっていることから *DG2* と同じダイをベースにしていると思われ、また他の Intel GPU に関する OSS でも、`0x56C0/0x56C1` は *DG2* に割り当てられた DeviceID となっている。[^igc-did]  
*DG2/Alchemist* と *ATS-M* はほぼ同じ機能、存在とされながら、現状プログラム的には影響が及ばさない部分で区別されている。これは *Xe-HP_SDV* と *ATS-P* の関係に対しても言える。  
[Intel(R) Media Driver for VAAPI](https://github.com/intel/media-driver) には `0x56C0/0x56C1` を対象にした部分があったが、メディアデコード/エンコード処理を複数のパイプラインで処理するかのフラグを無効としてセットする、というもので今後変更される可能性が高いように思う。 [^media-driver-ats_m]  

[^igc-did]: [intel-graphics-compiler/igfxfmid.h at 70fe72803348a2e44bcdbb31c6f50c9f448ede0a · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/70fe72803348a2e44bcdbb31c6f50c9f448ede0a/inc/common/igfxfmid.h#L644-L677)
[^media-driver-ats_m]: [media-driver/media_interfaces_dg2.cpp at 4e5c0b680fda6e108919ed2b2409858eb300d6a3 · intel/media-driver](https://github.com/intel/media-driver/blob/4e5c0b680fda6e108919ed2b2409858eb300d6a3/media_driver/media_interface/media_interfaces_dg2/media_interfaces_dg2.cpp#L527-L531)

*Arctic Sound* が *Premium (Xe-HP_SDV)* 、*Mainstream (DG2/Alchemist)* に分離したのははっきりとしているが、何故か対応する GPU とは、影響の無い範囲で区別されている。  
可能性として、一部機能を対象に、ハードウェア側はサポートしているが、ドライバーとしては一部 SKU のみをサポートする、といったことが今後あるのかもしれない。  
Intel のチーフアーキテクトである Raja Koduri 氏は、過去のインタビューにおいて *DG2/Alchemist* でワークステーション向け市場をサポートする意思を見せている。 (ただし具体的な説明は避けている)  
機能で差別化するとしたら、それはゲーミング向けとワークステーション向け、あるいはサーバー向けの SKU の間で行われるように思う。[^ascii]  

[^ascii]: [ASCII.jp：Ponte VecchioとIntel Arcに関する疑問をRaja Koduri氏が回答　インテル GPUロードマップ (3/3)](https://ascii.jp/elem/000/004/069/4069704/3/)
[^xe-hp]: <https://twitter.com/Rajaontheedge/status/1453808598283210752>

{{< ref >}}
* [Intel Xe - Wikipedia](https://ja.wikipedia.org/wiki/Intel_Xe#Xe-HP)
* [Intel Is Developing A Desktop Gaming GPU To Fight Nvidia, AMD](https://www.forbes.com/sites/jasonevangelho/2018/04/11/intel-is-developing-a-desktop-gaming-gpu-to-fight-nvidia-amd/)
* [Updated: Intel Cans Xe-HP Server GPU Products, Shifts Focus To Xe-HPC and Xe-HPG](https://www.anandtech.com/show/17041/intel-cans-xehp-gpu-products-shifts-focus-to-xehpc-and-xehpg)
* [ASCII.jp：Ponte VecchioとIntel Arcに関する疑問をRaja Koduri氏が回答　インテル GPUロードマップ (3/3)](https://ascii.jp/elem/000/004/069/4069704/3/)
{{< /ref >}}
