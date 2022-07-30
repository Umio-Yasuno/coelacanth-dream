---
title: "Versatile Processing Unit を搭載する Intel Meteor Lake"
date: 2022-07-29T03:48:56+09:00
draft: false
categories: [ "Hardware", "Intel", "VPU" ]
tags: [ "Meteor_Lake", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Jacek Lawrynowicz 氏により、Intel クライアント向け第14世代プロセッサ *Meteor Lake* に搭載される *Versatile Processing Unit (VPU)* の DRM (Direct Rendering Manager) ドライバーを Linux Kernel に追加するパッチが dri-dev メーリングリストに投稿されている。  

 * [[PATCH v1 0/7] New DRM driver for Intel VPU - Jacek Lawrynowicz](https://lore.kernel.org/dri-devel/20220728131709.1087188-1-jacek.lawrynowicz@linux.intel.com/)

Jacek Lawrynowicz 氏は VPU の動作は内蔵 GPU に近いとしており、ドライバーを `gpu/drm` サブシステムの一部としている。DRMドライバーの初期化処理、IOCTL ハンドリング、メモリ管理機能を再利用することで、Linux Kernel 内のコードの重複を抑えることができる。  
User Mode Driver (UMD) は Level Zero API ドライバーと OpenVINOプラグインで構成される予定であり、両方とも 2022Q3 の終わりまでにはオープンソース化されるとしている。  
VPU のファームウェアはクローズドソースなバイナリブロブとして配布される。  

## VPU {#vpu}
*Versatile Processing Unit (VPU)* は CPU/SoC に内蔵された、Computer Vision やディープラーニングアプリケーション向けの AI推論処理アクセラレータとなる。  

*VPU* の名は、*Myriad X* や *Keem Bay*、*Thunder Bay* にも使われているが、それらにおける *VPU* は *Vision Processing Unit* の略とされている。  
しかしパッチから *Meteor Lake* に搭載される *VPU* も、実行エンジンは SHAVE (Streaming Hybrid Architecture Vector Engine) と思われ、意味合いとしては従来の *VPU* からそれほど変わらないように思う。  
*Meteor Lake VPU* の IPバージョンは v2.7 とされ、今回のパッチは VPU IP v2.7 をサポート対象としている。  

 > 		+#define PCI_VENDOR_ID_INTEL 0x8086
 > 		+#define PCI_DEVICE_ID_MTL   0x7d1d
 > 		+
 > 		+#define VPU_GLOBAL_CONTEXT_MMU_SSID 0
 > 		+#define VPU_CONTEXT_LIMIT	    64
 > 		+#define VPU_NUM_ENGINES		    2
 >
 > {{< quote >}} [[PATCH v1 1/7] drm/vpu: Introduce a new DRM driver for Intel VPU - Jacek Lawrynowicz](https://lore.kernel.org/dri-devel/20220728131709.1087188-2-jacek.lawrynowicz@linux.intel.com/) {{< /quote >}}

また、*Meteor Lake VPU* の DeviceID `0x7D1D` は [openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin) だと `VPUX37XX (VPU3720)` として管理されている。  

[openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin) では、*VPU* を `VPUNN::VPUDevice::*, InferenceEngine::VPUXConfigParams::VPUXPlatform::*, VPU::ArchKind::*` でそれぞれ管理しており、*Keem Bay* は `VPUXPlatform::{VPU3400, VPU3700}` と `ArchKind::VPUX30XX`、*Thunder Bay* は `VPUXPlatform::{VPU3800, VPU3900}` と `ArchKind::VPUX311X` され、IPバージョンに相当する `VPUDevice` は両者 `VPU_2_0` とされている。[^vpudevice] [^archkind]  
*Meteor Lake VPU* は *Keem Bay* と *Thunder Bay* から世代が進んだアーキテクチャとなる。  

[^device-helper]: [vpux-plugin/device_helpers.cpp at 411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7 · openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin/blob/411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7/src/vpux_al/src/device_helpers.cpp)
[^vpudevice]: [vpux-plugin/dpu_tiler.cpp at 411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7 · openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin/blob/411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7/src/vpux_compiler/src/dialect/VPUIP/dpu_tiler.cpp#L90-L100)
[^archkind]: [vpux-plugin/compiler.cpp at 411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7 · openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin/blob/411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7/src/vpux_compiler/src/compiler.cpp#L55-L71)

 > 		//    KMD is setting usDeviceID from VpuFamilyID.h
 > 		#define IVPU_MYRIADX_DEVICE_ID 0x6200   // MyriadX device
 > 		#define IVPU_KEEMBAY_DEVICE_ID 0x6240   // KeemBay device
 > 		#define IVPU_VPUX37XX_DEVICE_ID 0x7D1D  // VPUX37XX device
 > 		#define IVPU_VPUX4000_DEVICE_ID 0x643E  // VPUX4000 device
 > 		
 > 		    std::string name;
 > 		    switch (properties.deviceId) {
 > 		    case IVPU_MYRIADX_DEVICE_ID:
 > 		        name = "2700";
 > 		        break;
 > 		    case IVPU_KEEMBAY_DEVICE_ID:
 > 		        name = "3700";
 > 		        break;
 > 		    case IVPU_VPUX37XX_DEVICE_ID:
 > 		        name = "3720";
 > 		        break;
 >
 > {{< quote >}} [vpux-plugin/zero_device.cpp at 411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7 · openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin/blob/411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7/src/zero_backend/src/zero_device.cpp#L36-L52) {{< /quote >}}

*Meteor Lake VPU* では *Myriad X* と同様にマイクロコントローラに RISC系 CPU を採用している。*Myriad X* では LEON4 SPARC V8 2-Core を採用しており、それらが RTOS のスケジューリングやパイプライン管理、センサー操作を担当していた。  
Memory Management Unit (MMU) は Arm MMU-600 ベースとしている。  
Neural Compute Subsystem (NCS) には実際の処理を行うコンピュートエンジン、コピーエンジンが位置する。  
*Keem Bay* では DPU (Data Processing Unit) を最大 20基搭載しており、*Meteor Lake VPU* でも引き続き搭載するものと思われる。DPU について詳細は公開されていないが、*Keem Bay* 採用製品の情報からニューラルネットワークのスパースや圧縮処理を担当している。  

[openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin) によれば、*Meteor Lake VPU* では新たに BF16フォーマットや混合精度をサポートしている。  
*Keem Bay* からではあるが、画像認識以外に推論処理も主なターゲットとなり、*Meteor Lake VPU* では推論処理に向けた機能が追加されている。そのため *Vison Processing Unit* から *Versatile Processing Unit* (Versatile, 多才な、多用途) へと名前を変えたのかもしれない。[^bf16] [^mix]  

[^mix]: [vpux-plugin/nce_sparsity.cpp at 411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7 · openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin/blob/411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7/src/vpux_compiler/src/dialect/VPU/nce_sparsity.cpp#L22-L24)
[^bf16]: [vpux-plugin/nce_invariant.cpp at 411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7 · openvinotoolkit/vpux-plugin](https://github.com/openvinotoolkit/vpux-plugin/blob/411d6bb1e1ff6daa146954b75b149fdc7c1bdcd7/src/vpux_compiler/src/dialect/VPU/nce_invariant.cpp#L23-L31)

{{< ref >}}
 * [Intel Movidius VPU に関するメモ ―― Keem Bay, Thunder Bay, Meteor Lake | Coelacanth's Dream](/posts/2022/01/11/intel-kmb-thb/)
 * [OAK Series 3 — DepthAI Hardware Documentation 1.0.0 documentation](https://docs.luxonis.com/projects/hardware/en/latest/pages/articles/oak-s3.html)
{{< /ref >}}
