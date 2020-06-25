---
title: "Intel、DG1 において OpenCL と oneAPI Level Zero をサポート ――巨大なキャッシュを持つ DG1"
date: 2020-06-20T06:56:57+09:00
draft: false
tags: [ "DG1", "Gen12" ]
keywords: [ "", ]
categories: [ "Hardware", "GPU", "Intel" ]
noindex: false
---

Intel は [oneAPI Level Zero](https://spec.oneapi.com/versions/latest/elements/l0/source/index.html) と OpenCLドライバー のための [compute-runtime](https://github.com/intel/compute-runtime) に、Gen12アーキテクチャ採用の dGPU、*DG1* をサポートするパッチを投稿した。  
{{< link >}}[Add DG1 support to OpenCL and Level Zero (1/n) · intel/compute-runtime@3029db0](https://github.com/intel/compute-runtime/commit/3029db07c3135ec5fde55de369ead2c14f9b3f9c){{< /link >}}

その中で *DG1* の詳細な構成が明らかとなっており、同じ Gen12アーキテクチャで、EU数が同規模の *Tiger Lake* とはキャッシュやスレッドの構成が大きく異なることがわかった。  

## 巨大なキャッシュ、より多くのスレッド

*DG1* は *Tiger Lake GT2* と同様に、  
EU数 96基、L3cacheバンク 8基、最大Pixel Fill Rate 16/clock (16ROP相当)、  
EUをまとめる Sub-Slice内の共有キャッシュ(SLM)のサイズ 64KB、EUあたりが保持するスレッド数 7スレッド、という構成になる。[^2][^3]  

しかし、*DG1* の L3cacheバンクあたりの容量は大幅に増量され、*Tiger Lake GT2* がバンクあたり 480KB、計 3840KB であるのに対し、  
*DG1* はバンクあたり 2048KB(2MB)、計 16384KB(16MB) となっている。  
{{< link >}}[compute-runtime/hw_info_dg1.inl at 3029db07c3135ec5fde55de369ead2c14f9b3f9c · intel/compute-runtime](https://github.com/intel/compute-runtime/blob/3029db07c3135ec5fde55de369ead2c14f9b3f9c/opencl/source/gen12lp/hw_info_dg1.inl){{< /link >}}
{{< link >}}[compute-runtime/hw_info_tgllp.inl at 3029db07c3135ec5fde55de369ead2c14f9b3f9c · intel/compute-runtime](https://github.com/intel/compute-runtime/blob/3029db07c3135ec5fde55de369ead2c14f9b3f9c/opencl/source/gen12lp/hw_info_tgllp.inl){{< /link >}}
また、各シェーダーステージでの生成可能なスレッド数も増やされており、その数 *DG1* は *Tiger Lake GT2* の倍となる。  

[^2]: [compute-runtime/hw_cmds_dg1.h at 3029db07c3135ec5fde55de369ead2c14f9b3f9c · intel/compute-runtime](https://github.com/intel/compute-runtime/blob/3029db07c3135ec5fde55de369ead2c14f9b3f9c/shared/source/gen12lp/hw_cmds_dg1.h)
[^3]: [compute-runtime/hw_cmds_tgllp.h at 3029db07c3135ec5fde55de369ead2c14f9b3f9c · intel/compute-runtime](https://github.com/intel/compute-runtime/blob/3029db07c3135ec5fde55de369ead2c14f9b3f9c/shared/source/gen12lp/hw_cmds_tgllp.h)

96EU(768SP)、16ROP相当というのは、AMD GPU では *Polaris11 (1024SP, 4RB/16ROP)* 、*Polaris12 (640SP, 4RB/16ROP)* なんかが近い規模となる。  
Intel GPUアーキテクチャでは L3cacheバンク内で、バッファとして機能する URB、DC(Data Cluster)、Color/Zキャッシュ等それぞれに容量を割り振るため、データキャッシュのサイズはその設定次第となる。  
また、キャッシュ階層等が異なるため単純な性能比較は難しいが、それでもキャッシュサイズ自体がかなり大きいことは確かだ。  
設定次第というのも、言い換えれば巨大なキャッシュをグラフィック処理向けにも、GPGPU向けにも設定できる。  

この点で *DG1* は尖っていると言え、またソフトウェア開発向けの立ち位置にいるのも頷けるかもしれない。  
そして巨大なキャッシュから、ゲーム等のグラフィック処理以外にも、{{< xe class="hpc" >}} に向けたGPGPU処理の最適化も *DG1* で進めたい思惑があるのかもしれない。  
これまでに *Tiger Lake* のパッケージ、ダイは公開しているのに[^1]、*DG1* はカードの外観のみだったのはダイサイズから巨大なキャッシュに気づかれることを避けたかったのではないか、とも考えられる。  

[^1]: [2020 CES: Intel Brings Innovation to Life with Intelligent Tech Spanning the Cloud, Network, Edge and PC | Intel Newsroom](https://newsroom.intel.com/news-releases/intel-ces-2020/)

| Gen12LP | TGL GT2 | DG1 |
| :-- | :--: | :--: |
| Dual Sub Slice | 6 | 6 |
| &emsp;EU | 96 | 96 |
| &emsp;&emsp;Shading Unit | 768 | 768 |
| GPU L3$ | 3840 KB | *16384 KB* |

{{< ref >}}

 * [2019 Intel(r) processors based on Ice Lake platform | 01.org](https://01.org/linuxgraphics/hardware-specification-prms/2019-intelr-processors-based-ice-lake-platform)
    * [Cover_Sheet - intel-gfx-prm-osrc-icllp-vol09-renderengine_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol09-renderengine_0.pdf)
    * [Volume 4: Configurations - intel-gfx-prm-osrc-icllp-vol04-configurations_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol04-configurations_0.pdf)
    * [Volume 7: Memory Cache - intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf)

{{< /ref >}}
