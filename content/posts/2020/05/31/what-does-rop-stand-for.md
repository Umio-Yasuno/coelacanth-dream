---
title: "ROP は何の略"
date: 2020-05-31T20:12:14+09:00
draft: false
# tags: [ "Database", ]
keywords: [ "", ]
categories: [ "Hardware", "GPU", "Database" ]
noindex: false
---

GPU のグラフィック性能を推し量る際の指標として用いられる *ROP* 。  
その役割はともかく、何の略称かはあんまり気にされなかったりする。  

## ROP の役割
*ROP* は GPU のレンダリングパイプライン最終段に位置し、結果をピクセルに対応させ、メモリ(フレームバッファ) に書き出す役割を持つ。  
他にも、MSAAやEQAAといったアンチエイリアス処理を行なうための処理ユニットも ROP に含まれる。  

*AMD GCNアーキテクチャ GPU* では効率を良くするために ROP 相当のユニットを L2cache経由でメモリコントローラにアクセス、書き出しを行なう。  
*NVIDIA GPU* も似た構成を取っている。  

この構成によって、製品の差別化のため搭載メモリチップ数を減らす(=一部メモリコントローラを無効化)と、合わせて L2cache、ROP も一部無効化される。  
それだけではないとも思うが、このことが性能に関係する印象を強め、指標に用いられやすくなったのかもしれない。  
しかし、*AMD RDNAアーキテクチャ GPU* からは新たに増設された L1cache に ROP を直接接続するようになったため、メモリコントローラを一部無効化しても ROP をそれに応じて無効化しなくてもいいようになった。  
実際に、メモリコントローラを一部無効化し、フル構成となる **AMD Radeon RX 5700シリーズ** の 256-bit から 192-bit に減った **RX 5600シリーズ** の ROP数は変わらず 64基となっている。  
そのため、*AMD RDNAアーキテクチャ GPU* においては、ROP数を指標に使うにはズレが生じることを考慮しなければいけない。  
(ただ L2cache は無効化される。また、する必要がなくとも今後製品差別化のため ROP を無効化することがあり得るかもしれない。)  

## ROP のフルネーム
ROP のフルネームはあやふやだ。  
*Render OutPut* または *Render OutPut Unit* とされたり、*Raster OPeration* の略だったり、少し変わって *Raster Operations Pipeline* だったり、またさらに少し変えた *Raster Operation Processor* だったりする。  

NVIDIA の各資料から見るに、2004年時点では *Raster OPeration* [^1][^3] と *Raster OPeration Unit* [^2]の両方を使っていた。  


[^1]: [ar10_po_v01c bjs1.fm - PO_GoForce_3D.pdf](https://download.nvidia.com/developer/Handheld_SDK/PO_GoForce_3D.pdf)
[^2]: [Evolution_of_GPUs.ppt - English_Evolution_of_GPUs.pdf](https://download.nvidia.com/developer/presentations/2004/Perfect_Kitchen_Art/English_Evolution_of_GPUs.pdf#page=8)
[^3]: [Chapter 28. Graphics Pipeline Performance | NVIDIA Developer](https://developer.nvidia.com/gpugems/gpugems/part-v-performance-and-practicalities/chapter-28-graphics-pipeline-performance)

2007年の NVIDIA Teslaブランドの立ち上げ後[^4]に、2008年の HotChips19 に合わせて発表したと思われる資料では *Raster Operation Processor* の略としている。[^5]  

2007年の **NVIDIA GeForce 8800** の資料では先の *Raster Operation Processor* と *Raster Operation Pipeline* の両方を使っている。[^10]  
何だかややこしいが、*Raster Operation Processor* は ROPのユニット全体を示し、*Raster Operation Pipeline* は ROP内部のパイプラインを示しているのだろうか？  
この2つは ROP の名称としてはしっくり来るためか、公式以外のメディアやユーザーコミュニティ内で ROP を解説する際はどちらかが名称に使われることが多い。  

[^4]: [NVIDIA、G80ベースのHPC向けGPU「Tesla」](https://pc.watch.impress.co.jp/docs/2007/0621/nvidia.htm)
[^5]: [micr-28-02-lind 39..55 - gpu.pdf](https://people.cs.umass.edu/~emery/classes/cmpsci691st/readings/Arch/gpu.pdf)
[^10]: [HC19.20.210.The NVIDIA GeForce 8800 GPU.v3.ppt - HC19.02.01.pdf](https://www.hotchips.org/wp-content/uploads/hc_archives/hc19/2_Mon/HC19.02/HC19.02.01.pdf)

2015年には *Render Output Unit* の略としている。[^6]  
Raster から離れたのは、ROP とは別に、それより前のレンダリングパイプライン段でラスタライザ処理を行なう *Raster Unit /Raster Engine* があるため混同を避けたかったのだろう。  

[^6]: [Life of a triangle - NVIDIA's logical pipeline | NVIDIA Developer](https://developer.nvidia.com/content/life-triangle-nvidias-logical-pipeline)

そして、今日では ROP は記号と化している。  
[NVIDIA-Turing-Architecture-Whitepaper.pdf](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/technologies/turing-architecture/NVIDIA-Turing-Architecture-Whitepaper.pdf) を見ても、わざわざ ROP が何の略かは説明していない。  
2015年時点でも、*Render Output Unit* としているのに、ROP Unit と書いていたりと、既に記号化が進んでいた。  
これは AMD もそうで、そも *AMD GPU* は ROP 4基相当の RB(Render Backend)を単位としているのに、  
公式のスペックで相当ROP数を出すのは、競合との比較のためという意味合いがあるのではないかと思う。  


Intel はどうなのかというと、 Slice Common が ROP としての機能を含んでおり、その中で結果をメモリ出力するユニットを PBE(Picel Back-End) と呼んでいる。[^7]  
関連して、わかりやすさを重視してか、NVIDIA、AMDも ROP を *Pixel Engine /Pixel Unit* としている資料もある。[^8][^9]  

[^7]: [the-architecture-of-intel-processor-graphics-gen11-r1new.pdf](https://software.intel.com/content/dam/develop/public/us/en/documents/the-architecture-of-intel-processor-graphics-gen11-r1new.pdf#page=17)
[^8]:[ Windowing System on a 3D Pipeline - Windowing_System_on_a_3D_Pipeline_Slides.pdf](http://developer.download.nvidia.com/assets/gamedev/docs/Windowing_System_on_a_3D_Pipeline_Slides.pdf)
[^9]: [M&E GBGH - HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf](https://www.hotchips.org/wp-content/uploads/hc_archives/hc29/HC29.21-Monday-Pub/HC29.21.10-GPU-Gaming-Pub/HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf)

個人的にもその方がまだ直感的にわかりやすいと思うが、今後 Intel が本格的に ハイパフォーマンスGPU へ進出すれば、  
その時は競合 NVIDIA、AMD を意識し、Intel もまた記号として ROP を使い始めるかもしれない。  

{{< ref >}}

 * [Render output unit - Wikipedia](https://en.wikipedia.org/wiki/Render_output_unit)
 * [ASCII.jp：詳細が判明したRDNAの内部構造　AMD GPUロードマップ (2/4)](https://ascii.jp/elem/000/001/877/1877938/2/)
 * [NVIDIAのGPU開発部門責任者に聞く「GeForce GTX 970のVRAM 3.5GB問題」 - 4Gamer.net](https://www.4gamer.net/games/274/G027467/20150130109/)
 * [AMD Radeon™ RX 5600 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5600-xt#product-specs)

{{< /ref >}}
