---
title: "LLVM に 新たな GPU ID \"gfx90c\" が追加される"
date: 2020-10-30T08:58:37+09:00
draft: false
tags: [ "Lucienne", "Cezanne", "Green_Sardine" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

AMD GPU のコンパイラバックエンド等に採用されている LLVM に、新たな GPU ID *gfx90c* を追加するパッチが投稿された。  
{{< link >}} [⚙ D90419 [AMDGPU] Add gfx90c target](https://reviews.llvm.org/D90419) {{< /link >}}

GPU ID は AMD GPU のマシンタイプ、アーキテクチャ、対応命令セットを示すものであり、コンパイラ等では最適化のためのターゲットIDとして使われている。  
{{< link >}} [AMD GPU の GPU ID は何を意味するか | Coelacanth's Dream](/posts/2020/06/22/amdgpu-gpuid-mean/) {{< /link >}}

## GFX90c {#gfx90c}

*gfx90c* は [Raven2 APU](/tags/raven2) 、[Renoir APU](/tags/renoir) の GPUコア部の *gfx909* と区別されるとあり、*gfx909* をベースにバグ修正、機能追加、命令追加を行なったものと思われる。  
GPU ID の数値は上から、主要なバージョン `pGfxIpMajorVer` 、補助的なバージョン `pGfxIpMinorVer` 、ステッピングを意味する `pGfxIpStepping` となる。  
*gfx909* から進んだとはいえ、そう大きな変更は行なわれてはいないだろう。  

*gfx90c* はある APU の GPUコア部の GPU ID とされる。  
将来的に登場する APU であり、GPUアーキテクチャの世代が *Vega /GFX9* となるものには *Lucienne* 、*Green Sardine* 、*Cezanne* がいる。  
その中の *Lucienne* 、*Green Sardine* は、CPU/GPUアーキテクチャは *Renoir* と同じだが、プロセスは進んだものになると考えていたが、  
**Ryzen 5000 シリーズ** 、**Radeon RX 6000 シリーズ** がその前世代と同じ 7nmプロセスノードで製造されるという話から、実際はアーキテクチャに若干の変更が施されている可能性が出てきた。  
続き、*gfx90c* がそれら APU に採用されているのではないかと考えられる。  

