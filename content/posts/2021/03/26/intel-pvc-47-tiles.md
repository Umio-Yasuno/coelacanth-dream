---
title: "47個のタイルで構成される Intel Ponte Vecchio"
date: 2021-03-26T12:46:06+09:00
draft: false
tags: [ "Xe-HPC", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

以前、Raja Koduri氏によってパッケージとチップ外観が公開された **Intel Ponte Vecchio** GPU だが、今回 47個のタイルで構成されることが新たに明かされた。  
{{< link >}} [Raja Koduri氏が Xe-HPC のチップ構成とパッケージを公開 | Coelacanth's Dream](/posts/2021/01/27/xe-hpc-package/) {{< /link >}}

 * {{< youtube title="Ponte Vecchio, Intel’s XPU for Exascale Computing and AI" id="JzbN1IOAcwY" >}}

Raja Koduri氏は 47個のタイルの内訳について、以下のように説明している。  
以前は 41個のタイルとしていたらしいが、これは HBM2eメモリを個々のタイルとして計上していなかったとのこと。**Ponte Vecchio** は 2つの Base Tile を搭載し、そこに HBM2eメモリが 4タイル (スタック) EMIB技術を用いて接続される。恐らく、以前は Base Tile にそれぞれ HBM2eメモリが接続される、ということで 2タイルとして計上していたのだろう。  

{{< twemb >}}
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Andreas, in Jan we didn&#39;t account the HBM&#39;s as individual tiles. That&#39;s the main difference. <br>16 Xe HPC (internal/external)<br>8 Rambo (internal)<br>2 Xe Base (internal)<br>11 EMIB (internal)<br>2 Xe Link (external)<br>8 HBM (external) <a href="https://t.co/uA0jAs8QDo">https://t.co/uA0jAs8QDo</a></p>&mdash; Raja Koduri (@Rajaontheedge) <a href="https://twitter.com/Rajaontheedge/status/1374721307879768064?ref_src=twsrc%5Etfw">March 24, 2021</a></blockquote>
{{< /twemb >}}

また、氏はIntel社内ではバンプピッチ 55μm以下の高密度実装を必要とするシリコンを「タイル」、標準的なパッケージに実装されるシリコンを「チップレット」と区別していると伝えている。[^tile-chiplet]  
バンプピッチ 55μm以下の高密度実装には EMIB (*Embedded Multi-die Interconnect Bridge*)、Foveros 技術が該当する。  

{{< figure src="/image/2021/03/26/intel-package.webp" title="Intel Packaging Technology" caption="画像出典: [Video: Architecture Day 2020 (Event Replay) | Intel Newsroom](https://newsroom.intel.com/video-archive/video-architecture-day-2020-event-replay/)" >}}


[^tile-chiplet]: [Raja KoduriさんはTwitterを使っています 「Internally we also are switching to "tile" nomenclature to differentiate silicon that needs to be packaged with advanced high density packaging (55 micron bump pitches and below) v/s silicon "chiplets" that can be packaged with standard packaging」 / Twitter](https://twitter.com/Rajaontheedge/status/1374721309372948492)


## 47個のタイル {#tiles}

以下は 2021/01/26 に Raja Koduri氏が公開した **Ponte Vecchio** のパッケージ画像を元に、チップ構成を推測した画像。  
 
{{< figure src="/image/2021/01/27/xe-hpc-package.webp" title="Xe-HPC / Ponte Vecchio (v1)" caption="画像元: <https://twitter.com/Rajaontheedge/status/1354103878426324994>" >}}

今回の Raja氏による内訳の説明と合わせれば、まず巨大な長方形をした *{{< xe >}} Base* タイルが 2個、 *{{< xe class="hpc" >}}* タイルがそれぞれに 8個、Foveros技術によって積層されている。  
また、*{{< xe >}} Base* には EMIB技術により HBM2eメモリ 4個と *{{< xe >}} Link* タイルが 1個それぞれ接続されている。  
それだと 10個の EMIBチップとなるが、2個の *{{< xe >}} Base* タイル同士もまた EMIB で接続されるため、それを合わせて 11個となる。*{{< xe >}} Base* のような Foveros技術を用いるタイル同士には Co-EMIB という名称が使われる。  
*Rambo Cache* タイルについては以前、*{{< xe >}} Base* の端に位置するタイルがそうではないかと推測したが、各 4個ということから、 *{{< xe class="hpc" >}}* の間に位置するタイルが *Rambo Cache* タイルだと考えられる。過去の資料を見返してみれば、完全なイメージではあるがそのように説明しているものがあった。[^rambo-image]  
*Rambo Cache* タイルもまた Foveros技術によって *{{< xe >}} Base* タイルに積層される。  
そのタイルからは思っていたよりも小さい印象を受けるが、容量は未公開であり、GPU内のキャッシュ (SRAM) とパッケージ内メモリ (HBM2eメモリ) の間を埋めるメモリ帯域であることだけが発表されている。  
端に位置するのはパッディング、ダミーダイの可能性が考えられる。ダミーダイと言っても役割が無い訳ではなく、全体を長方形に整え、主に *{{< xe class="hpc" >}}* タイルから発生する熱を逃し、熱密度を緩和することで冷却を容易にするなどの役割がある。  

[^rambo-image]: <https://s21.q4cdn.com/600692695/files/doc_presentations/2019/11/DEVCON-2019_16x9_v13_FINAL.pdf#page=64>

上記を元にアップデートしたのが以下。  

{{< figure src="/image/2021/03/26/xe-hpc-package_v2.webp" title="Xe-HPC / Ponte Vecchio (v2)" caption="画像元: <https://twitter.com/Rajaontheedge/status/1354103878426324994>" >}}


 * **Ponte Vecchio**
    * Base Tile (Intel 10nm SF)
        * |+ Foveros +| 8x {{< xe class="hpc" >}}/Compute Tile (Intel Next Gen & External)
        * |+ Foveros +| 4x Rambo Cache (Intel 10nm eSF)
    * | EMIB | 4x HBM2 (External)
    * | EMIB | 1x {{< xe >}} Link I/O Tile (EMIB) (External)
 * || Co-EMIB ||
    * Base Tile (Intel 10nm SF)
        * |+ Foveros +| 8x {{< xe class="hpc" >}}/Compute Tile (Intel Next Gen & External)
        * |+ Foveros +| 4x Rambo Cache (Intel 10nm eSF)
    * | EMIB | 4x HBM2 (External)
    * | EMIB | 1x {{< xe >}} Link I/O Tile (EMIB) (External)

{{< ref >}}
 * <https://s21.q4cdn.com/600692695/files/doc_presentations/2019/11/DEVCON-2019_16x9_v13_FINAL.pdf>
 * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)
{{< /ref >}}
