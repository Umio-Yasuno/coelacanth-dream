---
title: "ROP は何の略"
date: 2020-05-31T20:12:14+09:00
draft: false
# tags: [ "", ]
keywords: [ "", ]
categories: [ "Hardware", "GPU" ]
noindex: false
---

GPU のグラフィック性能を推し量る際の指標として用いられる *ROP* 。  
それが何の略称かはあんまり気にされなかったりする。  

## ROP の役割
*ROP* は GPU のレンダリングパイプライン最終段に位置し、結果をピクセルに対応させ、メモリ(フレームバッファ) に書き出す役割を持つ。  
*AMD GCNアーキテクチャ GPU* では効率を良くするために ROP 相当のユニットを L2cache経由でメモリコントローラにアクセス、書き出しを行なっていた。  
*NVIDIA GPU* も似た構成を取っている。  

この構成によって、製品の差別化のため搭載メモリチップ数を減らす(=一部メモリコントローラを無効化)と、合わせて L2cache、ROP も一部無効化される。  
それだけではないとも思うが、このことが性能に関係する印象を強め、指標に用いられやすくなったのかもしれない。  

また、MSAAやEQAAといったアンチエイリアス処理を行なうための処理ユニットも ROP に含まれる。  

## ROP のフルネーム
ROP のフルネームはあやふやだ。  
*Render OutPut* または *Render OutPut Unit* とされたり、*Raster OPeration* の略だったり、少し変わって *Raster Operations Pipeline* だったり、またさらに少し変えた *Raster Operation Processor* だったりする。  

NVIDIA の各資料から見るに、2004年時点では *Raster Operation* [^1][^3] と *Raster Operation Unit* [^2]の両方を使っていた。  

[^1]: [ar10_po_v01c bjs1.fm - PO_GoForce_3D.pdf](https://download.nvidia.com/developer/Handheld_SDK/PO_GoForce_3D.pdf)
[^2]: [Evolution_of_GPUs.ppt - English_Evolution_of_GPUs.pdf](https://download.nvidia.com/developer/presentations/2004/Perfect_Kitchen_Art/English_Evolution_of_GPUs.pdf#page=8)
[^3]: [Chapter 28. Graphics Pipeline Performance | NVIDIA Developer](https://developer.nvidia.com/gpugems/gpugems/part-v-performance-and-practicalities/chapter-28-graphics-pipeline-performance)

2007年の NVIDIA Teslaブランドの立ち上げ後[^4]に、2008年の HotChips19 に合わせて発表したと思われる資料では *Raster Operation Processor* の略としており、[^5]  
この影響か、その後も一部メディアや GPUコンピューティング関連の資料では *Raster Operation Processor* が使われている。  
しかし、NVIDIA が ROP をその略称としたのはそれきりである。  
単に見落としてるかもしれないが、少なくとも、検索エンジンを駆使し、資料を漁ったが他に見つからなかった。  

[^4]: [NVIDIA、G80ベースのHPC向けGPU「Tesla」](https://pc.watch.impress.co.jp/docs/2007/0621/nvidia.htm)
[^5]: [micr-28-02-lind 39..55 - gpu.pdf](https://people.cs.umass.edu/~emery/classes/cmpsci691st/readings/Arch/gpu.pdf)

2015年には *Render Output Unit* の略としている。[^6]  
Raster から離れたのは、ROP とは別に *Raster Unit* 、*Raster Engine* があるため混同を避けたかったのだろう。  

[^6]: [Life of a triangle - NVIDIA's logical pipeline | NVIDIA Developer](https://developer.nvidia.com/content/life-triangle-nvidias-logical-pipeline)

そして、今日では ROP は記号と化している。  
[NVIDIA-Turing-Architecture-Whitepaper.pdf](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/technologies/turing-architecture/NVIDIA-Turing-Architecture-Whitepaper.pdf) を見ても、わざわざ ROP が何の略かは説明していない。  
これは AMD もそうだ。それに AMD GPU は、ROP 4基相当の RB(Render Backend)を単位としているため、公式のスペックで ROP数を出すのは競合との比較という意義があるのではないかと思う。  

Intel はどうなのかというと、Slice Common が ROP としての機能を含んでおり、その中で結果をメモリ出力するユニットを PBE(Picel Backend) と呼んでいる。[^7]  
わかりやすさを重視してか、NVIDIA、AMDも ROP を *Pixel Engine* 、*Pixel Unit* とした資料もある。[^8][^9]  

[^7]: [the-architecture-of-intel-processor-graphics-gen11-r1new.pdf](https://software.intel.com/content/dam/develop/public/us/en/documents/the-architecture-of-intel-processor-graphics-gen11-r1new.pdf#page=17)
[^8]:[ Windowing System on a 3D Pipeline - Windowing_System_on_a_3D_Pipeline_Slides.pdf](http://developer.download.nvidia.com/assets/gamedev/docs/Windowing_System_on_a_3D_Pipeline_Slides.pdf)
[^9]: [M&E GBGH - HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf](https://www.hotchips.org/wp-content/uploads/hc_archives/hc29/HC29.21-Monday-Pub/HC29.21.10-GPU-Gaming-Pub/HC29.21.120-Radeon-Vega10-Mantor-AMD-f1.pdf)

一般的に見てもその方がわかりやすいと個人的には思うが、今後 Intel が本格的に ハイパフォーマンスGPU へ進出すれば、  
その時は競合 NVIDIA、AMD を意識し、記号として ROP を使い始めるかもしれない。  

{{< ref >}}

 * [Render output unit - Wikipedia](https://en.wikipedia.org/wiki/Render_output_unit)
 * [ASCII.jp：詳細が判明したRDNAの内部構造　AMD GPUロードマップ (2/4)](https://ascii.jp/elem/000/001/877/1877938/2/)
 * [NVIDIAのGPU開発部門責任者に聞く「GeForce GTX 970のVRAM 3.5GB問題」 - 4Gamer.net](https://www.4gamer.net/games/274/G027467/20150130109/)

{{< /ref >}}
