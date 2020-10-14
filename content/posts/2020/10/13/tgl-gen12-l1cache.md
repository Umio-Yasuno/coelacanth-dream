---
title: "Intel Gen12 GPU で増設された L1データキャッシュの効果"
date: 2020-10-13T20:28:04+09:00
draft: false
tags: [ "Gen12", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Intel GPU のオープンソースドライバーに、*Tiger Lake* GPU における L1キャッシュに対応するマージリクエストが投稿されている。  
その中で、L1キャッシュ対応による性能への効果について触れられていた。  

 * [WIP: intel: Enable L1/HDC caches on Tigerlake (!7104) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7104)
   * [isl: Enable Tigerlake HDC:L1 caches via MOCS in various cases.](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7104/diffs?commit_id=758bc2daeb21e2972f985d199d2b162b6cacb8e0)

{{< pindex >}}

 * [L1データキャッシュ](#l1-data-cache)
 * [L1データキャッシュの効果](#l1-effect)

{{< /pindex >}}

## L1データキャッシュ {#l1-data-cache}
ここでの L1キャッシュは *{{< xe class="lp" >}}/Gen12.1 アーキテクチャ* より Sub-Slice部に増設された L1データキャッシュのことを指す。  
また、HDC は Data Cluster または Data Port の意とされ、キャッシュやローカルメモリへのアクセス帯域に関わる部分である。  
MOCS は Memory Object Control State の略。GPU部の L3キャッシュや LLC(Last Level Cache)、eDRAM をキャッシュとして定義するために用いられる。  

{{< figure src="/image/2020/08/14/xe-lp-cache.webp" caption="画像出典: [VimeoArchitecture Day 2020 (Event Replay)](https://vimeo.com/intelpr/review/447304765/179933d14f)" >}}

ただ、Intel は *{{< xe class="lp" >}}/Gen12.1 アーキテクチャ* に関するドキュメントをまだ公開しておらず、L1データキャッシュの容量や帯域等は不明。  
上記画像では L1/TEX$ という風に書かれており、元々テクスチャサンプラー部に含まれていたキャッシュを汎用的に扱えるよう移したとも考えられるが。  

## L1データキャッシュの効果 {#l1-effect}
今回のマージリクエストでは、テクスチャやイメージの格納、データの移動に用いることを想定した L1 + L3 と、  
フレームバッファや定数を想定した L1 + L3 + LLC の MOCS が追加されている。  
ここに L2 が無いのは、Intel GPU では L3キャッシュバンクそれぞれの内部を用途別に割り当てる形式を採っており、その中でテクスチャサンプラー等がアクセスするリードオンリーキャッシュ部を L2キャッシュとしているためである。  
L3キャッシュの一部が実質 L2キャッシュと扱われ、メモリ階層としてそれ以上は CPU と共有する LLC、次に eDRAM や DRAM となる。  

コメントによると、Windows の Vulkan/OpenGL ドライバーでも近い対応 (全く同じではない) を行なっているようだ。  

L1データキャッシュの対応により、いくつかのゲームタイトルやグラフィクスベンチマークで性能向上が確認され、Unreal Engine 4 Shooter Demo では 11.84%、Doom (2016) では 11.40%、Shadow of the Tomb Raider では 9.40% 性能が向上している。  
これらは Vulkan API に対応したゲーム、ベンチマークであり、OpenGL API に対応したゲームでは性能向上幅が Vulkan よりも小さくなり、Xonotic では 2.78%、Counter Strike: Global Offensive では 1.43% となっている。  
GFXBench5 では Vulkan、OpenGL 関係無く小さい性能向上に留まり、Vulkan で 1.52%、OpenGL で 1.01％。  
また、中には性能が下がったタイトルもあり、Dota 2 では Vulkan で 2.2%、OpenGL で 1.6% 性能が低下している。  

<br>
L1データキャッシュへの対応で最大で 12% 近く性能が向上するというのは、アーキテクチャの改良においてかなり効果的であるように思える。  
ますますキャッシュサイズ、帯域が気になる所だ。  
中には逆に性能低下しているゲームタイトルもあるが、L1データキャッシュは *{{< xe class="lp" >}}/Gen12.1 アーキテクチャ* で増設されたメモリ階層であり、今後の最適化での解消が期待される。  

また、L1データキャッシュはコンピュート処理にも効果的と思われるが、[compute-runtime](https://github.com/intel/compute-runtime) では現在無効化されているようだ。[^disable-l1]  

[^disable-l1]: [Disable L1 for Gen12lp · intel/compute-runtime@38ca6e9](https://github.com/intel/compute-runtime/commit/38ca6e986244fcf783dd711950b78376a85731ff)

{{< ref >}}

 * [2019 Intel(r) processors based on Ice Lake platform | 01.org](https://01.org/linuxgraphics/hardware-specification-prms/2019-intelr-processors-based-ice-lake-platform)
    * [Volume 4: Configurations - intel-gfx-prm-osrc-icllp-vol04-configurations_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol04-configurations_0.pdf)
    * [Volume 7: Memory Cache - intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf)


{{< /ref >}}
