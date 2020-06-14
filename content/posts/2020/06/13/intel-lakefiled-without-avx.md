---
title: "Intel、Lakefiled を正式発表 & AVX命令には非対応？"
date: 2020-06-13T09:46:31+09:00
draft: false
tags: [ "Tremont", "Lakefield", "Gen11" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

Intel は 2020/06/10付で、3Dパッケージング技術 **Foveros** を用いたハイブリッドプロセッサ、*Lakefield* を正式発表した。  
{{< link >}}[Intel Hybrid Processors: Uncompromised PC Experiences for Innovative Form Factors Like Foldables, Dual Screens | Intel Newsroom](https://newsroom.intel.com/news/intel-hybrid-processors-uncompromised-pc-experiences-innovative-form-factors-foldables-dual-screens/){{< /link >}}
{{< link >}}[Intel® Core™ Processors with Intel® Hybrid Technology Brief](https://www.intel.com/content/www/us/en/products/docs/processors/core/core-processors-with-hybrid-technology-brief.html){{< /link >}}

Intelプロセッサでは初のネイティブで 2つのディスプレイパイプを持つとのことだが、これはざっくりと、eDP出力が 2つあるという認識で合っているのだろうか。  

また、メディア機能では EU/シェーダーによる動画エンコード、HDRトーンマッピングのようなポストプロセッシングにも対応しており、  
同じ [Gen11](/tags/gen11)GPU でもそれらが削られていた *Elkhart /Jasper Lake* とは違い、*Ice Lake* 同等の機能を備えているようだ。  

SKUは **i5-L16G7** 、**i3-L13G4** の 2種類となる。  

| Lakefield | i5-L16G7 | i3-L13G4 |
| :-- | :---: | :---: |
| Core/Thread | 5/5 | 5/5 |
| Base Clock | 1.4 GHz | 0.8 GHz |
| Max Single Core Clock | 3.0 GHz | 2.8 GHz |
| Max All Core Clock | 1.8 GHz | 1.3 GHz |
| GPU EU | 64 | 48 |
| Base GPU Clock | 0.2 GHz | 0.2 GHz |
| Max GPU Clock | 0.5 GHz | 0.5 GHz |
| TDP | 7 W | 7 W |

## AVX命令には非対応か
*Lakefield* で一番気になるのがここで、  
まず *Lakefield* が 1-Core 持つ高性能な *Sunny Cove* コアは、*Ice Lake* のコアと同じだ。  
だが、**i5-L16G7** の製品ページを見ると、`Instruction Set Extensions` に AVX という文字列は無い。[^1]  
`AVX512-VNNI` 命令による高速な推論、**Intel DL Boost** が *Ice Lake* の特徴の1つだったが、やはりそれも製品ページには記載が無く、  
代わりか、GPU を用いた推論において **i5-L16G7** は **i7-8500Y** と比べて 2倍のスループットで実行できることがリリースではアピールされている。  

[^1]: [Intel® Core™ i5-L16G7 Processor (4M Cache, up to 3.0GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/202777/intel-core-i5-l16g7-processor-4m-cache-up-to-3-0ghz.html#tab-blade-1-0-7)

もしも、AVX命令の対応を潰しているのだとしたら、*Lakefield* は命令別のスケジューリングには対応していないということになる。  
そうなると、AVX命令以外にも、*Sunny Cove* コア、*Tremont* コアの両方が対応していない命令は、*Lakefield* では外されている可能性が高い。  
つまり、CPUコアは非対称であっても、命令レベルでは対象、となる感じか。  
*Lakefield* のスケジューリングはあくまで負荷とシングルスレッド、マルチスレッドに応じたものなのだろうか。  

その点、ハイブリッドプロセッサとして惜しい気がするが、*Lakefield* は第1世代であり、  
次世代のハイブリッドプロセッサではより高度なスケジューリングが実装されていることを期待したい。  

{{< ref >}}

 * <https://www.hotchips.org/hc31/HC31_2.10_LKF_HC_2019_Final_v7.pdf>

{{< /ref >}}
