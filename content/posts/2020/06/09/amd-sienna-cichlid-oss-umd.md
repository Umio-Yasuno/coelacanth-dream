---
title: "AMD Sienna Cichlid をサポートするパッチが OpenGL、Vulkanドライバーに投稿される"
date: 2020-06-09T04:29:37+09:00
draft: false
tags: [ "Sienna_Cichlid", "Navi21", "RDNA_2", "GFX10", "gfx1030", "RadeonSI", "RADV" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMDGPU向けのオープンソースなドライバー、RadeonSI(OpenGL)、RADV(Vulkan) が *Sienna Cichlid* サポートに向けて動き始めた。  
{{< link >}}[radv: add Sienna Cichild (!5389) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/commits){{< /link >}}

今回読み取れるのは、*Sienna Cichlid* が

 * [SIMDユニットあたりで保持する実行中の Wave数が Navi1x の 20エントリ から 16エントリ に減らされている](#wavefront-16)[^2]
 * [GPU ID は gfx1030](#gfx1030)[^1]
 * [DFSM(Deterministic Finite State Machine) のサポートが外された](#dfsm)[^3]
 * [`pc_lines` は 1024](#pc-lines)[^5]
 * [フレームバッファの `big_page` に対応](#big-page)[^4]
 * [SDMAが巨大なサイズ(約1GB)のパケットに対応](#sdma-big-packet)[^6]

ということだ。  

[^1]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5383/diffs?commit_id=c09cac343eb8dbca0b8dda24941540b20768702b#diff-content-fbbe09bc487101e3f2f96c0b8fe543c915fa99bf>
[^2]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=3239184db55f21c6708c87805284e257b780b5cb#diff-content-c3cf206d71203e77a4252c3915daf913c9251dc3>
[^3]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=3239184db55f21c6708c87805284e257b780b5cb#diff-content-c5f2982df5fe472882e2234484ac9885cc80b82f>
[^4]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=8c855740cc71cf56c99e7a37ce9b782b2b32a966>
[^5]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5383/diffs?commit_id=c09cac343eb8dbca0b8dda24941540b20768702b#diff-content-c3cf206d71203e77a4252c3915daf913c9251dc3>
[^6]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=859d7a5fe951ce45b4693f0a3bb55fbbe0c5b2d5>

RADV のバックエンドに使われる LLVM は ver11 で *Sienna Cichlid* をサポートする予定でいるらしく[^8]、  
LLVM のリリース間隔を見ると、ver11 は 2020/09 に来る可能性が高い。  
{{< link >}}[Download LLVM releases](https://releases.llvm.org/){{< /link >}}
インターネット上の噂でもそのくらいに新GPUが出るのではと囁かれている。  

[^8]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=7366021e3854532837dc5c569bc8a24bb023c11b>

## SIMDユニットあたりで保持する Wave数が減る、スレッド性能の向上が目的か {#wavefront-16}
*AMD GCNアーキテクチャ* では SIMD16ユニットあたりに保持する実行中の Wave数(Wavefront)が 10エントリであり、CU全体では 40エントリとなっていた。  
*AMD RDNAアーキテクチャ* でもそのバランスを保ち、SIMD32ユニットあたり 20エントリ、WGP(2CU)全体では 80エントリとなっていた。  
それが *Sienna Cichlid* では SIMD32ユニットあたり 16エントリ、WGP(2CU)全体で 64エントリになることが今回のマージリクエストからわかる。  
同時に実行可能な Wave(スレッド)数は減るが、保持した Wavefrontを順番に実行する方式であるため、同じ Wavefrontを実行するまでの間隔が小さくなり、スレッド性能は高まる。  
スレッドに割り当てられるレジスタ数も増えるため、その点でもスレッド性能の向上が期待できる。  
{{< link >}}参考: <cite>[コンピュータアーキテクチャの話(365) GCNのブロックダイヤグラムを読む | マイナビニュース](https://news.mynavi.jp/article/architecture-365/)</cite>{{< /link >}}

ちなみに、コードを見る限り *AMD Polaris系 (Polaris10/11/12, VegaM)* も SIMD16ユニットあたり 8エントリ、CU全体で 32エントリとなっているらしく、過去にゲーミングGPUのため取り入れた改良点を再度取り入れたとも見られる。  

## GPU ID: gfx1030 {#gfx1030}
ディスプレイコントローラの eChipRev が *Navi21* と一致していたことから、`Sienna Cichlid == Navi21` と考えたが、*gfx1030* という GPU ID もやはり *Navi21* と一致する。  

## DFSM のサポートが外される {#dfsm}
DFSM(Deterministic Finite State Machine) は、  
GFX9 で追加された、レンダリング領域を細かいタイルに分割、頻繁に使われる部分を L2cache に置くことで効率化する DSBR(Draw Stream Binning Rasterizer) 機能のソフトウェア側実装だが、  
まあ色々あったり効果が無かったりで微妙な扱いを受けていた。  
{{< link >}}[Vegaの新機能今いずこ | Coelacanth's Dream](/posts/2019/12/07/where-vega-new-feature/#dsbr-draw-stream-binning-rasterizer){{< /link >}}

今回追加されたコメントにも、Navi1x では効果が無かったとあり、  

```
/* DFSM is not supported on GFX 10.3 and not beneficial on Navi1x. */
```

引用元: <cite><https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=3239184db55f21c6708c87805284e257b780b5cb#diff-content-c5f2982df5fe472882e2234484ac9885cc80b82f></cite>

そこに元あったコードでは、専用VRAMが無い場合、つまり APU ならば有効可能にする処理を取っていたが、Navi1x GPUを統合した APU は現在無い。  
*Raven* でも DFSM有効化により、かえって性能が下がるケースがあったため、今後 GFX 10.3 またはそれ以降の GPU を統合した APU が出てきても DFSM が有効にされることはないだろう。  

## pc_lines は 1024 {#pc-lines}
`pc_lines` は Parameter Cache Lines の略とされている[^7]、のだが正直、自分自身詳しいことはわかっていない。  
上記 DFSM機能にその一部が使われるというくらいで。  

*Sienna Cichlid* の `pc_lines` の値は 1024 で、*Navi10 /Navi12* と同一である、ということを言及するに留めておく。  

[^7]: <https://github.com/GPUOpen-Drivers/pal/blob/82390674c40402c81dbf311a5ee488fb972894f5/src/core/hw/gfxip/gfx9/gfx9Device.cpp#L4198>

## フレームバッファの big_page に対応 {#big-page}
使用するメモリを表すバッファオブジェクトをより大きな単位で取り扱う機能と推察される。  
メモリの効率化が目的だろうか。  

## SDMAが巨大なサイズ(約1GB)のパケットに対応 {#sdma-big-packet}
これまでは 4MBまでだったパケットサイズが、1GBまでに拡張された。  
これもまた上記 `big_page` のように効率化が目的と思われる。  

加えて *Sienna Cichlid* はSDMAコントローラを4基持つことからも、SDMA周りの機能をかなり強化している印象を受けるが、それが向かう先はまだベールに包まれている。  

{{< ref >}}

 * <https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf>
 * <https://www.olcf.ornl.gov/wp-content/uploads/2019/10/ORNL_Application_Readiness_Workshop-AMD_GPU_Basics.pdf>
 * [AMDのRDNAアーキテクチャの「Navi GPU」を読み解く - Hot Chips 31 | マイナビニュース](https://news.mynavi.jp/article/20191023-912850/)
 * [コンピュータアーキテクチャの話(365) GCNのブロックダイヤグラムを読む | マイナビニュース](https://news.mynavi.jp/article/architecture-365/)

{{< /ref >}}
