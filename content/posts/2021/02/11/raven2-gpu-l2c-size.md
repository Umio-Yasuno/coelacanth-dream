---
title: "Raven2 の GPU L2キャッシュは 128KB"
date: 2021-02-11T20:29:38+09:00
draft: false
tags: [ "Raven2", "Dali", "Pollock" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

オープンソースで開発されるユーザーモードドライバー **Mesa3D** に投稿されたパッチから、*Raven2 (Dali/Pollock) APU* が持つ GPU L2キャッシュのサイズが 128KB であることが明かされた。  
*Raven2 (Dali/Pollock) APU* は CPUに *Zenアーキテクチャ* 2-Core/4-Thread、GPU に *Vega/GFX9 (gfx909)* 3CU を持つ、小型な省電力 SoC。GF 14nmプロセスで製造される。  

 * [ac/gpu_info: conceal L2 cache sizes (29ca71e1) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/29ca71e10e58077fb847a914b5051e69a4add352)

## Raven2 GPU L2$

今回のパッチで、*Raven2* の GPU L2キャッシュはブロックあたり 64KB とされ、  
別のオープンソースで開発される AMDVLK の一部を構成する [PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal) のコードには、2ブロックあると記述されている。  
そして、*Raven2* の GPU L2キャッシュは計 128KB となる。  

ちなみに、コード中に使われている *TCC* は *Texture Channel Cache* の略とされている。[^tcc]  
そのフルネームが記述されることは最近ではなくなり、また資料中で解説されることもほとんどなく、半ば記号と化している。  

[^tcc]: [[radeonsi] Tahiti LE: GFX block is not functional, CP is okay (#1208) · Issues · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/issues/1208#note_240161)

 >           case CHIP_RAVEN2:
 >              info->l2_cache_size = info->num_tcc_blocks * 64 * 1024;
 >              break;
 >
 > {{< quote >}} [ac/gpu_info: conceal L2 cache sizes (29ca71e1) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/29ca71e10e58077fb847a914b5051e69a4add352) {{< /quote >}}
 >
 >             else if (AMDGPU_IS_RAVEN2(m_nullIdLookup.familyId, m_nullIdLookup.eRevId))
 >            {
 >                pChipInfo->doubleOffchipLdsBuffers = 1;
 >                pChipInfo->gbAddrConfig            = 0x26010001; // GB_ADDR_CONFIG_DEFAULT;
 >                pChipInfo->numShaderEngines        =    1; // GPU__GC__NUM_SE;
 >                pChipInfo->numShaderArrays         =    1; // GPU__GC__NUM_SH_PER_SE;
 >                pChipInfo->maxNumRbPerSe           =    1; // GPU__GC__NUM_RB_PER_SE;
 >                pChipInfo->nativeWavefrontSize     =   64; // GPU__GC__WAVE_SIZE;
 >                pChipInfo->minWavefrontSize        =   64;
 >                pChipInfo->maxWavefrontSize        =   64;
 >                pChipInfo->numPhysicalVgprsPerSimd =  256; // GPU__GC__NUM_GPRS;
 >                pChipInfo->maxNumCuPerSh           =    3; // GPU__GC__NUM_CU_PER_SH;
 >                pChipInfo->numTccBlocks            =    2; // GPU__TC__NUM_TCCS;
 >
 > {{< quote >}} [pal/ndDevice.cpp at 57cd977c79e4321c28dcb1a18a4aa23880aa48f4 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/57cd977c79e4321c28dcb1a18a4aa23880aa48f4/src/core/os/nullDevice/ndDevice.cpp#L921) {{< /quote >}}

L2キャッシュサイズが何故今更分かったか、明かされたかと言えば、オープンソースドライバー等で最適化に使われるのは、基本、メモリ帯域や RB (Render Backend) に対応する L2キャッシュブロックの数であり、サイズ自体はあまり使われていないからだ。  
今回 L2キャッシュサイズの情報が追加されたのは、ドライバーのトレース機能から出力された情報を解析する [Radeon GPU Analyzer (RGA)](https://github.com/GPUOpen-Tools/radeon_gpu_analyzer) に使われるためで、オープンソースドライバーでも RGA への対応を強化する考えがあるものと思われる。  

*Raven/Picasso APU* 等は資料中で GPU L2キャッシュに 1MB 持つことが触れられていたが、*Raven2* は省電力低コストという特徴のためか、個人的には他の APU よりもひっそりしているという印象を受ける。 

以前に、[Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/) 氏が撮影したダイショットを元に各ブロックを推測したことがあったが、その時には気が付けなかった。  
言い訳をすると、SoC として多くの機能を持つプロセッサは、各ブロックが入り組んだ構造をしており {{< comple  >}} CPUコアや GPU CU 等の分かりやすいもの以外 {{< /comple >}} 2つを並べて見比べても、あるブロックが何のユニットで、片方のどれと対応するかを推測することは難しい。  
{{< link >}} [AMD Raven2 のダイ観察 & Zen系 APU を比較 | Coelacanth's Dream](/posts/2020/07/24/raven2-dieshot-and-compare-zen-apu/) {{< /link >}}

*Raven2* の GPU部は *Raven/Picasso* のそれから規模を大きく減らしており、CU数は約1/4、RB は 2基から 1基に、そして L2キャッシュは 1/8 の規模となる。  
AMD GPU は *Vega/GFX9* 世代から、L2キャッシュのブロックあたりのサイズは 256KB を基本としており、*RDNA/GFX10* 世代もそのようになっている。( Polaris10/11 等、GFX8 世代でもブロックあたり 256KB の GPU は存在する)  
L2キャッシュのサイズは選択でき、[rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf) には 64KB から 512KB の構成を取ることができるとある。  
それでも *RDNA/GFX10* 世代では 256KB の構成を取り続けていることから、性能や効率、必要とされる面積という点で 256KB が最適だという判断があったはずで、  
比較的最近の APU でありながらブロックあたり 64KB の構成を取ったことは、*Raven2* が省電力低コストという特徴を強める要素だと言える。  



{{< dia >}}
```
## Raven2 Diagram

 +- ShaderEngine(00) -----------------+
 | +- ShaderArray(00) --------------+ |
 | |  ==== ====  CU (00) ==== ====  | |
 | |  ==== ====  CU (01) ==== ====  | |
 | |  ==== ====  CU (02) ==== ====  | |
 | |  [ RB+ ]                       | |
 | |  [ Rasterizer/Primitive Unit ] | |
 | +--------------------------------+ |
 |      [- Geometry Processor -]      |
 +------------------------------------+
 [L2$  64K] [L2$  64K]
```
```
## Raven/Picasso Diagram

 +- ShaderEngine(00) -----------------+
 | +- ShaderArray(00) --------------+ |
 | |  ==== ====  CU (00) ==== ====  | |
 | |  ==== ====  CU (01) ==== ====  | |
 | |  ==== ====  CU (02) ==== ====  | |
 | |  ==== ====  CU (03) ==== ====  | |
 | |  ==== ====  CU (04) ==== ====  | |
 | |  ==== ====  CU (05) ==== ====  | |
 | |  ==== ====  CU (06) ==== ====  | |
 | |  ==== ====  CU (07) ==== ====  | |
 | |  ==== ====  CU (08) ==== ====  | |
 | |  ==== ====  CU (09) ==== ====  | |
 | |  ==== ====  CU (10) ==== ====  | |
 | |  [ RB+ ][ RB+ ]                | |
 | |  [ Rasterizer/Primitive Unit ] | |
 | +--------------------------------+ |
 |      [- Geometry Processor -]      |
 +------------------------------------+
 [L2$ 256K] [L2$ 256K] [L2$ 256K] [L2$ 256K]
```
{{< /dia >}}

|       | Raven2    | Raven/Picasso |
| :--   | :--:      | :--: |
| CPU Core/Thread | 2/4 | 4/8 |
| GPU | gfx909 | gfx902 |
| GPU CU | 3CU | 11CU |
| GPU L2$ | 128KB | 1024KB |
| GPU RB+[^rbplus] | 1 RB+<br>(8-ROP) | 2 RB+<br>(16-ROP) |

[^rbplus]: [一部の AMD GPU で実装、有効化されている RB+ とは何か | Coelacanth's Dream](/posts/2020/11/10/what-is-rbplus/)
