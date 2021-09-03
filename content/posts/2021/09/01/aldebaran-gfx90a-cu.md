---
title: "Aldebaran/MI200 はダイあたり 55CU、トータル 110CU か?"
date: 2021-09-01T15:40:37+09:00
draft: false
tags: [ "Aldebaran", "gfx90a", "CDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

オープンソースで開発される AMD GPU の高性能な機械学習用ライブラリ [MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen) に、*Aldebaran/MI200 (gfx90a)* に向けた畳み込みアルゴリズムの最適化情報を記述したデータベースファイルを追加するプルリクエストが投稿されている。  
そこに *Aldebran/MI200* の CU数を示すようなコメントがあった。  

 * [[Tuning] rocm 4.4 update for release branch by cderb · Pull Request #1132 · ROCmSoftwarePlatform/MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen/pull/1132)

 > 		Binary file has shrunk: only includes perf entries for gfx906_60, gfx908_120, gfx90a_110.
 >
 > {{< quote >}} <https://github.com/ROCmSoftwarePlatform/MIOpen/pull/1132#issue-723584777> {{< /quote >}}

MIOpen では、いくつかの製品に向けて CU数を含めた最適化情報を記述したデータベースファイルを作成、使用している。  
コメントにある `gfx906_60` は [Instinct MI50](https://www.amd.com/en/products/professional-graphics/instinct-mi50-32gb#product-specs)、`gfx908_120` は [AMD instinct MI100](https://www.amd.com/en/products/server-accelerators/instinct-mi100#product-specs) を想定しているものと思われる。  

 * [MIOpen/find_and_immediate.md at develop · ROCmSoftwarePlatform/MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen/blob/develop/doc/src/find_and_immediate.md#limitations-of-immediate-mode)

## 110CU? {#110cu}

まず断っていくと、{{< comple >}} 多くのことに言えるが {{< /comple >}} いくつか根拠はあっても 100%正しくは無い。取分け、自分が取り上げるような情報や推測は。  
そのため、110CU という規模を前提して何かしら書くのではなく、あくまでもその規模がどうか、ということを書いていきたい。  

*Aldebran/MI200 (gfx90a)* で確定してる情報をいくつか列挙すれば、  

 * フルレートFP64演算に対応し、FP32 のパックド実行にも対応
    {{< link >}} [LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か | Coelacanth's Dream](/posts/2021/02/19/llvm-gfx90a/#pkfp32-fullfp64) {{< /link >}}
 * プライマリーダイとセカンダリーダイの GPUダイ 2基による MCM 構成を採る
    {{< link >}} [プライマリーダイとセカンダリーダイで構成される Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/06/09/aldebaran-primary-secondary/) {{< /link >}}
 * ダイあたり 4基の UMC (Unified Memory Controller) を持つ
    {{< link >}} [ダイあたり 4基のメモリコントローラーを持つ Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/07/01/aldebaran-gpu-edac/) {{< /link >}}

等がある。  

そして、KFD (Kernel Fusion Driver) 部に追加されたキャッシュ情報から、Shader Engine あたり 14CU の構成を採ると考えられる。  
{{< link >}} [AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/) {{< /link >}}

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

従来の AMD GPUアーキテクチャでは、各 Shader Engine の規模がそれぞれが同規模になるように設定される。例えば *Arcturus/MI100* は Shader Engine 8基、各 SE に最大 16基の CU を持つが、歩留まり向上等の理由から、製品では各 SE の CU を 1基を無効化し、合計 CU 120基という設定を採っていた。  

これを *Aldebran/MI200* にも適用すると、全体では 10基の SE を持ち、各 SE から CU 3基を無効化、合計 CU 110基となる。  
各 SE から CU 3基を無効化というのは、わずかに冗長部が多いようにも感じるが、フルレートFP64 に対応する関係で CU のリソース量は大幅に増えていると予想され、歩留まり向上のため冗長部が増えるのは自然だとも考えられる。  

関係して、*Arcturus/MI100* では FP64 の性能が FP32 の半分のレートであったため、*Aldebaran/MI200* が CU 110基という *Arcturus/MI100* より 10基少ない規模だとしても性能は大きく向上している。  

別の Shader Engine、CU の設定を考えると、*Aldebaran/MI200* は GPUダイ 2基による MCM構成であるため、*仮に* Shader Engine ではなくダイ単位で対称となる設定を採るならば、各ダイが SE 4基 (CU 56基) を持ち、その内 1基をそれぞれ無効化して合計 CU 110基という構成も有り得る。  
こちらの方が構成としてはシンプルで、すっきりした印象を受ける。  

CU 110基という規模自体はそうおかしいものではなく、*Aldebaran/MI200* は各ダイに 4基の UMC (メモリインターフェイス: 4096-bit) を持ち、これは *Arcturus/MI100* と同じ規模だ。  
*Aldebran/MI200* は HBM2e に対応しているが、*Arcturus/MI100* は 2.4 Gbps の HBM2 を採用しているため、メモリ帯域の向上は約 1.2〜1.3倍程度に留まると思われる。それよりもメモリスタックあたり 16GB の容量が選択できることの恩恵が大きい。  
そして *Aldebaran/MI200* では CU あたりの演算性能が大幅に向上し、クロックあたりのピーク性能は倍になることを考えれば、各ダイに CU 55基というのは演算性能とメモリ性能のバランスを保った構成ということがわかる。  

| CDNA | Arcturus/MI100 | Aldebaran/MI200 |
| :-- | :--: | :--: |
| Active CUs | 120 CU | 110 CU?<br> (2x 55 CU?) |
| FP32:FP64 Rate <br> (Arcturus FP32 == 1) | 1 : 0.5 | 2 : 1 ? |
| Memory | HBM2 2.4Gbps  | HBM2e |
| Memory Bus width | 4096-bit | 8192-bit? <br> (2x 4096-bit) |
| Memory Size | 32 GB | 128GB ? <br> (2x 64GB ?) |
