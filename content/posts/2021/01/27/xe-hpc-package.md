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

{{< xe class="hpc" >}} は Base Tile、Compute Tile、Rambo Cache Tile、{{< xe >}} Link I/O Tile、それと HBM2メモリで構成され、それらは 3Dパッケージング技術 Forveros、Forverosパッケージを複数組み合わせる Co-EMIB によってパッケージングされる。  
より詳細な構成を言えば、Base Tile、Compute Tile、Rambo Cache Tile を Forveros技術で積層、それに HBM2メモリ、{{< xe >}} Link I/O Tile が EMIB で接続される。  
そして、それらをもう 1セット Co-EMIB で互いを接続したものが、コードネーム **Ponte Vecchio** 1パッケージとなる。  

Raja氏が公開した画像から、各チップの配置を推測したものが以下。  
一見 HBM2メモリの大きさが合ってないように思えるが、これは画像が真上ではなくそこから少し右から撮影されたからで、縦幅は一致した。  

{{< figure src="/image/2021/01/27/xe-hpc-package.webp" title="Xe-HPC / Ponte Vecchio" caption="画像元: <https://twitter.com/Rajaontheedge/status/1354103878426324994>" >}}

{{< xe class="hpc">}} と HBM2メモリに間にあるチップを Rambo Cache と判断したのは、Intel は以前 XeMF (Xe Memory Fabric) と Rambo Cache がセットであるように発表しており、  
XeMF は 8x {{< xe class="hpc">}}用ともう片方のタイルとの接続用とで計 6基確認でき、Rambo Cache の数と一致するからだ。  
Rambo Cache の容量等は明かされていないが、帯域については、GPU内のキャッシュ (SRAM) と HBM2メモリの間に位置し、2つのギャップを埋める程であることが発表されている。  

[^rambo-cache]: [NVMe® Technology Powering the Connected Universe](https://nvmexpress.org/wp-content/uploads/NVMe-Technology-Powering-the-Connected-Universe.pdf)

エクサスケールスパコン Aurora は、ノードを *Sapphire Rapids* CPU 2基、*Ponte Vecchio* GPU 6基で構成し、今年 2021年に納入予定にある。  
*Sapphire Rapids* のパッケージ等はまだ公開されていないが、Linux Kernel へのパッチにログの一部があり、そちらも既に Intelラボ内部では動作していると思われる。[^idxd-spr]  

[^idxd-spr]: [dmaengine: idxd: Fix list corruption in description completion](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/commit/?h=next-20210125&id=16e19e11228ba660d9e322035635e7dcf160d5c2)


{{< ref >}}

 * <https://s21.q4cdn.com/600692695/files/doc_presentations/2019/11/DEVCON-2019_16x9_v13_FINAL.pdf>
 * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)

{{< /ref >}}
