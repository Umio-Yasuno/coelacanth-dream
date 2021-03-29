---
title: "AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish, Aldebaran, VanGogh"
date: 2021-03-30T02:12:10+09:00
draft: false
tags: [  "Aldebaran", "VanGogh", "Linux_Kernel", "Dimgrey_Cavefish" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

*Vega10* 以降の AMD GPU のキャッシュ構成情報をドライバーに追加するパッチが Linux Kernel (amd-gfx) に投稿された。  
キャッシュ情報が追加されたのは、Kernelドライバーでも KFD (Kernel Fusion Driver) の部分であり、KFD は各種ROCmソフトウェアを動作させるためのドライバーとして機能する。  
これまでは CU ごとに持つプライベートな L1ベクタキャッシュの情報だけだったが、今回のパッチにより、複数の CU で共有するスカラL1データ/命令キャッシュ、L2データキャッシュ、そして *RDNA 2/GFX10.3* 世代から追加された L3データキャッシュの情報が追加された。  

 * [[PATCH] drm/amdkfd: Update L1 and add L2/3 cache information](https://lists.freedesktop.org/archives/amd-gfx/2021-March/061392.html)


{{< pindex >}}

 * [Aldebaran のキャッシュ構成](#aldebaran)
 * [VanGogh のキャッシュ構成と CU数　―― GL1キャッシュは持つが L3キャッシュは持たず](#vgh)
 * [Dimgrey Cavefish/Navi23 のキャッシュ構成　―― 32MB の L3キャッシュ](#dimgrey_cavefish)
 * [AMD GPU cache info](#cache-info)

{{< /pindex >}}

## Aldebaran のキャッシュ構成 {#aldebaran}

*Aldebaran* の CU はプライベートL1キャッシュ 16KB を持ち、これは他の AMD GPU アーキテクチャ、*GCN, CDNA, RDNA/2* と同じである。  
L1データ/命令キャッシュは、*Vega/GFX9* 世代では基本 CU 3基で共有していたが、2基で共有する形となる。L1データキャッシュは 16KB、L1命令キャッシュは 32KB で、キャッシュサイズに変わりはない。  
L2データキャッシュは 8192KB (8MB)、CU 14基で共有すると記述されているが、ここでの CU は SE (ShaderEngine) あたりの数であり、全体の規模ではないと思われる。  
しかし、**Radeon Instinct MI50/60** のベースとなる *Vega20* や *CDNA アーキテクチャ* を採用する *Arcturus/MI100* が SE あたりの CU 16基、というバランスを採り続けてきたことを考えると、それよりもわずかに少ない CU 14基となるのは興味深い点ではある。  
それでも、*Aldebaran* では CU あたりの演算性能が大幅に強化されており、SE あたりの CU数が前世代よりも 2基減ったとしても、SE のような広い範囲で見た演算性能は前世代よりもずっと大きくなる。  
{{< link >}} [LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か | Coelacanth's Dream](/posts/2021/02/19/llvm-gfx90a/) {{< /link >}}


 >        +static struct kfd_gpu_cache_info aldebaran_cache_info[] = {
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
 >        +		/* L2 Data Cache per GPU (Total Tex Cache) */
 >        +		.cache_size = 8192,
 >        +		.cache_level = 2,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 14,
 >        +	},
 >        +};
 >
 > {{< quote >}} [[PATCH] drm/amdkfd: Update L1 and add L2/3 cache information](https://lists.freedesktop.org/archives/amd-gfx/2021-March/061392.html) {{< /quote >}}


今回のパッチでは *Vega20* と *Arcturus* のキャッシュ情報を共通のものとしており、そのためか *Vega20* の L2キャッシュサイズが 8192KB(8MB) になっているが、*Vega20* は 4MB、*Arcturus* は 8MB というのが正しいように思われる。  

*Aldebaran* では *Arcturus* から L2キャッシュサイズが変わらないこととなるが、L2キャッシュラインサイズを変更することは明らかにされており、*RDNA/2 アーキテクチャ* と同じ 128Byte になる。  
キャッシュラインサイズを大きくすることは、メモリアクセスのデータ単位を大きくすることとなり、少ないメモリリクエストで広いメモリ帯域を活用することができる。  
{{< link >}} [RadeonSI に Aldebaran GPU のサポートを追加するパッチが投稿される | Coelacanth's Dream](/posts/2021/03/04/aldebaran-umd/#tcc-line-128B) {{< /link >}}

## VanGogh のキャッシュ構成と CU数　―― GL1キャッシュは持つが L3キャッシュは持たず {#vgh}

パッチでは *VanGogh APU* のキャッシュ情報も追加されている。  
プライベートL1キャッシュ、スカラL1データ/命令キャッシュは他と同様。  
*VanGogh* において興味深いのは *RDNA/GFX10* から追加された GL1キャッシュは持つが、*RDNA 2/GFX10.3* から追加された *L3キャッシュ/Infinity Cache* は持たないことだ。  

*VanGogh* の前にキャッシュの呼称についての解説を挟むと、  
*RDNA系アーキテクチャ* では CU内のプライベートL1キャッシュ、複数の CU で共有する L1データ/命令キャッシュと、グローバルでアクセス可能な L2キャッシュの間に階層が 1つ増設された。  
そして、*GCN/CDNA アーキテクチャ* における L1キャッシュを *RDNA アーキテクチャ* では L0キャッシュとし、新たなキャッシュ階層を L1キャッシュとした。  
マーケティング上の資料等ではそうした呼び方で問題無いと思われるが、ドライバー等の扱いでは、呼び方が違ってもキャッシュの扱いは同じであるため、その呼び方をそのまま使うのではややこしく、問題が生じる原因となる。  
そのため、オープンソース・ドライバーのコードを読む限りでは、*RDNA アーキテクチャ* における L1キャッシュを `GL1` としている。  
AMD GPU のキャッシュの別名は他にもあり、プライベートL1キャッシュは `TCP (Texture Cache Private?)` 、L2キャッシュは `TCC (Total Texture Cache?)` [^tcc] 、L3キャッシュは `MALL (Memory Access at Last Level)` とされる。  

[^tcc]: Texture Channel Cache とされることも

*VanGogh* の各 L1キャッシュサイズは他と同じで、L1データ/命令キャッシュを CU 2基で共有するのは *RDNA系アーキテクチャ* で共通する特徴 (WGP) となる。  
そして、*VanGogh* は GL1キャッシュ 128KB を持ち、これは他の *RDNA系アーキテクチャ* を採用するディスクリートGPUと同じ規模のキャッシュサイズとなる。  
L2キャッシュは 1024KB (1MB) であり、これは近年の *Vega/GFX9* 世代 APU {{< comple >}} Raven, Picasso, Renoir, Lucienne, Cezanne {{< /comple >}} と同じサイズであり、AMD としては APU の GPU部に持たせる L2キャッシュはまだ 1MB がちょうど良いという判断なのかもしれない。  
それでも GL1キャッシュを持つため、GPU のメモリ性能向上が期待できる。  

また、CU 8基で共有するとされているが、他の RDNA系dGPU の情報を見るに、SA (ShaderArray) あたりの CU数ではないかと思われる。  
*RDNA系アーキテクチャ* では SE あたりの SA数が 2基となっているため、*VanGogh* でもそうなるかはここでは読み取れない。  
しかし、L2キャッシュ 1MB ということから、*Vega/GFX9* 世代 APU からそう CU数を増やすことはせず、SE あたり SA 1基という構成を採る可能性は考えられる。  

そして、*VanGogh APU* は GPU部に *RDNA 2 アーキテクチャ* を採用するが、L3キャッシュ/Infinity Cache は持たない。  
このことはディスプレイエンジン周りのドライバーコードから読み取ることができ、正確な情報と思われる。  
{{< link >}} [Dimgrey Cavefish/Navi23 の Infinity Cache はメモリチャネルあたり 4MiB | Coelacanth's Dream](/posts/2021/03/04/dimgrey_cavefish-4mb-mall-per-ch/#mall-size) {{< /link >}}

*RDNA 2アーキテクチャ* でありながら L3キャッシュ/Infinity Cache を持たず、また SE あたり SA 1基という構成 {{< comple >}} を採る可能性がある {{< /comple >}} からは、コストを重視した APU/SoC という設計思想が垣間見えるだろう。  

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


## Dimgrey Cavefish/Navi23 のキャッシュ構成　―― 32MB の L3キャッシュ {#dimgrey_cavefish}

*Dimgrey Cavefish/Navi23* のキャッシュ情報も追加されており、そこからはメモリバス幅が読み取れる。  

*Dimgrey Cavefish* は L2キャッシュ 2048KB (2MB)、L3キャッシュ/Infinity Cache 32MB を持つ。  
L3キャッシュ/Infinity Cache については、ディスプレイエンジン周りのドライバーコードから、メモリチャネルあたり 4MB ということが読み取れ、他 *RDNA 2* dGPU同様に GDDR6メモリ (1ch == 16-bit) だとするとメモリバス幅は 128-bit だと考えられる。  
{{< link >}} [Dimgrey Cavefish/Navi23 の Infinity Cache はメモリチャネルあたり 4MiB | Coelacanth's Dream](/posts/2021/03/04/dimgrey_cavefish-4mb-mall-per-ch/#mall-size) {{< /link >}}
L2キャッシュ 2MB からも、他同様に L2キャッシュブロックあたり 256KB とすると 8ブロック (ch) となり、ここでも 128-bit と考えられる。  

既に製品が出ている *RDNA 2* dGPU、*Sienna Cichlid/Navi21* 、*Navy Flounder/Navi22* では、メモリチャネルあたり 8MB の L3キャッシュ/Infinity Cache を持ち、*Dimgrey Cavefish/Navi23* はそれよりも小さいバランスとなるが、これは製造コストを意識した結果、あるいはターゲット帯に必要な性能を考えてのものだろう。  
また、 *Dimgrey Cavefish* の SA あたりの CU数は 8基とされ、*Sienna Cichlid* 、*Navy Flounder* は 10基であったから、ここでも小さめバランスを取ったことが読み取れる。  

 >        +static struct kfd_gpu_cache_info dimgrey_cavefish_cache_info[] = {
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
 >        +		.cache_size = 2048,
 >        +		.cache_level = 2,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 8,
 >        +	},
 >        +	{
 >        +		/* L3 Data Cache per GPU */
 >        +		.cache_size = 32*1024,
 >        +		.cache_level = 3,
 >        +		.flags = (CRAT_CACHE_FLAGS_ENABLED |
 >        +				CRAT_CACHE_FLAGS_DATA_CACHE |
 >        +				CRAT_CACHE_FLAGS_SIMD_CACHE),
 >        +		.num_cu_shared = 8,
 >        +	},
 >        +};
 >
 > {{< quote >}} [[PATCH] drm/amdkfd: Update L1 and add L2/3 cache information](https://lists.freedesktop.org/archives/amd-gfx/2021-March/061392.html) {{< /quote >}}

## AMD GPU cache info {#cache-info}

| AMD GPU<br>cache info | RDNA L1 (GL1) | L2 (TCC) | L3/Infinity Cache (MALL) |
| :-- | :--: | :--: | :--: |
| Raven/Picasso<br>Renoir/Lucienne/Cezanne | - | 1M | - |
| Raven2 | - | 512K | - |
| Vega20 | - | 4M | - |
| Arcturus | - | 8M? | - |
| Aldebaran | - | 8M | - |
| Navi10/Navi12 | 128K | 4M | - |
| Navi14 | 128K | 2M | - |
| VanGogh | 128K | 1M | - |
| Sienna Cichlid | 128K | 4M | 128M |
| Navy Flounder | 128K | 3M | 96M |
| Dimgrey Cavefish | 128K | 2M | 32M |
