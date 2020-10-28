---
title: "Navi14 でもブロックチェーン/マイニング向け SKU が追加される"
date: 2020-10-28T15:23:42+09:00
draft: false
tags: [ "Navi14", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

先日 Linux Kernel (amd-gfx) にディスプレイエンジン部 (DCN) とマルチメディアエンジン部 (VCN) を無効化した、ブロックチェーン向けの *Navi10* SKU をサポートするパッチが投稿されたが、  
{{< link >}} [ブロックチェーン向けの Navi10 SKU とそこから読める文脈 | Coelacanth's Dream](/posts/2020/10/21/navi10-sku-for-blockchain/) {{< /link >}}
今度は同様の *Navi14* SKU を追加するパッチが投稿されている。  
{{< link >}} [[PATCH 1/2] drm/amdgpu: disable DCN and VCN for Navi14 0x7340/C9 SKU](https://lists.freedesktop.org/archives/amd-gfx/2020-October/055322.html) {{< /link >}}

先日の *Navi10* SKU では、追加された DeviceID:RevisionID が 2つ (`0x731F:0xC6, 0x731F:C7`) であったが、今回の *Navi14* SKU では 1つのみとなる。(`0x7340:0xC9`)  

また、関連するパッチで `nv_is_blockchain_sku` という関数名が `nv_is_headless_sku` に置き換えられたが、意味合いとしてはそう変わらない。  

*Navi14* には *Navi10* 、というよりそれをベースとする **RX 5700/XT** と違い、流通減少の話やブロックチェーン/マイニング向け製品にチップが転用されたことによる在庫切れの話は無いが、  
*Navi14* をベースとしたブロックチェーン/マイニング向け製品もまた予定されているものと思われる。  

*Navi10* と *Navi14* は規模が異なり、*Navi14* の方が小さい。  
総 WGP(2CU) 数は *Navi10* 20基、*Navi14* 12基と半分より少し多いが、  
メモリバス幅と PCIeレーン数は *Navi10* 256-bit に 16レーン、*Navi14* は 128-bit、8レーンと、*Navi14* は *Navi10* の半分となる。  
しかし、マイニングにおいては PCIeレーン数の少なさ (PCIe 帯域の狭さ) はあまり問題とならず、マイニングリグを構築する上では GPU の密度を増やすことが重要となる。  
それには GPU を搭載するマザーボード、CPU、メインメモリの数を減らすことが必要となり、そのためライザーケーブル等を使用し、マザーボード (CPU) と GPU 間は PCIe 1レーンで接続されることが多い。  
また、イーサリウム等では、取引情報の繋がりを記録した DAGファイルを VRAM(GPUメモリ) 上に展開するため、VRAM容量の方が重要とされる。  
その点では、メモリバス幅の広い (=搭載可能なメモリチップが多い) *Navi10* の方がマイニング向けとしては需要が高いと思われる。  
GDDR6メモリ 16Gb(2GB) 8チップを各 32-bit で接続すれば計16GB、16Gb(2GB) 16チップを各 16-bit で接続すれば計32GBの VRAM容量を実現できる。  
*Navi14* をベースとする製品 **RX 5500 XT** 等には、影響が出ていないのはそうしたことが関係しているのかもしれない。  

| | Navi10 | Navi14 |
| :-- | :--: | :--: |
| Shader Engine | 2 | 1 |
| Shader Array | 4 | 2 |
| WGP(CU) | 20(40) | 12(24) |
| Memory Bus Width | 256-bit | 128-bit |
| PCIe Lane (Gen4) | x16 | x8 |

{{< ref >}}

 * [アルトコイン暴落の引き金を引いたイーサリアムのマイニング問題 - イーサリアム・ジャパン](https://ethereum-japan.net/ethereum/ethereum-radeon-rx400-500-mining-problems/)

{{< /ref >}}
