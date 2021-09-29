---
title: "AMDVLK GPUOpenドライバーが Navi12 に対応"
date: 2021-04-09T00:08:11+09:00
draft: false
tags: [ "GPUOpen", "Navi12", "RDNA" ]
# keywords: [ "", ]
categories: [ "AMD", "GPU", "Hardware" ]
noindex: false
# summary: ""
---

AMD が公式に提供しているオープンソース Vulkanドライバー、[AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK)、それを構成する [LLPC (LLVM-Based Pipeline Compiler)](https://github.com/GPUOpen-Drivers/llpc)、[PAL (Platform Abstraction Library)](https://github.com/GPUOpen-Drivers/pal)、[XGL (Vulkan API Layer)](https://github.com/GPUOpen-Drivers/xgl) が、**Radeon Pro V520** 等の製品に採用されている [Navi12](/tags/navi12) に対応した。  

 * [2021-4-7 update · GPUOpen-Drivers/AMDVLK@8891dec](https://github.com/GPUOpen-Drivers/AMDVLK/commit/8891decec42bb27cf6f9dfeebecb6b84c3b8ec31)
    * [Update llpc from commit: bb8a05414 · GPUOpen-Drivers/llpc@a8ec3c6](https://github.com/GPUOpen-Drivers/llpc/commit/a8ec3c6e6372dcfd812a2ea592141a821e9584b1)
    * [Update xgl from commit: c728233ca8 · GPUOpen-Drivers/xgl@e1be7ee](https://github.com/GPUOpen-Drivers/xgl/commit/e1be7ee14c39d3a36d9d1aacd00caf3acc437cde)
    * [Update pal from commit: e0d171c15 · GPUOpen-Drivers/pal@83635fb](https://github.com/GPUOpen-Drivers/pal/commit/83635fbee82fb21662a4737e34437c41172c6fe0)

## Navi12 内部構成 {#navi12}
 
*Navi12* の構成は PAL のコード中から読み取ることができ、全体の SE (ShaderEngine) 数は 2基、SE あたりの SH (ShaderArray) 数は 2基、SH あたりの WGP (CU) 数は 5 (10) 基、SE あたりの RB (RenderBackend) 数は 8基となっている。  
この規模は *Navi10* と同じであり、メモリサイドキャッシュの L2キャッシュも、ブロック数は 16基、総キャッシュサイズは 4MiB というのも同じだ。  
*Navi10* と *Navi12* の大きな違いは対応しているメモリにあり、*Navi10* は GDDR6メモリを採用するが、*Navi12* は 3D実装を用いる HBM2メモリを採用している。  
**Radeon Pro V520** では HBM2メモリの ECC機能を有効することもできるようになっている。  
メモリバス幅は 2048-bit、HBM2メモリではメモリチャネルを 128-bit としているため、メモリチャネル数は 16ch (16x128-bit) になる。*Navi10* もメモリチャネル数で言えば 16ch (16x16-bit) 構成であり、このあたりが L2キャッシュサイズが同じな理由になっている。  
そして、*Navi12* は INT4/INT8、FP16 等の低い精度を用いるドット積演算を高速化する命令に対応しており、機械学習における推論には *Navi10* よりも向いている。同様の命令には *Navi14* も対応しているが、*Navi10* は対応していない。  
{{< link >}} [Navi12情報近況 (2020-03-09) & 推測 | Coelacanth's Dream](/posts/2020/03/09/navi12-recent-info/) {{< /link >}}

また、*Navi12* は RDNA/Navi1x 系列では唯一 MxGPU/SR-IOV、1GPU を仮想的に分割し、複数のユーザーで利用する機能に対応している。  

総じて言えば、*Navi12* は電力効率に優れ高速な HBM2メモリを採用し、低精度のドット積を高速に処理するための命令と MxGPU/SR-IOV に対応している、クラウドゲーミングやサーバー用途に向いた GPU であり、**Radeon Pro V520** はそうした特徴を活かした製品である。  
今回 GPUOpenドライバーにサポートが追加されたのは、Vulkan API を用いるゲームのサポートを強化する向きがあるものと思われる。  

**Radeon Pro V520** は AWS EC2 G4adインスタンスに採用されており、カード自体の入手は難しいが、クラウド上であれば (比較的) 気軽に利用できる。  
{{< link >}} [AWS、RDNAアーキテクチャ GPU 「Radeon Pro V520」を採用する G4adインスタンスを発表 | Coelacanth's Dream](/posts/2020/12/02/aws-radeon-pro-v520/) {{< /link >}}
自分も一度試したことがあるが、まず GPU を使う環境構築に手こずり、そこからも公式のドライバーのバージョンが少し前のもので結局あまり活用できずにインスタンスを削除した。  
環境構築については、マーケットプレイスにあるものを使うのが一番手っ取り早い。  

 > 		    else if (AMDGPU_IS_NAVI12(m_nullIdLookup.familyId, m_nullIdLookup.eRevId))
 > 		    {
 > 		        pChipInfo->supportSpiPrefPriority  =    1;
 > 		        pChipInfo->doubleOffchipLdsBuffers =    1;
 > 		        pChipInfo->gbAddrConfig            = 0x44; // GB_ADDR_CONFIG_DEFAULT;
 > 		        pChipInfo->numShaderEngines        =    2; // GPU__GC__NUM_SE;
 > 		        pChipInfo->numShaderArrays         =    2; // GPU__GC__NUM_SA_PER_SE
 > 		        pChipInfo->maxNumRbPerSe           =    8; // GPU__GC__NUM_RB_PER_SE;
 > 		        pChipInfo->nativeWavefrontSize     =   32; // GPU__GC__SQ_WAVE_SIZE;
 > 		        pChipInfo->minWavefrontSize        =   32;
 > 		        pChipInfo->maxWavefrontSize        =   64;
 > 		        pChipInfo->numPhysicalVgprsPerSimd = 1024; // GPU__GC__NUM_GPRS;
 > 		        pChipInfo->maxNumCuPerSh           =   10; // GPU__GC__NUM_WGP_PER_SA * 2;
 > 		        pChipInfo->numTccBlocks            =   16; // GPU__GC__NUM_GL2C;
 > 		        pChipInfo->gsVgtTableDepth         =   32; // GPU__VGT__GS_TABLE_DEPTH;
 > 		        pChipInfo->gsPrimBufferDepth       = 1792; // GPU__GC__GSPRIM_BUFF_DEPTH;
 > 		        pChipInfo->maxGsWavesPerVgt        =   32; // GPU__GC__NUM_MAX_GS_THDS;
 > 		    }
 >
 > {{< quote >}} [Update pal from commit: e0d171c15 · GPUOpen-Drivers/pal@83635fb](https://github.com/GPUOpen-Drivers/pal/commit/83635fbee82fb21662a4737e34437c41172c6fe0#diff-334c7874500bba7b14e748047f54e1d18fa3e0db1289adacbb6d62da11124c3f) {{< /quote >}}

| RDNA | Navi10 | Navi14 | Navi12 |
| :-- | :--: | :--: | :--: |
| Total SE | 2 | 1 | 2 |
| Total SH | 4 | 2 | 2 |
| Total WGP (CU) | 20 (40) | 12 (24) | 20 (40) |
| Total RB | 16 | 8 | 16 |
| Memory Type | GDDR6 | GDDR6 | HBM2 |
| Memory Bus Width | 256-bit | 128-bit | 2048-bit |
| Memory Channel | 16ch (16x16-bit) | 8ch (8x16-bit) | 16ch (16x128-bit) |
| L2cache Size | 4MiB | 2MiB | 4MiB |

{{< ref >}}
 * [AMD Radeon™ Pro V520 Graphics | AMD](https://www.amd.com/en/products/server-accelerators/amd-radeon-pro-v520#product-specs)
{{< /ref >}}
