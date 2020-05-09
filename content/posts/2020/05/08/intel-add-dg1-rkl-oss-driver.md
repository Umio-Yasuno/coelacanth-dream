---
title: "Intel、オープンソースドライバーに DG1 と Rocket Lake の関するコードを追加"
date: 2020-05-08T17:07:08+09:00
draft: false
tags: [ "DG1", "Rocket_Lake", "Gen12" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "GPU" ]
noindex: false
---

まだマージリクエストの段階ではあるが、オープンソースなGPUドライバーに *DG1* と *Rocket Lake* に関するコードが追加された。  
{{< link >}}[intel/dev: Add DG1 platform (!4956) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/4956/diffs){{< /link >}}
{{< link >}}[intel/dev: Add RKL platform (!4955) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/4955/diffs){{< /link >}}

*DG1* 、 *Rocket Lake* は現段階でGPU部の製品名は **Intel(R) Graphics** となっている。  
*Tiger Lake* はそれ以外に **Intel(R) Xe Graphics** 、**Intel(R) UHD Graphics** という名前が用意されており、GPU性能の違いでも製品展開するつもりがあることが窺える。[^1]  

[^1]: [intel: Update TGL PCI strings (1c6ef016) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/1c6ef0165f03a8e8c20a2c33a78584166a73487c)

## GPU規模
### DG1
コードを読むに、*DG1* の規模は *Tiger Lake GT2* とほぼ変わらない、Dual-SubSlice数 6基、L3キャッシュバンク数 8基というもの。  
しかし、URB(Unified Return Buffer)のサイズは *Tiger Lake GT2* の 1024KB よりも小さい 768KB となっている。URBはL3キャッシュに統合されており、各シェーダーの入出力やローカルスレッドの発行のためのバッファとして機能する。[^2]  
Intel が公開している *Ice Lake* GPUのドキュメントを読むと、L3キャッシュバンクあたりの URB を 96KBとする設定は1つだけであり、L3キャッシュのタグとして使われる Rest が省かれている。そしてその分をグローバルメモリアクセス等に使用される Data Cluster と、 Read-Only である命令キャッシュや状態管理、テクスチャーを収める分に割り振っている。  
このことから、{{< comple >}}L3キャッシュ構成が *Ice Lake /Gen11* と同様であるという前提付きではあるが{{< /comple >}}URBのサイズが *DG1* で減らされているというのは、性能をGPGPUにも向ける意味があるのではないかと思う。  

[^2]: <https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf>
[^3]: <https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf#page=9>

### Rocket Lake GT0.5/GT1
*Rocket Lake* には GT0.5、GT1 の2種類が用意されており、  
GT0.5 は Dual-SubSlice数 1基、L3キャッシュバンク数 4基。  
GT1 は Dual-SubSlice数 2基、L3キャッシュバンク数は GT0.5 と同数の 4基。  

GTの後の数字が示すように、GT0.5 のGPUコア部はGen12LPアーキテクチャでは最小の規模だ。  
また *Rocket Lake* では GT2 が現時点で存在しないが、GPU性能は *Tiger Lake* で、ということと思われる。  

## Gen12LP
Gen12LPアーキテクチャではGen11アーキテクチャから踏襲する、Slice内の SubSlicesを増やして性能を調整する方法を取っているため、*DG1* でも *Tiger Lake* でも *Rocket Lake* でも総Slice数は1基で変わらない。  
L3キャッシュバンクあたりの容量はまだ確定していないが、Gen11アーキテクチャから変わらないとすれば 384KBとなる。  
以下はGen12アーキテクチャ採用GPUを比較した表。  

| Gen12LP | TGL GT2 | DG1 | RKL GT0.5 | RKL GT1 |
| :--- | :---: | :---: | :---: | :---: |
| Dual-Sub Slices | 6 | 6 | 1 | 2 |
| &emsp;EUs | 96 | 96 | 16 | 32 |
| &emsp;Shading Units | 768 | 768 | 128 | 256 |
| GPU L3$ | 3072KB? | 3072KB? | 1536KB? | 1536KB? |
| &emsp;URB Size | 1024KB | 768KB | 512KB | 512KB |
