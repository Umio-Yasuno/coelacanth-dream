---
title: "Intel Movidius VPU に関するメモ ―― Keem Bay, Thunder Bay, Meteor Lake"
date: 2022-01-11T12:13:25+09:00
draft: false
# tags: [ "Keem_Bay", "Thunder_Bay", "Meteor_Lake" ]
tags: [ "Meteor_Lake" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "VPU", "Note" ]
noindex: false
# summary: ""
---

Intel Movidius VPU (Vision Processing Unit) についてのメモ。  
Movidius社が開発する 128-bit の SIMD-VLIWプロセッサ、SHAVE (Streaming Hybrid Architecture Vector Engine) と、Intel に Movidius社が買収するまでの流れは以下の記事が詳しい。  

 * [ASCII.jp：マルチメディア向けからAI向けに大変貌を遂げたMovidiusのMyriad 2　AIプロセッサーの昨今 (1/3)](https://ascii.jp/elem/000/004/015/4015611/)

{{< pindex >}}
 * [Myriad X](#myriad-x)
 * [Keem Bay](#kmb)
    * [Keem Bay SKU](#kmb-sku)
 * [Thunder Bay](#thb)
    * [Thunder Bay Full / Prime (Standard)](#thb-variant)
 * [Meteor Lake](#mtl)
    * [GNA](#gna)
{{< /pindex >}}

## Myriad X {#myriad-x}
Intel の Movidius社買収から約 1年後に、*Myriad X* が発表された。  
前世代にあたる *Myriad 2* からの改良点としては、SHAVEプロセッサの増量 (12基 -\> 16基)、プロセスの更新 (TSMC 28nm HPM -\> TSMC 16nm FFC)、オンチップメモリ CMX (Connection MatriX) の拡大 (2MB, 400GB/s -\> 2.5MB, 450GB/s)、LPDDR4メモリのサポートが挙げられる。  
また、新たに Neural Compute Engine を搭載し、推論処理において 1TOPS の演算性能を発揮するとしている。  
RISC系CPU (LEON4 SPARC V8) 2-Core を搭載しており、それらは RTOS (Real Time Operating System) のスケジューリング、パイプライン管理、センサー操作を担当している。  
ハードウェアエンコーダも搭載し、H.264/H.265 4K 30Hz、MPEG/JPEG 4K 60Hz に対応する。インターフェイスに USB 3.1、PCIe Gen3 も備える。  

*Myriad X* は LPDDR3/LPDDR4 32-bit インターフェイスを持つ MA2485 と、外部メモリへのインターフェイスのみを持つ MA2085 の 2種類が発表された。  
MA2485 は 2020Q2 に生産中止、現在は MA2085 のみが提供されている。  

## Keem Bay {#kmb}
コードネーム *Keem Bay (KMB)* は、Gen 3 Intel Movidius VPU として 2019/11/15 に Intel AI Summit で発表された。  
CPU に Arm Cortex-A53 4-Core を搭載し、これによりスタンドアローンで Linuxシステムを動作させることができるようになった。  
4-15W という範囲の電力で動作し、M.2カードや複数の *Keem Bay* を搭載する PCIeカードといったフォームファクタに対応する。  
推論性能において、NVIDIA Jetson TX2 と比較では、電力効率は 6.2倍、面積あたりの性能は 8.7倍優れているとされる。  
前世代 *Myriad X* との比較では 10倍の推論性能を持つとしている。  

ただ競合製品と比較したときの性能効率は公開されているが、VPU部や Neural Compute Engine、オンチップメモリ CMX の規模、*Myriad X* からの改良点は公開されていない。  
また Intel AI Summit 2019 では 2020年の前半に発売予定としていたが、具体的な SKU について Intel は未だ発表していない。製造プロセスも同様。  

 * [The Cutting Edge of AI… Everything Matters! - IT Peer Network](https://itpeernetwork.intel.com/ai-summit-2019/)
 * [AI の最先端… すべてが重要！ | XLsoft エクセルソフト](https://www.xlsoft.com/jp/products/intel/openvino/document/ai-summit-2019.html)

### Keem Bay SKU {#kmb-sku}
しかし検索エンジンに頼ったところ、一部の顧客は *Keem Bay* の SKU と採用製品について発表していた。  
それによると、*Keem Bay* は TSMC 12nmプロセス (12FFC?) で製造され、SKU には **3400VE/3700VE/3500SE** の 3種類が存在する。  
SKU間の違いは主に VPU 数と動作クロック、それと DPU (Deep-Learning Processor Unit?, = Neural Compute Engine?) の数となる。  
推論性能は、*Myriad X* 1-Chip を 0.7 TOPS とし、そして **3700VE** は 7.1 TOPS 持つため、ここで上記の 10倍が達成されている。  
メモリインターフェイスは 2x32-bit LPDDR4/X 1600-2133 MHz に対応し、*Myriad X* から倍以上のメモリ帯域を実現可能になっている。  
SHAVEプロセッサは 12基か 16基となり、最大 16基という点では *Myriad X* と同数。DPU とメモリインターフェイスの強化がメインとなっているのだろうか。  
外部インターフェイスも強化されており、PCIe Gen4 x2 と eMMC に対応する。  

 * [Keem-Bay-One-pager_Draft_NA_External_Feb-26.pdf](https://www.arrow.com/ais/intel/wp-content/uploads/sites/6/2021/04/Keem-Bay-One-pager_Draft_NA_External_Feb-26.pdf)
 * [PowerPoint Presentation - DefenceLine-webinar_20200625.pdf](https://up-board.org/wp-content/uploads/2020/06/DefenceLine-webinar_20200625.pdf?mc_cid=800de085f9&mc_eid=0cd2fc3ef1)

| Intel Keem Bay | 3400VE | 3700VE | 3500SE |
| :-- | :--: | :--: | :--: |
| Process | TSMC 12nm | TSMC 12nm | TSMC 12nm |
| VPU clock | 500 MHz | 700 MHz | 700 MHz |
| Max TOPS (AI inference) | 5.1/3.0 TOPS | 7.1 TOPS | 4.3 TOPS |
| DPU count | 20 (5x4) / 12 (3x4) | 20 (5x4) | 12 (3x4) |
| SHAVE processors count | 16 / 12 | 12 | 16 | 12 |
| CPU | Arm A53 4-Core (1.5 GHz) | Arm A53 4-Core (1.5 GHz) | Arm A53 4-Core (1.5 GHz) |
| Memory | 2x 32-bit LPDDR4/X 1600-2133 MHz | 2x 32-bit LPDDR4/X 1600-2133 MHz | 2x 32-bit LPDDR4/X 1600-2133 MHz |

Linux Kernel へのパッチから得られる情報では、*Keem Bay* は OCS (Offload and Crypto Subsystem) を持ち、楕円曲線暗号 (Elliptic Curve Cryptography, ECC) 処理を専用ハードウェアで実行できる。  
専用ハードウェアは ECDH-256, ECDH-384 アルゴリズムに対応。[^kmb-ocs]  

[^kmb-ocs]: [[PATCH 0/5] Keem Bay OCS ECC crypto driver](https://lore.kernel.org/linux-devicetree/20211020103538.360614-1-daniele.alessandrelli@linux.intel.com/T/#m0a328ead665886f48ae0c60a79d694662042468a)

## Thunder Bay {#thb}
まだ公式発表はされていないが、Intel Movidius VPU系列には *Thunder Bay (THB)* なる SoC も存在し、Linux Kernel にパッチが投稿されている。  
パッチでは、*Thunder Bay* のスペックとバリアント間の違いにも触れている。  

 * [[PATCH v3 0/2] Add pinctrl support for Intel Thunder Bay SoC](https://lore.kernel.org/linux-gpio/20211216150100.21171-1-lakshmi.sowjanya.d@intel.com/T/#u)
 * [[PATCH V2 0/3] Add initial Thunder Bay SoC / Board support](https://lore.kernel.org/linux-devicetree/1626758569-27176-1-git-send-email-kenchappa.demakkanavar@intel.com/T/#u)

### Thunder Bay Full / Prime (Standard) {#thb-variant}

*Thunder Bay* は CPUクラスタ 4基を持ち、クラスタごとに *Keem Bay* と同じく Arm Cortex-A53 4-Core を持つ。  
バリアントには Full と Prime (Standard) と呼ぶ 2種類が存在し、CPUクラスタ 4基、クラスタあたり Arm A53 4-Core というのは共通。  
VPUスライスの数と搭載メモリが異なり、Full は VPUスライス 4基を持ち、メモリは 2x8GB + 2x4GB という構成。Prime (Standard) は VPUスライス 2基、メモリは 8GB + 4GB とされている。  
VPUスライスについては、複数の SHAVEプロセッサと Neural Compute Engine をまとめたものと推察される。この場合、*Keem Bay* は VPUスライス 1基で固定ということになる。  

 > 		This patch-set adds initial support for a new Intel Movidius SoC
 > 		code-named Thunder Bay. The SoC couples an ARM Cortex A53 CPU
 > 		with an Intel Movidius VPU.
 > 		
 > 		This initial patch-set enables only the minimal set of components
 > 		required to make the Thunder Bay full or prime configuration boards
 > 		boot into initramfs.
 > 		
 > 		Thunder Bay full configuration board has 4 clusters of 4 ARM
 > 		Cortex A53 CPUs per cluster, 4 VPU processors and
 > 		(8GB + 8GB + 4GB + 4GB) DDR memory.
 > 		
 > 		Thunder Bay prime configuration board has 4 clusters of 4 ARM
 > 		Cortex A53 CPUs per cluster, 2 VPU processors and
 > 		(8GB + 4GB) DDR memory.
 >
 > {{< quote >}} [[PATCH V2 0/3] Add initial Thunder Bay SoC / Board support](https://lore.kernel.org/linux-devicetree/1626758569-27176-1-git-send-email-kenchappa.demakkanavar@intel.com/T/#u) {{< /quote >}}

 > 		Add initial device tree for Intel Movidius SoC code-named Thunder Bay.
 > 		
 > 		This initial DT includes nodes for 4 CPU clusters with 4 Cortex-A53
 > 		cores per cluster, UARTs, GIC, ARM Timer and PSCI.
 > 		
 > 		thunderbay-soc.dtsi   - Thunder Bay SoC dtsi file
 > 		hddl_hybrid_4s.dts    - Thunder Bay full configuration board dts
 > 					with 4 VPU processors
 > 		hddl_hybrid_2s_02.dts - Thunder Bay prime configuration board dts with
 > 					2 VPU processors (slice 0 and slice 2 enabled)
 > 		hddl_hybrid_2s_03.dts - Thunder Bay prime configuration board dts with
 > 					2 VPU processors (slice 0 and slice 3 enabled)
 > 		hddl_hybrid_2s_12.dts - Thunder Bay prime configuration board dts with
 > 					2 VPU processors (slice 1 and slice 2 enabled)
 > 		hddl_hybrid_2s_13.dts - Thunder Bay prime configuration board dts with
 > 					2 VPU processors (slice 1 and slice 3 enabled)
 >
 > {{< quote >}} [[PATCH V2 0/3] Add initial Thunder Bay SoC / Board support](https://lore.kernel.org/linux-devicetree/1626758569-27176-1-git-send-email-kenchappa.demakkanavar@intel.com/T/#e5e8fba959b11f1cc80d9c261a78a0285fa86f342) {{< /quote >}}

*Thunder Bay* は *Keem Bay* の複数並べたような構成と見えるが、VPUスライスあたりの規模は不明。  
しかし *Keem Bay* よりも全体の規模は大きくなっているのは確かであり、15Wより上か、Arm A53 16-Core からサーバ向けの製品を想定しているのではないかと思われる。  

[Intel® Movidius™ Accelerator on Edge Software Hub](https://www.intel.com/content/www/us/en/developer/articles/technical/movidius-accelerator-on-edge-software-hub.html) には、Gen 3 Intel Movidius VPU として、*Keem Bay* ともう 1つ、*Shamrock Bay* を挙げられているが、*Thunder Bay* の名はない。  
*Shamrock Bay* の名だけが変更されて *Thunder Bay* になったのか、あるいはキャンセルされて *Thunder Bay* のプロジェクトが立ち上げられたのか。  
*Shamrock Bay* はそのページで触れられているのみであるため、どちらにしても実質 *Thunder Bay* に置き換えられたものと思われる。  

## Meteor Lake {#mtl}
オープンソースライセンスの元に公開されている [Intel In-Band Manageability Framework (INBM)](https://github.com/intel/intel-inb-manageability) 等では、VPU Platform/Device List に *Meteor Lake* がすでにリストされており、*Meteor Lake* の世代で VPU を統合するものと思われる。  
*Meteor Lake* は複数の Tile (CPU [Compute], SoC, GPU) で構成されることが公式に発表されている。VPU を統合するとしたら SoC Tile になるだろうか。  

 > 		    """Check the platform type based on b7 to b4 in sw device id.
 > 		       0000b – KMB device
 > 		       0001b – THB Prime device with 2 compute slices
 > 		       0010b – THB Standard device with 4 compute slices
 > 		       0011b – OYB device
 > 		       0100b – MTL device
 > 		       0101b – STF device
 > 		       Rest of the values – reserved for future or not used
 > 		    @param sw_device_id: unique sw device id obtained from xlink
 > 		    @return: platform type
 > 		    """
 >
 > {{< quote >}} [intel-inb-manageability/ixlink_wrapper.py at 48d7af6f49e2bb5faf0e6dd7754fb55eb68dd5f0 · intel/intel-inb-manageability](https://github.com/intel/intel-inb-manageability/blob/48d7af6f49e2bb5faf0e6dd7754fb55eb68dd5f0/inbm-lib/inbm_vision_lib/xlink/ixlink_wrapper.py#L328-L340) {{< /quote >}}

 > 		enum class VPUXPlatform: int {
 > 		    AUTO            = 0,    // auto detection
 > 		    MA2490          = 1,    // Keem bay A0
 > 		    MA2490_B0       = 2,    // Keem bay B0
 > 		    MA3100          = 3,    // Thunder bay harbor A0
 > 		    MA3720          = 4,    // Meteor lake
 > 		};
 >
 > {{< quote >}} [openvino_sample/vpux_plugin_config.hpp at 55f4011f326211e0d9efc15a31e403272acc59e7 · YaoxinShi/openvino_sample](https://github.com/YaoxinShi/openvino_sample/blob/55f4011f326211e0d9efc15a31e403272acc59e7/openvino/inference_engine/include/vpux/vpux_plugin_config.hpp#L60-L70) {{< /quote >}}

### GNA {#gna}

推論処理向けアクセラレータとしては、Atom系では *Gemini Lake*、Core系では *Cannon Lake* から GNA (Gaussian & Neural Accelerator) が搭載されている。  

OpenVINO のドキュメントでは、GNA が 推論処理において CPU, GPU, VPU を置き換えるものではなく、ノイズ除去や音声認識といった推論処理 (continuous inference workloads) をオフロードし、CPU やメモリに負荷を掛けることなく低消費電力で処理することを目的に設計されたものだとしている。[^openvino-gna]  
OpenVINO は、Intel がオープンソースで開発する AI推論ツールキット。CPU/GPU/VPU/GNA といった複数のデバイスに対応している。  
VPU は Vision Processing Unit の名の通り、画像認識等に特化している。  
GNA、VPU ともに、それぞれ音声処理、画像処理に限定されてはいないが、向いた処理は異なる。  
*Meteor Lake* ではより推論処理を高速かつ低消費電力に処理するために、GNA と VPU を両方搭載する可能性もある。  

*Meteor Lake* の他に、*OYB*、*STF* といったプラットフォームもリストされているが、何の略称かは明かされておらず、将来登場するであろう VPU としか言えない。  

[^openvino-gna]: [GNA Plugin — OpenVINO™ documentation](https://docs.openvino.ai/latest/openvino_docs_IE_DG_supported_plugins_GNA.html)

{{< ref >}}
 * [Intel® Movidius™ Accelerator on Edge Software Hub](https://www.intel.com/content/www/us/en/developer/articles/technical/movidius-accelerator-on-edge-software-hub.html)
 * [Movidius - Wikipedia](https://en.wikipedia.org/wiki/Movidius)
 * Myriad 2
    * [Hot Chips 26 - Movidius、「Myriad 2」ビジョンプロセサを発表 | TECH+](https://news.mynavi.jp/techplus/article/20140905-hotchips26_myraid2/)
    * [PowerPoint Presentation - HC28.23.811-Deep-Neural-Moloney-Movidius_16x9.pdf](https://old.hotchips.org/wp-content/uploads/hc_archives/hc28/HC28.23-Tuesday-Epub/HC28.23.80-Big-Data-Epub/HC28.23.811-Deep-Neural-Moloney-Movidius_16x9.pdf)
 * Myriad X
    * [Intel Announces Movidius Myriad X VPU, Featuring ‘Neural Compute Engine’](https://www.anandtech.com/show/11771/intel-announces-movidius-myriad-x-vpu)
    * [Intel Unveils Neural Compute Engine in Movidius Myriad X VPU to Unleash AI at the Edge | Intel Newsroom](https://newsroom.intel.com/news/intel-unveils-neural-compute-engine-movidius-myriad-x-vpu-unleash-ai-edge/#gs.lzzj2p)
    * [Intel_ProductOverview_Neural_Compute_Stick_2.pdf](https://www.mouser.com/catalog/additional/Intel_ProductOverview_Neural_Compute_Stick_2.pdf)
    * {{< youtube id="oYQwRdDGqzw" title="Introducing Movidius Myriad-X | OpenVINO™ toolkit | Ep. 27 | Intel Software" >}}
 * Keem Bay
    * [PowerPoint Presentation - intel-ai-summit-keynote-slides.pdf](https://newsroom.intel.com/wp-content/uploads/sites/11/2019/11/intel-ai-summit-keynote-slides.pdf)
 * GNA
    * [Intel® GMM and Neural Network Accelerator - 003 - ID:655258 | Core™ Processors](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/client/platforms/alder-lake-desktop/12th-generation-intel-core-processors-datasheet-volume-1-of-2/003/intel-gmm-and-neural-network-accelerator/)
    * [GNA Plugin - OpenVINO™ Toolkit](https://docs.openvino.ai/2020.4/openvino_docs_IE_DG_supported_plugins_GNA.html)
    * [[PATCH v3 06/14] intel_gna: add hardware ids - Maciej Kwapulinski](https://lore.kernel.org/lkml/20210513110040.2268-7-maciej.kwapulinski@linux.intel.com/T)
{{< /ref >}}
