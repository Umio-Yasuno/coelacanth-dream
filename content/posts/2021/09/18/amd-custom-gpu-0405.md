---
title: "VanGogh APU のマーケティングネームは「AMD Custom GPU 0405」"
date: 2021-09-18T06:29:14+09:00
draft: false
tags: [ "RDNA_2", "VanGogh" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

AMD のソフトウェアエンジニア Alex Deucher 氏により、[libdrm](https://gitlab.freedesktop.org/mesa/drm) に新たな AMD GPU/APU のマーケティングネームを追加するパッチ(マージリクエスト) が公開された。  
そしてその中に `DeviceID: 0x163F, RevisionID: 0xAE`、マーケティングネーム **AMD Custom GPU 0405** という異質なものがある。  

 > 		+163F,	AE,	AMD Custom GPU 0405
 >
 > {{< quote >}} [amdgpu: add marketing names from 21.30 (!195) · Merge requests · Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm/-/merge_requests/195) {{< /quote >}}

名前こそ異質だが、`0x163F, 0xAE` に該当する AMD APU は既に姿を現しており、2021/03 頃に公開された *VanGogh APU* のブートログに登場していた。  
下引用部は `initializing kernel modesetting` の後、AMDGPU ASIC NAME, VendorID, DeviceID, SubSystem VendorID, SubSystem DeviceID, RevisionID の順番で表示されている。[^drm-info]  
{{< link >}} [AMD VanGogh APU ブートログ | Coelacanth's Dream](/posts/2021/03/17/vgh-bootlog/) {{< /link >}}

[^drm-info]: [linux/amdgpu_device.c at bdb575f872175ed0ecf2638369da1cb7a6e86a14 · torvalds/linux](https://github.com/torvalds/linux/blob/bdb575f872175ed0ecf2638369da1cb7a6e86a14/drivers/gpu/drm/amd/amdgpu/amdgpu_device.c#L3459)

 >        [   99.930720] [drm] initializing kernel modesetting (VANGOGH 0x1002:0x163F 0x1002:0x0123 0xAE).
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

## VanGogh APU と Steam Deck {#steam-deck}

Hot Chips 33 では、VanGogh APU と Steam Deck についてはノーコメントとされたらしいが[^hc33]、多くの人が考えているように、自分も Steam Deck に採用されている APU が *VanGogh* だと考えている。  

[^hc33]: [Hot Chips 2021 Live Blog: Graphics (Intel, AMD, Google, Xilinx)](https://www.anandtech.com/show/16912/hot-chips-2021-live-blog-graphics-intel-amd-google-xilinx)

 * [Steam Deck :: Tech Specs](https://www.steamdeck.com/en/tech)

**Steam Deck** に採用されているカスタム APU は 4-Core/8-Thread、2.4-3.5 GHz の範囲で動作するがこれは上記ブートログのスペックと同じだ。  
*AMD Eng Sample* において、プロセッサ名の末尾 `35/24_N` はベースクロック 2.4GHz、ブーストクロック 3.5GHz であることを示す。  

 >        [    0.353153] smpboot: CPU0: AMD Eng Sample: 100-000000405-03_35/24_N (family: 0x17, model: 0x90, stepping: 0x1)
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

また **Steam Deck** のカスタム APU は GPU に *RDNA 2 アーキテクチャ* を採用し、CU 8基 (WGP 4基) という規模を取っている。  
*VanGogh* のキャッシュ情報と ShaderEngine の構成は、以前投稿されたパッチである程度明かされていたが、それによると *VanGogh* は ShaderEngine あたり CU 8基の構成を取る。  
{{< link >}} [AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/) {{< /link >}}
となると、*VanGogh APU* の GPU部は全体 SE (ShaderEngine) 1基、SE あたりの SA (ShaderArray) も 1基、SA あたりの CU数は 8基 (WGP 4基) とされる。  
*RDNA/2 アーキテクチャ* では *VanGogh APU* 以外、SE あたりの SA 2基という構成を取っており、*VanGogh* はそこから外れた珍しい構成となる。  
SA 1基のみとすることで GL1データキャッシュも 1基となり、キャッシュは減るがダイサイズを小さくすることができる。  
*VanGogh APU* は CPU部も *Zen 2 アーキテクチャ* 4コアと、CCX 1基という構成であり、CPU, GPU 共にコンパクトな設計となっている。  

 >        +static struct kfd_gpu_cache_info vangogh_cache_info[] = {
 >        +	{
 >        +		/* TCP L1 Cache per CU */
 >        +		.cache_size = 16,
 >        +		.cache_level = 1,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 1,
 >        +	},
 >        +	{
 >        +		/* Scalar L1 Instruction Cache per SQC */
 >        +		.cache_size = 32,
 >        +		.cache_level = 1,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_INST_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 2,
 >        +	},
 >        +	{
 >        +		/* Scalar L1 Data Cache per SQC */
 >        +		.cache_size = 16,
 >        +		.cache_level = 1,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 2,
 >        +	},
 >        +	{
 >        +		/* GL1 Data Cache per SA */
 >        +		.cache_size = 128,
 >        +		.cache_level = 1,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 8,
 >        +	},
 >        +	{
 >        +		/* L2 Data Cache per GPU (Total Tex Cache) */
 >        +		.cache_size = 1024,
 >        +		.cache_level = 2,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 8,
 >        +	},
 >        +};
 >
 > {{< quote >}} [[PATCH] drm/amdkfd: Update L1 and add L2/3 cache information](https://lists.freedesktop.org/archives/amd-gfx/2021-March/061392.html) {{< /quote >}}

で、**Steam Deck** のカスタム APU を *VanGogh APU* とすると、自分はメモリ構成に関して読み間違えていたことになる。  
自分はブートログから、*VanGogh APU* を LPDDR5 64-bit (4x 16-bit) だと考えていたが、**Steam Deck** のハードウェアスペックには LPDDR5 128-bit (4x 32-bit) と記載されている。  
LPDDR5、クアッドチャネルだというところまでは合っていたが、チャネルあたりのバス幅は間違っていた。  
ブートログはメモリサイズ 8GB、**Steam Deck** は 16GB となっているが、チャネル数は同じであるため、メモリチップあたりの容量違いではないかと思われる。  
`RAM width 256bits DDR5` という表示が間違っていた所は合っていて安心はしたが、不完全だったのは気恥ずかしい。  

