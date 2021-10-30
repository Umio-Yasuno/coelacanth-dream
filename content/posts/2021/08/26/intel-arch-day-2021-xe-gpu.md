---
title: "Intel Architecture Day 2021 個人的まとめ　―― 用語が整理された Xe GPU"
date: 2021-08-26T18:53:10+09:00
draft: false
tags: [ "Xe-HPC", "DG2", "Xe-HPG", "Gen12" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

相変わらず地味な話を取り上げていく。  

Intel Architecture Day 2021 にて、ゲーミング向け Intel GPU アーキテクチャ *{{< xe class="hpg" >}}* と、コンピュート性能に特化した *{{< xe class="hpc" >}}* の一部概要と新たなブロック図が公開された。  

 * [Intel Architecture Day 2021](https://www.intel.com/content/www/us/en/newsroom/resources/press-kit-architecture-day-2021.html)
    * [Presentation Deck: Intel Architecture Day 2021](https://download.intel.com/newsroom/2021/client-computing/intel-architecture-day-2021-presentation.pdf)

{{< pindex >}}

 * [用語の再整理](#cleanup)
    * [{{< xe class="hpg" >}}](#xe-hpg)
    * [{{< xe class="hpc" >}}](#xe-hpc)
        * [Rambo Cache = L2キャッシュ?](#rambo)
{{< /pindex >}}

## 用語の再整理 {#cleanup}
### {{< xe class="hpg" >}} {#xe-hpg}

まず {{< xe class="hpg" >}} GPU の単位となる {{< xe >}}-Core には、Vector Engine (256-bit) 16基、Matrix Engine (1024-bit) 16基、ロード/ストアユニット、L1キャッシュと命令キャッシュ、SLM (Shared Local Memory) が搭載される。  
今回新たに出てきた Vector Engine と Matrix Engine (XMX, {{< xe >}}Matrix eXtensions) だが、その 2つを統合したものが従来の Intel GPU における EU (Execution Unit) に相当すると考えられる。  

{{< figure src="../xe-hpg-core.webp" title="Intel Xe-Core" caption="画像出典: [Presentation Deck: Intel Architecture Day 2021](https://download.intel.com/newsroom/2021/client-computing/intel-architecture-day-2021-presentation.pdf)" >}}

EU は Thread control と演算ユニット、レジスタファイルで構成される。  
*Tiger Lake* や *Rocket Lake* 、*DG1* 等で採用されている *Gen12LP/{{< xe class="lp" >}} アーキテクチャ* では、8-wide の FP32/INT32 を処理する演算パイプラインを 1つ搭載していた。  
つまり *{{< xe class="lp" >}}* では EU あたり 256-bit のスループットを持っていたと言え、同時に *{{< xe class="hpg" >}}* の EU あたりの FP32/INT32 演算性能は *{{< xe class="lp" >}}* と変わらないと考えられる。  
また、*{{< xe class="lp" >}}* では、Thread Control が EU 2基をペアとして扱う機能を持っていたが、{{< xe >}}-Core のイメージ図を見ると それぞれ 2基の Vector Engine と Matrix Engine が密接しており、*{{< xe class="hpg" >}}* でも同機能が採用されているのだろう。  

L1データキャッシュは *{{< xe class="lp" >}}* から追加されたものだが、帯域、サイズ等は公開されてこなかった。  
それが今回、ソフトウェアから L1キャッシュかローカルメモリ (SLM) のどちらかに設定可能なものだということが明かされた。  
ただ、このことは *{{< xe class="hpc" >}}* の発表のタイミングで説明されていたことで、最近の NVIDIA GPU のように L1データキャッシュとローカルメモリとで分割可能なのか、*{{< xe class="lp/hpg" >}}* も同様か、といった点は不明。  

以前 *{{< xe class="lp" >}}* の L1データキャッシュを有効化するパッチが Intel GPU のオープンソースドライバーに投稿されており、パッチのコメントによれば Unreal Engine 4 Shooter Demo で 11.84% 、Doom (2016) で 11.40% の性能向上が確認されたとしている。  
{{< link >}} [Intel Gen12 GPU で増設された L1データキャッシュの効果 | Coelacanth's Dream](/posts/2020/10/13/tgl-gen12-l1cache/) {{< /link >}}

*{{< xe class="hpg" >}}* では {{< xe >}}-Core と同数のレイトレーシングユニットとテクスチャサンプラーを持ち、以前はそれら 3つを合わせて Sub-slice(s) と呼称していた。  
全体で共有するキャッシュについても、以前は L3キャッシュと呼んでいたが、今回で L2キャッシュと再定義された。  
以前の Intel GPU で L2キャッシュがどこに位置していたのかというと、サンプラーが持つリードオンリーキャッシュであり、AMD/NVIDIA GPU アーキテクチャと比較した時に、キャッシュ階層とその意味が異なっていた。  
今回それが整理されたことで他の GPU アーキテクチャとの比較がしやすくなったが、今後 Intel GPU を広くに展開し、推しだしていく上では必要なことだったのかもしれない。  


### {{< xe class="hpc" >}} {#xe-hpc}

*{{< xe class="hpc" >}}* の {{< xe >}}-Core は、Vector Engine (512-bit) 8基、Matrix Engine (4096-bit) 8基、ロード/ストアの帯域はクロックあたり 512B、L1キャッシュ/SLM は 512KB 搭載している。  
*{{< xe class="hpg" >}}* と比較すると、Vector/Matrix Engine はどちらもスループットが増やされているが、{{< xe >}}-Core あたりのエンジン数は減っており、また Matrix Engine の比率が大きくなっている。  
ロード/ストア性能、L1キャッシュ/SLM のサイズは、*{{< xe class="hpg" >}}* では公開されていなかったため比較できないが、参考までに *{{< xe class="lp" >}}* の場合を書くと、  
*{{< xe class="lp" >}}* では Subslice あたりのロード/ストアは 128B/clk、SLM は 128KB となっていた。  

| | {{< xe class="lp" >}} | {{< xe class="hpg" >}} | {{< xe class="hpc" >}} |
| :-- | :--: | :--: | :--: |
| Vector Engine | 256-bit? | 256-bit | 512-bit |
| VE per (Subslice or {{< xe >}}-Core) | 16 | 16 | 8 |
| Matrix Engine | N/A | 1024-bit | 4096-bit |
| Load/Store (SLM) | 128B? | ? | 512B |

複数の Subslice ({{< xe >}}-Core + Fixed function) で構成される Slice は、*{{< xe class="hpc" >}}* では Subslice 16基の仕様となっている。  
**Ponte Vecchio** では *{{< xe class="hpc" >}}* を採用し、実装される Compute Tile は 8基搭載する訳だが、  
Compute Tile あたりの {{< xe >}}-Core は 4基と説明されており、Compute Tile 2基で {{< xe class="hpc" >}} Slice を構成、1つの Hardware Context を持つこととなる。  

Compute Tile は TSMC N5プロセスで製造される。以前より製造プロセスは Intel Next Gen & External になると説明されてきたが、TSMC版が先行することとなる。  
ただ今回は Intel Next Gen の部分には触れておらず、提供開始時期の問題もあるのだろうが、本当に Intel Next Genプロセス版が出てくるのかが怪しくなってきた。  

#### Rambo Cache = L2キャッシュ? {#rambo}
**Ponte Vecchio** 、*{{< xe class="hpc" >}}* では Rambo Cache なるものが採用されることが、初出から発表されていたが、明言は避けられているが今回、144MB という L2キャッシュであるような話が出てきた。  

{{< figure src="../xe-hpc-base-tile.webp" title="Intel Ponte Vecchio Base Tile" caption="画像出典: [Presentation Deck: Intel Architecture Day 2021](https://download.intel.com/newsroom/2021/client-computing/intel-architecture-day-2021-presentation.pdf)" >}}

Rambo Cache Tile は Intel 7 (Intel 10nm eSF) プロセスで製造され、**Ponte Vecchio** では Base Tile あたりに 3D積層技術 Foveros を用いて 4基搭載される。4基というのは {{< xe class="hpc" >}} Slice (Compute Tile 2基) の数と一致する。  

Rambo Cache = L2キャッシュ とは明言されていないが、Compute Tile には L1キャッシュ/SLM 4MB までしか搭載しておらず、全体で共有するキャッシュは搭載していないため、Rambo Cache が L2キャッシュだと考えられる。  
144MB というサイズはキャッシュとして巨大であり、NVIDIA A100 GPU が持つ 40MB の 3.6倍にもなる。Rambo Cache という独自の名前をつけるのも不自然ではないように思う。  
AMD の RDNA 2 GPU、*Sienna Cichlid (Navi21)* も 128MB という巨大な L3キャッシュを持つが、そちらも Infinity Cache といったマーケティングネームがつけられている。  


{{< ref >}}
 * [HC32 Archive -](https://hc32.hotchips.org/)
    * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://hc32.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)
 * [Fact Sheet: Intel Unveils Biggest Architectural Shifts in a Generation - intel-architecture-day-2021-fact-sheet.pdf](https://download.intel.com/newsroom/2021/client-computing/intel-architecture-day-2021-fact-sheet.pdf)
{{< /ref >}}
