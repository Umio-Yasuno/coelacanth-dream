---
title: "Raja Koduri氏が Xe-HPC のチップ構成とパッケージを公開"
date: 2021-01-27T06:08:25+09:00
draft: false
tags: [ "Xe-HPC", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Intel の現チーフアーキテクトである [Raja Koduri](https://newsroom.intel.com/biography/raja-m-koduri/) 氏が自身の Twitterアカウントにて、**{{< xe class="hpc" >}}** が電源投入が可能な段階にあることを、**{{< xe class="hpc" >}}** のパッケージ画像と一緒にツイートした。  
公開されたパッケージにはヒートスプレッダが無く、チップ構成も分かるものとなっている。  

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Xe HPC ready for power on! <br>7 advanced silicon technologies in a single package<br>Silicon engineers dream<br>Thing of beauty <a href="https://twitter.com/intel?ref_src=twsrc%5Etfw">@intel</a> <a href="https://t.co/RF8Prsy05f">pic.twitter.com/RF8Prsy05f</a></p>&mdash; Raja Koduri (@Rajaontheedge) <a href="https://twitter.com/Rajaontheedge/status/1354103878426324994?ref_src=twsrc%5Etfw">January 26, 2021</a></blockquote>


## {{< xe class="hpc" >}} のチップ構成

{{< xe class="hpc" >}} は *Base Tile* 、 *Compute Tile* 、 *Rambo Cache Tile* 、 *{{< xe >}} Link I/O Tile* 、それと HBM2メモリで構成され、それらは 3Dパッケージング技術 Forveros、Forverosパッケージを複数組み合わせる Co-EMIB によってパッケージングされる。  
より詳細な構成を言えば、*Base Tile* 、 *Compute Tile* 、 *Rambo Cache Tile* を Forveros技術で積層、それに HBM2メモリ、 *{{< xe >}} Link I/O Tile* が EMIB で接続される。  
そして、それらをもう 1セット Co-EMIB で互いを接続したものが、コードネーム **Ponte Vecchio** 1パッケージとなる。  
また、各チップ (Tile) はそれぞれ別のプロセスで製造され、  
*Base Tile* は Intel 10nm SuperFin、 *Compute Tile* は Intel の次世代プロセス {{< comple >}} 恐らくは 7nm {{< /comple >}} と外部ファウンダリ、*Rambo Cache Tile* は Intel 10nm Enhanced SuperFin、 *{{< xe >}} Link I/O Tile* は外部ファウンダリのプロセスのみを採用している。  
Intel 7nmプロセスは 2023年頃に製品が出てくる予定にあるため、画像の *Compute Tile/Xe HPC* は外部ファウンダリで製造されたものである可能性が高い。  
また、{{< xe >}} MF については、恐らく *Base Tile* に含まれているものと思われる。  

{{< figure src="/image/2021/01/27/xe-building.webp" caption="画像出典: [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)" >}}

Raja氏が公開した画像から、各チップの配置を推測したものが以下。  
一見 HBM2メモリの大きさが合ってないように思えるが、これは画像が真上ではなくそこから少し右から撮影されたからで、縦幅は一致した。  

{{< figure src="/image/2021/01/27/xe-hpc-package.webp" title="Xe-HPC / Ponte Vecchio" caption="画像元: <https://twitter.com/Rajaontheedge/status/1354103878426324994>" >}}

{{< xe class="hpc">}} と HBM2メモリに間にあるチップを Rambo Cache と判断したのは、Intel は以前 XeMF (Xe Memory Fabric) と Rambo Cache がセットであるように発表しており、それでいて Rambo Cache は Intel 10nm eSF で製造され、別チップとなる。そして、XeMF は 8x {{< xe class="hpc">}}用ともう片方のタイルとの接続用とで計 6基確認でき、Rambo Cache の数と一致するため。  
配置については過去に発表された時の CG 等を元に推測したが、それがどこまで実物に忠実かは定かでない。  
Rambo Cache の容量等も明かされていないが、帯域については、GPU内のキャッシュ (SRAM) と HBM2メモリのようなインパッケージメモリとの間に位置し、2つのギャップを埋める程であることが発表されている。[^rambo-cache]  

[^rambo-cache]: [NVMe® Technology Powering the Connected Universe](https://nvmexpress.org/wp-content/uploads/NVMe-Technology-Powering-the-Connected-Universe.pdf)

{{< xe class="hpc" >}} の規模もまた不明だが、アーキテクチャの特徴として HPC、機械学習に最適化されており、{{< xe class="lp"  >}}アーキテクチャには搭載されていない FP64ユニットと Matrix Extension (XMX) が {{< xe class="hpc" >}} では搭載されている。  
マルチメディアエンジンの有無についてもこれまでに触れられていないが、**AMD MI100** が機械学習時のオブジェクト検出等を想定してエンジンを搭載していたことを考えると、{{< xe class="hpc"  >}} でも最低限搭載している可能性はある。  

エクサスケールスパコン Aurora は、ノードを *Sapphire Rapids* CPU 2基、*Ponte Vecchio* GPU 6基で構成し、今年 2021年に納入予定にある。  
*Sapphire Rapids* のパッケージ等はまだ公開されていないが、Linux Kernel へのパッチにログの一部があり、そちらも既に Intelラボ内部では動作していると思われる。[^idxd-spr]  

[^idxd-spr]: [dmaengine: idxd: Fix list corruption in description completion](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/commit/?h=next-20210125&id=16e19e11228ba660d9e322035635e7dcf160d5c2)


{{< ref >}}

 * <https://s21.q4cdn.com/600692695/files/doc_presentations/2019/11/DEVCON-2019_16x9_v13_FINAL.pdf>
 * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)

{{< /ref >}}
