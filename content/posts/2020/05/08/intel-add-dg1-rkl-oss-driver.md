---
title: "Intel、オープンソースドライバーに DG1 と Rocket Lake の関するコードを追加"
date: 2020-05-08T17:07:08+09:00
draft: true
tags: [ "DG1", "Rocket_Lake", "Gen12" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "GPU" ]
noindex: false
---

まだマージリクエストの段階ではあるが、オープンソースなGPUドライバーに *DG1* と *Rocket Lake* に関するコードが追加された。  
{{< link >}}[intel/dev: Add DG1 platform (!4956) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/4956/diffs){{< /link >}}
{{< link >}}[intel/dev: Add RKL platform (!4955) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/4955/diffs){{< /link >}}

*DG1* 、 *Rocket Lake* は現段階でGPU部の製品名は **Intel(R) Graphics** となっている。  
*Tiger Lake* はそれ以外に **Intel(R) Xe Graphics** 、**Intel(R) UHD Graphics** という名前が用意されており、GPU性能の違いでも製品展開するつもりがあることが窺える。  

## GPU規模

コードを読むに、*DG1* の規模は *Tiger Lake GT2* とほぼ変わらない、Dual-SubSlice数 6基、L3キャッシュバンク数 8基というもの。  
しかし、URB(Unified Return Buffer)のサイズは *Tiger Lake GT2* の 1024KB よりも小さい 768KB となっている。また、統合GPUではないため、CPUと共用するLLC(Last Level Cache)はないとされている。  

*Rocket Lake* には GT0.5、GT1 の2種類が用意されており、  
GT0.5 は Dual-SubSlice数は 1基、L3キャッシュバンク数は 4基。  
GT1 は Dual-SubSlice数は 2基、L3キャッシュバンク数は GT0.5 と同数の 4基。  

GPUコア部の規模は、プロプライエタリなドライバー等から出てきたものであろう前情報と一致している。  
{{< link >}}[Intel DG1は96EU構成? & 推測 | Coelacanth's Dream](http://localhost:1313/posts/2019/12/28/intel-dg1-96eu-guess/){{< /link >}}
{{< link >}}[Rocket Lakeの噂、そこからの推測 | Coelacanth's Dream](/posts/2020/05/04/rocketlake-rumor-guess/){{< /link >}}

Gen12LPアーキテクチャではGen11アーキテクチャから踏襲する、Slice内の SubSlicesを増やして性能を調整する方法を取っているため、*DG1* でも *Tiger Lake* でも *Rocket Lake* でも Slice数は1基で変わらない。  
L3キャッシュバンクあたりの容量はまだ確定していないが、Gen11アーキテクチャから変わらないとすれば 384KBとなる。  

| Gen12LP | TGL GT2 | DG1 | RKL GT0.5 | RKL GT1 |
| :--- | :---: | :---: | :---: | :---: |
| Dual-Sub Slices | 6 | 6 | 1 | 2 |
| &emsp;EUs | 96 | 96 | 16 | 32 |
| &emsp;Shading Units | 768 | 768 | 128 | 256 |
| GPU L3$ | 3072KB? | 3072KB? | 1536KB? | 1536KB? |
