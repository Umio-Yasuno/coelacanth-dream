---
title: "Linux Kernel に 「Aldebaran」 GPU をサポートするパッチが投稿される　―― CDNA 2/MI200 のコードネーム?"
date: 2021-02-25T13:40:22+09:00
draft: false
tags: [ "MI200", "CDNA_2", "gfx90a" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

新たな AMD GPU、*Aldebaran* をサポートするためのパッチ 159個が Linux Kernel (amd-gfx) に投稿された。  

 * [[PATCH 000/159] Aldebaran support](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059694.html)

*Aldebaran* は *Arcturus/MI100/CDNA 1* 同様に GFX Ring を持たない。グラフィクス処理を行うための固定機能 {{< comple >}} Rasterizer, RenderBackend, Geometry Engine ... {{< /comple >}} も搭載していないと思われる。  


 >        -	if (adev->asic_type == CHIP_ARCTURUS)
 >        +	if (adev->asic_type == CHIP_ARCTURUS ||
 >        +	    adev->asic_type == CHIP_ALDEBARAN)
 >         		adev->gfx.num_gfx_rings = 0;
 >         	else
 >
 > {{< quote >}} [[PATCH 017/159] drm/amdgpu: add gfx v9 block support for aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059701.html) {{< /quote >}}

そしてその特徴等から、*CDNA 2アーキテクチャ* を採用する **MI200** のコードネームではないかと考えられる。  
**MI200** に関連するパッチは最近になって投稿され始め、先日には対応すると思われる GPUID *gfx90a* のサポートが LLVM に追加されている。  
{{< link >}} [LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か | Coelacanth's Dream](/posts/2021/02/19/llvm-gfx90a/) {{< /link >}}

{{< pindex >}}

 * [HBM2Eメモリをサポート](#hbm2e)
 * [少ない SDMAエンジン](#sdma)
 * [Aldebaran では GDS が削除](#removed-gds)
 * [CPU との XGMI/Infinity Fabric Link 接続に対応](#xgmi-to-cpu)
 * [Aldebaran は複数ダイ構成 (MCM) か](#multi-die)
 * [SMUv13、VCN 2.6、DeviceID](#other)
 * [コードネーム: Aldebaran](#codename)

{{< /pindex >}}

## HBM2Eメモリをサポート {#hbm2e}

同時に HBM2Eメモリのサポートが追加されており、*Aldebaran/MI200* は HBM2Eメモリを採用すると見られる。  

 * [[PATCH 070/159] drm/amdgpu: support get_vram_info atomfirmware i/f for aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059756.html)
 * [[PATCH 071/159] drm/amdgpu: correct vram_info for HBM2E](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059757.html)
 * [[PATCH 097/159] drm/amdpgu: add ATOM_DGPU_VRAM_TYPE_HBM2E vram type](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059783.html)

HBM2メモリではスタックあたり最大 8GB までだったが、HBM2Eメモリではスタックあたり 16GB という容量をサポートしており、*LUMI* システムの開発者の発言から、少なくとも 40GB以上のメモリを持つ *Aldebaran/MI200* においては HBM2Eメモリの採用は必定だった。[^lumi-vram]  

[^lumi-vram]: [The aftermath of the LUMI end user webinar - The aftermath of the LUMI end user webinar - CSC Company Site](https://www.csc.fi/en/-/the-aftermath-of-the-lumi-end-user-webinar)

## 少ない SDMAエンジン {#sdma}

*Aldebaran* は次世代 *CDNA GPU* と考えているが、PCIe や *XGMI/Infinity Fabric* 等のバスを経由してデータをコピー、転送する際に用いられる SDMAエンジン/コントローラーの数は *Arcturus* よりも少なくなっている。  
*Arcturus* は SDMAエンジン 8基を持ち、内 2基は CPU とのデータのやり取りに使われ、残り 6基は *XGMI/Infinity Fabric* に最適化された GPU間通信用のものとなる。  
対し *Aldebaran* は、CPU用に 2基というのは変わらないが、GPU間用の SDMAエンジンは 3基に減っている。  


 >        static const struct kfd_device_info arcturus_device_info = {
 >        	.asic_family = CHIP_ARCTURUS,
 >        	.asic_name = "arcturus",
 >        	.max_pasid_bits = 16,
 >        	.max_no_of_hqd	= 24,
 >        	.doorbell_size	= 8,
 >        	.ih_ring_entry_size = 8 * sizeof(uint32_t),
 >        	.event_interrupt_class = &event_interrupt_class_v9,
 >        	.num_of_watch_points = 4,
 >        	.mqd_size_aligned = MQD_SIZE_ALIGNED,
 >        	.supports_cwsr = true,
 >        	.needs_iommu_device = false,
 >        	.needs_pci_atomics = false,
 >        	.num_sdma_engines = 2,
 >        	.num_xgmi_sdma_engines = 6,
 >        	.num_sdma_queues_per_engine = 8,
 >        };
 >        
 >        static const struct kfd_device_info aldebaran_device_info = {
 >        	.asic_family = CHIP_ALDEBARAN,
 >        	.asic_name = "aldebaran",
 >        	.max_pasid_bits = 16,
 >        	.max_no_of_hqd	= 24,
 >        	.doorbell_size	= 8,
 >        	.ih_ring_entry_size = 8 * sizeof(uint32_t),
 >        	.event_interrupt_class = &event_interrupt_class_v9,
 >        	.num_of_watch_points = 4,
 >        	.mqd_size_aligned = MQD_SIZE_ALIGNED,
 >        	.supports_cwsr = true,
 >        	.needs_iommu_device = false,
 >        	.needs_pci_atomics = false,
 >        	.num_sdma_engines = 2,
 >        	.num_xgmi_sdma_engines = 3,
 >        	.num_sdma_queues_per_engine = 8,
 >        };
 >
 > {{< quote >}} [drivers/gpu/drm/amd/amdkfd/kfd_device.c · 1b1515513cdcc6cfa86d7f4f8c1db899695f25f6 · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/blob/1b1515513cdcc6cfa86d7f4f8c1db899695f25f6/drivers/gpu/drm/amd/amdkfd/kfd_device.c#L379) {{< /quote >}}

*Arcturus* は GPU用の SDMAエンジンを 6基持っていたことで最大 8-GPU との接続が可能だったが[^arcturus-sdma]、*Aldebaran* は不可能か、出来たとしても遠くの GPU とやり取りする際に経由しなければならない GPU が多く、性能を出すことが難しいと考えられる。  
だが、*Arcturus/MI100* は製品としては最大 4-GPU での相互接続に対応し、8-GPU のトポロジ構成には対応しなかったことや、  
*Aldebaran/MI200* を採用することが決まっているスパコンは、現状どれもノード構成が 1-CPU:4-GPU となっていることを考えると、そこまで SDMAエンジン数を必要としないから減らしたとも考えられる。  
{{< link >}} [MI200 は Frontier、LUMI に採用され、LUMI は 2021年半ばから運用開始予定 | Coelacanth's Dream](/posts/2021/02/22/mi200-hpc-system/) {{< /link >}}

[^arcturus-sdma]: [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/)
[^mi100-links]: [AMD、初の CDNAアーキテクチャ GPU 「MI100」 を発表 | Coelacanth's Dream](/posts/2020/11/17/amd-cdna-arch-mi100-arcturus/)

コード中には *Aldebaran* が最大 8-GPU に対応するように記述されているが、今の所はサポートのため (*Arcturus* の) 設定を使い回しているだけで、後で書き直す必要があるとしている。  

 >        	switch (adev->asic_type) {
 >        	case CHIP_VEGA20:
 >        		max_num_physical_nodes   = 4;
 >        		max_physical_node_id     = 3;
 >        		break;
 >        	case CHIP_ARCTURUS:
 >        		max_num_physical_nodes   = 8;
 >        		max_physical_node_id     = 7;
 >        		break;
 >        	case CHIP_ALDEBARAN:
 >        		/* just using duplicates for Aldebaran support, revisit later */
 >        		max_num_physical_nodes   = 8;
 >        		max_physical_node_id     = 7;
 >        		break;
 >
 > {{< quote >}} [drivers/gpu/drm/amd/amdgpu/gfxhub_v1_1.c · amd-staging-drm-next-aldebaran · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/blob/amd-staging-drm-next-aldebaran/drivers/gpu/drm/amd/amdgpu/gfxhub_v1_1.c) {{< /quote >}}

## Aldebaran では GDS が削除 {#removed-gds}

AMD GPUアーキテクチャでは、全ての CU がアクセス可能なオンチップメモリ、GDS (Global Data Share) を持っているが、*Aldebaran* では GDS が削除されている。  
GDS はスレッド間でのデータ共有やソフトウェアが用いるキャッシュ等の用途がある。  

 * [[PATCH 072/159] drm/amdgpu: init gds for aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059758.html)

GDS を削除した理由としてはアトミック操作/使用のためとしている。  
これだけではよく分からないが、1-GPU に搭載される CU が増えるにつれ、GDS ではなく増やした L2キャッシュを利用した方が良い、となったのかもしれない。  
それと、追加された実行フローにおける新モード *TgSplit* の存在が関係している可能性もある。  

そして今になって調べて気付いたが、GDS の変更は *Arcturus/MI100* の時点で行われていた。  
*Vega10/Vega20* は GDS 64KB を持っていたが、*Arcturus/MI100* では 4KB (4バンク) に減らされている。[^arct-gds]帯域やバンク数は資料に記述されなくなった。[^gds-size]
一応、*RDNA 2/GFX10.3* も確認したが、そちらは 64KB のままであり、*CDNA アーキテクチャ* 特有の変更点と言える。  

[^gds-size]: [Vega_7nm_Shader_ISA_26November2019.pdf](https://gpuopen.com/wp-content/uploads/2019/11/Vega_7nm_Shader_ISA_26November2019.pdf) ["AMD Instinct MI100" Instruction Set Architecture: Reference Guide - CDNA1_Shader_ISA_14December2020.pdf](https://developer.amd.com/wp-content/resources/CDNA1_Shader_ISA_14December2020.pdf)
[^arct-gds]: [[PATCH 089/102] drm/amdgpu: init gds config for arct](https://lists.freedesktop.org/archives/amd-gfx/2019-July/036836.html)


## CPU との XGMI/Infinity Fabric Link 接続に対応 {#xgmi-to-cpu}

*Aldebaran* には CPU との *XGMI/Infinity Fabric Link* 接続に対応し、これは以前 *3rd Gen Infinity Architecture* として発表されていたものと一致する。  
*3rd Gen Infinity Arcitecture* では、GPU メモリを CPU側のシステムアドレスにマッピングすることで CPU と GPU のコヒーレントを保ち、GPGPUプログラムを複雑にしていたメモリ空間の違いとそれによるデータ転送の手間を無くし、より広い用途での GPU の活用を可能とする。  
{{< link >}} [Linux Kernel に CPU + GPU の統合メモリ空間をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/07/add-svm-to-amdgpu-kfd/) {{< /link >}}

 * [[PATCH 040/159] drm/amdgpu: define address map for host xgmi link (v3)](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059726.html)
 * [[PATCH 042/159] drm/amdkfd: expose host gpu link via sysfs (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059728.html)
 * [[PATCH 048/159] drm/amdgpu: new cache coherence change for Aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059733.html)

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_10.webp" title="AMD 3rd Gen Infinity Architecture Enables Accelerated Computing" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - Mark Papermaster: Future of High Performance](https://ir.amd.com/static-files/ccef22f0-f641-4fc5-861f-cb3d7d125a68)" >}}

## Aldebaran は複数ダイ構成 (MCM) か {#multi-die}

先日、HPE の資料に、製品名 **AMD Instinct MI200 OAM x1 MCM Special FIO Accelerator for HPE Cray EX** があったことが話題となり[^mcm]、**MI200** は MCM 構成ではないかとされたが、それを裏付けるようなパッチとコメントがあった。  
{{< link >}} [Intel OneAPI - a50002555enw](https://assets.ext.hpe.com/is/content/hpedam/a50002555enw) {{< /link >}}

*Aldebaran* はに複数の die ID が割り振られ、GPU と CPU (ホスト) とのリンクタイプを得るのに使われ、  

 >        is_host_gpu_xgmi_supported is used to query gpu and
 >        cpu/host link type. get_die_id is used to query die
 >        ids.
 >
 > {{< quote >}} [[PATCH 036/159] drm/amdgpu: add new smuio callbacks for aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059719.html) {{< /quote >}}

新たなクロック調節機能である "Performance Determinism" はダイごとに有効可能だとしている。  

 >        Performance Determinism is a new mode in Aldebaran where PMFW tries to
 >        maintain sustained performance level. It can be enabled on a per-die
 >        basis on aldebaran.
 >
 > {{< quote >}} [[PATCH 124/159] drm/amd/pm: Enable performance determinism on aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059807.html) {{< /quote >}}

また、*Aldebran* の SMUv13 (System Management Unit) のファイルにはマクロ名に `MCM` が付くものがあり、それはダイ、パッケージ、ソケット、トポロジ等に関係している。  

 >        //SMUIO_MCM_CONFIG
 >        #define SMUIO_MCM_CONFIG__DIE_ID__SHIFT                                                                       0x0
 >        #define SMUIO_MCM_CONFIG__PKG_TYPE__SHIFT                                                                     0x1
 >        #define SMUIO_MCM_CONFIG__SOCKET_ID__SHIFT                                                                    0x4
 >        #define SMUIO_MCM_CONFIG__PKG_SUBTYPE__SHIFT                                                                  0x8
 >        #define SMUIO_MCM_CONFIG__TOPOLOGY_ID__SHIFT                                                                  0xa
 >        #define SMUIO_MCM_CONFIG__DIE_ID_MASK                                                                         0x00000001L
 >        #define SMUIO_MCM_CONFIG__PKG_TYPE_MASK                                                                       0x0000000EL
 >        #define SMUIO_MCM_CONFIG__SOCKET_ID_MASK                                                                      0x000000F0L
 >        #define SMUIO_MCM_CONFIG__PKG_SUBTYPE_MASK                                                                    0x00000300L
 >        #define SMUIO_MCM_CONFIG__TOPOLOGY_ID_MASK                                                                    0x00007C00L
 >
 > {{< quote >}} [drivers/gpu/drm/amd/include/asic_reg/smuio/smuio_13_0_2_offset.h · amd-staging-drm-next-aldebaran · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/blob/amd-staging-drm-next-aldebaran/drivers/gpu/drm/amd/include/asic_reg/smuio/smuio_13_0_2_offset.h) {{< /quote >}}

*Aldebaran/MI200* が MCM構成だとして、そこで出てくるのは、上述した SDMAエンジンの数がダイごとのものなのか、GPU全体なのか、ダイ同士の接続は SDMAエンジンを用いる *XGMI* 接続なのかそうでないか、それぞれが I/O やメディアエンジンを持つ独立可能な複数のダイで構成されるのか、といった疑問が出てくるが、  
まあそれは今後パッチという形か、あるいは正式発表時に明かされるだろうから、それを待つこととする。  


## SMUv13、VCN 2.6、DeviceID {#other}

既に触れたが、*Aldebran* の SMU は Version13 となり、これは以前ひょっこりと出てきてそれきりな [Yellow Carp APU](/tags/yellow_carp) と同じ世代となる。  
{{< link >}} [新たな AMD APU、"Yellow Carp"　―― SMU は v13 に | Coelacanth's Dream](/posts/2020/12/08/amd-yellow-carp-apu/) {{< /link >}}
さらに言えば、*MI200* が SMUv13 となるのは、以前ファームウェアバイナリに混じってアップロードされた更新履歴にあった情報だ。  
{{< link >}} [Cezanne の Coreboot 対応が進行中　―― MI200, Mero, Rembrandt のネームプレート | Coelacanth's Dream](/posts/2020/12/16/czn-coreboot-mi200-mero-rmb/) {{< /link>}}

しかし、対応するプラットフォームや時期の違いから、それぞれの SMUv13 は全く同じではなく、*Yellow Carp* は SMU v13.0.1、*Aldebaran* は SMU v13.0.2 となっている。  

デコード/エンコードを処理する VCN は *Arcturus* からわずかにバージョンが進んだ VCN 2.6 を搭載しているが、VCN 2.5 との違いは不明。  
また、*Arcturus* 同様に VCN を 2基搭載するが、片方は VCN 2.6、もう片方は VCN 2.5 としている。配置される MMHUB (MultiMedia Hub) は異なるが、やはりこちらも違いは不明。[^vcn-2_6]  
*Navi21/Sienna Cichlid* のようにエンコードとデコードで役割を分けているようでもない。  

[^vcn-2_6]: [[PATCH 044/159] drm/amdgpu/vcn2.6: Add vcn2.6 support](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059730.html)

*Aldebaran* の DeviceID (PCI ID) には現在 3種リストされているが、*Arcturus* の例から、2種はエンジニアサンプリングか予約用と思われる。  


 >        +	/* Aldebaran */
 >        +	{0x1002, 0x7408, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_ALDEBARAN},
 >        +	{0x1002, 0x740C, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_ALDEBARAN},
 >        +	{0x1002, 0x740F, PCI_ANY_ID, PCI_ANY_ID, 0, 0, CHIP_ALDEBARAN},
 >        +
 >
 > {{< quote >}} [[PATCH 067/159] drm/amdgpu: Add DID for aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059750.html) {{< /quote >}}

## コードネーム: Aldebaran {#codename}

*Aldebaran (アルデバラン)* はおうし座で最も明るい恒星 (首星)、α星の固有名である。  
*RDNA 2 アーキテクチャ* からは **色+魚** をコードネームにするようになったが、*CDNA アーキテクチャ* では引き続き恒星をコードネームに採用するようだ。  

*Vega (ベガ)* 、*Arcturus (アークトゥルス)* もまた α星の固有名で、*Vega* はこと座、*Arcturus* はうしかい座に位置する。  
星には詳しくない上、参考資料によって異なったりであやふやだが、地球から *Vega* は 25光年、*Arcturus* は 36光年、*Aldebaran* は 65光年離れており、最新の世代ほど地球から遠くなっている。  

ランダムで選ばれている可能性もあるが、*Arcturus* は大きな固有運動を示す高速度星でもあり、*GCNアーキテクチャ* から *CDNA アーキテクチャ* への転換を象徴する GPU のコードネームとしてはぴったりと言える。  
そして、*Aldebaran* はアラビア語で「後に続くもの」の意で、同じおうし座に位置するプレアデス (和名: すばる) よりも遅れて昇ってくることからその名が付けられている。  
こちらも、*Arcturus/CDNA 1* から続く第2世代 *CDNA アーキテクチャ* 、*CDNA 2* GPU に付けられたコードネームとして合っている。  


