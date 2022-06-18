---
title: "Intel OSS OpenGL/Vulkan ドライバーが \"Compute Engine\" に対応"
date: 2022-06-16T13:22:53+09:00
draft: false
categories: [ "Hardware", "Software", "Intel", "GPU" ]
tags: [ "Gen12", "Xe-HP", "Xe-HPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Mesa3D における Intel Iris (OpenGL) / ANV (Vulkan) ドライバーに *Compute Engine* の対応を追加するマージリクエストが、Intel の Jordan Justen 氏より投稿された。当該マージリクエストはすでに `main` ブランチにマージされている。  

 * [Use INTEL_COMPUTE_CLASS=1 to enable using a compute engine (!14395) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14395)

機能は現状デフォルトでは無効化されており、環境変数 `INTEL_COMPUTE_CLASS` に `[1, true, y, yes]` のどれかを設定することで有効化できる。  
有効化され、かつ *Compute Engine* が検出された場合、Compute dispatch (OpenGL) と Compute queue (Vulkan) において *Compute Engine* を用いるようになる。  

 > 		+:envvar:`INTEL_COMPUTE_CLASS`
 > 		+   If set to 1, true or yes, then I915_ENGINE_CLASS_COMPUTE will be
 > 		+   supported. For OpenGL, iris will attempt to use a compute engine
 > 		+   for compute dispatches if one is detected. For Vulkan, anvil will
 > 		+   advertise support for a compute queue if a compute engine is
 > 		+   detected.
 >
 > {{< quote >}} [anv, iris: Enable compute engine with INTEL_COMPUTE_CLASS=1 (81d6ae31) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/81d6ae31d6f18d6fd2894a8b6dfe4323eea797f9) {{< /quote >}}

当初は環境変数 `INTEL_DEBUG` に `c-cs` を設定することで有効化される形だったが、色圧縮機能 `Color Control Surface` と略称が被り[^color-control-surface]、また `INTEL_DEBUG` にそれを無効化する `noccs` がすでに定義されていたため、`INTEL_COMPUTE_CLASS` を使う形となった。[^ref-comment-1]  

[^color-control-surface]: [Single-sampled Color Compression — The Mesa 3D Graphics Library latest documentation](https://docs.mesa3d.org/isl/ccs.html)
[^ref-comment-1]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14395#note_1366681>

## Command Streamer {#cs}
ここでの *Compute Engine* は実質 *Compute Command Streamer* を指していると思われる。  

Intel GPU において Command Streamer (CS) は各 Engine への主要なインターフェイスとして機能し、CPU から転送された命令のパース、パイプラインの管理と制御、データのフェッチ、EU へのディスパッチ等を担当している。  
CS の種類には、Render (RCS)、Copy/Blitter (BCS)、Video (VCS)、Video Enhancment (VECS) がある。  
*Gen11 アーキテクチャ* からカリング処理を行うための POSH (Position Only SHader) パイプライン専用の POCS、*Gen12/{{< xe >}} アーキテクチャ* から GPGPU/Compute 専用の CCS が追加された。  

### POSH, POCS {#posh}
POSH はカリング処理を目的とした新たなジオメトリパイプラインであり、Cull Pipe、Record Pipe とも呼ばれる。  
POSH では位置のみの (position only) Vertex Shader を先行して実行、カリングテストを行う。テスト結果は圧縮されてメモリに保存され、メインの Render パイプラインはテスト結果を用いて無駄なフェッチやシェーディングを省く。  
POSH パイプラインと POCS は *Gen11 アーキテクチャ* から追加、実装されているが、*Tiger Lake* のドキュメントにおいて、GPU は 1つのジオメトリパイプラインで構成され、POSH パイプラインは維持する (maintain) とあることから、現状無効化されているのではないかと思われる。[^tgl-configuration]  

[^tgl-configuration]: [Volume 4: Configurations - intel-gfx-prm-osrc-tgl-vol04-configurations_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-tgl-vol04-configurations_0.pdf)

{{< figure src="../posh-pipeline.webp" title="POSH Pipeline" caption="引用元: [Volume 9: Render Engine - intel-gfx-prm-osrc-tgl-vol09-renderengine_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-tgl-vol09-renderengine_0.pdf)" >}}

### Compute Command Streamer {#ccs}
RCS/Render Engine は 3D、GPGPU (Compute)、シェーダーを用いたメディア処理に使われる命令をサポートしている。  
CCS/Compute Engine はそこから GPGPU (Compute) の命令のみをサポートする、RCS/Render Engine のサブセットとなる。  
機能的には AMDGPU の MEC (MicroEngine Compute) と近いのではないかと思う。  
*Gen11 アーキテクチャ* までは RCS/Render Engine 1基の構成だったが、*Gen12/{{< xe >}} アーキテクチャ* から CCS/Compute Engine を追加で搭載する構成となった。  

CCS/Compute Engine は *Xe_HP SDV ({{< xe class="hp" >}})*、*DG2/Alchemist ({{< xe class="hpg" >}})*、*Ponte Vecchio ({{< xe class="hpc" >}})* では 4基搭載されている。  
また、*Xe_HP SDV* と *Ponte Vecchio* は RCS/Render Engine、3Dパイプラインを持たず、CCS/Compute Engine のみを搭載する構成となっている。  

*Tiger Lake* のドキュメントに CCS/Compute Engine の記述があり、またダイアグラムには CCS 1基分が描かれているが、Linux Kernel における Intel GPU ドライバー、i915 では `platform_engine_mask` に `CSS{0..4}` のビットが設定されているのは *Xe_HP SDV, DG2/Alchemist, Ponte Vecchio* のみであるため、こちらも *Tiger Lake* GPU では無効化されていると思われる。[^engine-mask]  

そのため、今回 Intel Iris (OpenGL) / ANV (Vulkan) ドライバーに追加された CCS/Compute Engine のサポートは *Xe_HP SDV, DG2/Alchemist, Ponte Vecchio* に向けたものとも見れる。  

[^engine-mask]: <https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/i915/i915_pci.c?h=next-20220616>

{{< ref >}}
 * [[Intel-gfx] [PATCH v3 01/13] drm/i915/xehp: Define compute class and engine](https://lists.freedesktop.org/archives/intel-gfx/2022-March/291525.html)
 * [[Intel-gfx] [PATCH 1/2] drm/i915/xehp: Support platforms with CCS engines but no RCS](https://lists.freedesktop.org/archives/intel-gfx/2022-March/291770.html)
 * [[Intel-gfx] [PATCH 05/10] drm/i915: Xe_HP SDV and DG2 have 4 CCS engines](https://lists.freedesktop.org/archives/intel-gfx/2022-April/295662.html)
 * [2020-2021 INTEL(R) PROCESSORS BASED ON THE "TIGER LAKE" PLATFORM | 01.org](https://01.org/node/37295)
 * [2020 Intel(r) Processors with Hybrid Technology based on the Lakefield platform | 01.org](https://01.org/node/37196)
 * [2020 Discrete GPU formerly named "DG1" | 01.org](https://01.org/node/37109)
 * [2019 Intel(r) processors based on Ice Lake platform | 01.org](https://01.org/linuxgraphics/hardware-specification-prms/2019-intelr-processors-based-ice-lake-platform)
{{< /ref >}}
