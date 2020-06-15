---
title: "AMDは8 GPUsのシステムを計画している？"
date: 2019-11-26T14:15:14+09:00
draft: false
tags: [ "Radeon", "GCN", "gfx908", "GFX9", "Arcturus" ]
keywords: [ "Radeon", "Arcturus" ]
categories: [ "Hardware", "GPU", "AMD" ]
---

### びみょうな妄想話。  
根拠とするのは以下のパッチ/プルリクエストだ。  

[[PATCH 1/1] drm/amdgpu: Optimize KFD page table reservation](https://lists.freedesktop.org/archives/amd-gfx/2019-November/043149.html)  
[Add -emulate-8-gpu option #5](https://github.com/ROCmSoftwarePlatform/dlrm/pull/5)  

上のパッチは、ページサイズを大きくすることで必要なVRAMのページテーブルを減らすというもので、  
多くのGPUを搭載するシステムにおいて、ページテーブルに割かなければならないメモリ量を大きく減らすことができると説明されている。  
そして気になるのはExampleのところで、  

 > Example: 8 GPUs with 32GB VRAM each + 256GB system memory = 512GB  
 > Old page table reservation per GPU:  1GB  
 > New page table reservation per GPU: 32MB  

とある。  

下のは題通り、8 GPUsでのエミュレート機能を実装し、スケーリングの効果をテストするというもの。  
この動きから、AMDは8 GPUsでのシステムを計画している――と言いたいがそのシステムは既にあったりする。  
[深掘り！「AMD Next Horizon」 - Vega 7nm Deep Dive。7nmのVegaはコンシューマに来ない？ - マイナビニュース](https://news.mynavi.jp/article/20181227-20181227-amd_next_horizon/3)  
Rome + Vega20のお披露目の際、Infinity Fabricでの接続が4枚ずつではあるものの8 GPUsのシステムを構成している。  
VRAM容量もMI60であれば32GB持っているため、上パッチのExampleも満たす。  
最近になってMI50(32GB)も追加されたため、単に上記はそういった製品を使った4 GPUs*2のシステムを、  
より効率良く動かすためと見ることができる。  
というよりそうした見方のが強いだろう。  
その上、AMDは次期エクサスケールスパコン、Frontierの構成において、GPU:CPUが4:1の比率だと発表している。  
[Powering the Exascale Era - AMD](https://www.amd.com/en/products/frontier)  

それでもこんな記事を書いたのは[Arcturus](/tags/arcturus)がSDMAを8個持っているからだ。  
内6個はXGMI（X Global Memory Interconnect）に最適化されている。  
[sdma_v4_0.c#n1668](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/sdma_v4_0.c?h=amd-staging-drm-next#n1668)  
[kfd_device.c#n371](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdkfd/kfd_device.c?h=amd-staging-drm-next#n371)  

そしてNVIDIAはNVLINKにおいて、1 CPUに4 GPUというのは同じでも、GPUが反対側のCPUに接続された1 GPUに、 直接接続する構成を可能としている。  
[ユニファイドメモリの改良によりCPUとGPUの連携を強化 - マイナビニュース](https://news.mynavi.jp/article/pascal-4/)  
[Hot Chips 29 - NVIDIAの最強GPU「Volta」](https://news.mynavi.jp/article/20170915-hotchips29_volta/2)  
さすがに8 GPUsを全対全でやったりはしていない。  
繋いでも所々リンク数を変えている。  

Frontierも4 GPU:1 CPUをクラスターとしているが、クラスター同士の接続に関しては明らかにしていない。  

結論として、要はAMDもGPUがクラスターの違うGPUへの接続をサポートするかもしれない。  
その接続にはInfinity Fabricが使われ、コヒーレントが取れ、256（32*8）GBをローカルメモリとして使える  

――かもしれないという**妄想話**だ。  

<br>
一応、EPYCはPCIe128レーンを持っているから8 GPU:1 CPUが可能ではあるはずだが、  
そうしないのはネットワーク規模がデメリットを無視できないほど大きくなってしまうからとか、ストレージその他も繋ぎたいからとかの理由なのだろうか。  
2ソケットであれば160レーン使えるため余裕は出来る。  

GPU偏重?な構成にしてあるのだから、GPU同士に効率の良い接続方法を用意するのは自然なように思えなくもない。  

（追記 2019/11/30 01:27）  
こんな資料を見つけてしまった。  
![4GPUs System](/image/2019/11/30/4gpus-system.webp)  
ソースはHotchips 2019のプレゼンテーションPDFから。  
[AMD Hot Chips 2019 Presentation - AMD](http://ir.amd.com/static-files/02e2f0b7-dc8b-4432-841a-64578ad441b6)  

ArcturusではVega20の倍近い演算器を持つだろうから、8 GPUも繋ぐ必要はそこまでない。  
1 CPUに8 GPUs繋いでもバランスが崩れて、多いGPUを使うだけのプログラムを書くのが難しなる、とかなのだろうか。  

妄想はほぼ散った……が未来のことはわかるはずもなく、妄想はそれまでの楽しみに過ぎない。  
どんな実物が出てくるかは常に期待して待っている。  
（追記終了）  

（追記 2019/12/03 18:30）  
やっぱ8 GPUs来るんじゃね？  

[drm/amdgpu: Added ASIC specific checks in gfxhub V1.1 get XGMI info](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=14996d7a405047b8bae40dafdaf9c75bf92cdaa4)  
Fronterでは4 GPUsに留まるとしても、Arcturusのダイ自体は8 GPUsをサポートする可能性が濃厚となった。  
（追記終了）  
