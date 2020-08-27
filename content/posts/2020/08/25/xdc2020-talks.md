---
title: "XDC2020 注目の講演　―― ACO, P2P DMA, レイトレーシング……"
date: 2020-08-25T12:54:19+09:00
draft: false
tags: [ "ACO", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

2020/09/16 - 2020/09/18 にかけて、Linux Kernel や Mesa3D、DRM、Wayland、X11 等、オープンソース・ソフトウェアにおけるグラフィクス部の開発者達が各種講演を行なう *X.Org Developers Conference 2020 (XDC2020)* が開催される。  
{{< link >}} [X.Org Developers Conference 2020 (16-18 September 2020): Overview · Indico](https://xdc2020.x.org/event/9/) {{< /link >}}
今回は COVID-19 の影響を受け、バーチャルでの開催となる。  

[Hot Chips](https://hotchips.org)のような最新ハードウェアの詳細が明かされるイベントに比べれば注目度は低いかもしれないが、  
{{< link >}} [Hot Chips 32 個人的まとめ | Coelacanth's Dream](/posts/2020/08/18/hc32-matome_1/) {{< /link >}}
ソフトウェア開発者が直接技術の詳細を解説してくれる貴重な機会であり、ハードウェアに関する知識を別の側面から深められるし、今後投稿されるパッチを読み解く手助けにもなる。  

カンファレンスの中で特に自分が気になる講演をここで紹介したい。  
講演の様子は、昨年と同様であれば [X.Org Foundation の Youtubeチャンネル](https://www.youtube.com/c/XOrgFoundation/videos)にてリアルタイムで配信が行なわれ、また個別の講演も投稿されるはずだ。  
スライドもまた [XDC2020](https://xdc2020.x.org/) のサイトページにて PDFファイル形式で公開される。  

## 個人的に注目の講演

題目一覧と概要は以下のページに掲載されている。  
{{< link >}} [X.Org Developers Conference 2020 (16-18 September 2020): List of all talks/topics · Indico](https://xdc2020.x.org/event/9/contributions/) {{< /link >}}

### ACO

 * [A year of ACO: from prototype to default](https://xdc2020.x.org/event/9/contributions/612/)

 [ACO バックエンド](/tags/aco) は AMD GPU (GCN/RDNA) の新たなシェーダーコンパイラバックエンドとして、Valve の支援を受けて開発が行なわれ、約1年前に実験的な機能として [RADV](/tags/radv) (オープンソース Vulkan ドライバー) に導入された。  

コンパイラバックエンドとしての性能は確かなものであり、それまでの *LLVM バックエンド* と比較して、最大でグラフィクス処理では約1.2倍近くの性能向上、個人的に行なった検証ではコンピュート処理で約3倍の性能向上を実現している。[^cd-aco-ncnn]  

[^cd-aco-ncnn]: [RADV/ACO 検証: waifu2x-ncnn-vulakan の実行が最大3倍近く高速に | Coelacanth's Dream](/posts/2020/04/26/waifu2x-ncnn-vulkan-speedup-aco/)

そして、2020/06 には *ACO バックエンド* がデフォルトで有効にされるパッチがメインラインに組み込まれている。[^radv-aco-default]  
これは、それまで実験的な機能としてオプションから有効にする必要があった *ACO バックエンド* が、*LLVM バックエンド* 同等の安定性と機能を得たことを意味する。  

[^radv-aco-default]: [radv: enable ACO by default (63e1e720) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/63e1e7209c2bf7d8bbce18380d4f1b2cff4c0bbb?merge_request_iid=5445)

また、*ACO* を [RadeonSI](/tags/radeonsi) (オープンソース OpenGL ドライバー) のバックエンドにも採用するプランがあることが **RADV** 開発者の Samuel Pitoiset 氏から明かされている。[^radeonsi-aco]  

[^radeonsi-aco]: [RADV/ACO をデフォルトで有効にする動き、RadeonSIに ACO を バックエンドに用いる計画も | Coelacanth's Dream](/posts/2020/06/14/radv-aco-default-and-radeonsi-aco-plan/)

そんなオープンソースドライバーに飛躍をもたらした *ACO* の沿革とデザインについて、Valve の Timur Kristóf 氏が講演を行なう。  

### Peer to Peer DMA

 * [Why is Peer to Peer DMA so hard on Linux?](https://xdc2020.x.org/event/9/contributions/617/)

CPU を介さずに他デバイス間で直接メモリアクセス、データ転送を行なう DMA は HPC でもゲーミング等でも I/O のスループットと、デバイスの性能を向上させる重要な役割を持つ。  
だが、Linux においては最近になってようやく P2P DMA が機能するようになったと言う。  
Linux における P2P DMA 機能の歴史と現状、実装を困難とする部分、そして将来的な改善について、Linux Kernel の AMD GPU ドライバー開発を担当している、AMD の Alex Deucher 氏が講演を行なう。  

AMD は 2021年納入予定の *Frontier* に向けてか、スパコンに向けた GPU 管理機能等を Linux Kernel に実装、強化している。[^amd-gfx-for-hpc]  
その中で、今回の講演は興味深い内容になるのではないかと思う。  

[^amd-gfx-for-hpc]: [スパコンに向けて強化される Linux Kernel の AMDGPU 関連機能 | Coelacanth's Dream](/posts/2020/04/29/linux-kernel-advanced-amdgpu-monitor/)

### レイトレーシング {#ray-tracing}

 * [Ray-tracing in Vulkan: A brief overview of the provisional VK_KHR_ray_tracing API](https://xdc2020.x.org/event/9/contributions/613/)

レイトレーシングのための Vulkan 拡張機能 `VK_KHR_ray_tracing` を使うにあたって理解が求められる新しいシェーダーステージやオブジェクト等について、Intel の Kason Ekstrand 氏が講演を行なう。  
この講演に実装の詳細等は含まれておらず、X/Mesa の開発者コミュニティに向けた勉強会のような位置付けにある。  

GPU内の固定機能ユニットを用いた高速なレイトレーシングは、NVIDIA RTX、DXR、新世代ゲーム機といった波に乗って、身近なものになり *かけて* きている。  
しかし、オープンソースに関連させて言えば、レイトレーシングをサポートする GPU は現状 NVIDIA RTX シリーズのみであり、そして NVIDIA はドライバーをクローズドソースで提供している。  
GPU ドライバーがオープンソースで提供されている GPU ベンダーには Intel と AMD がいるが、両者はまだコンシューマ向けにハードウェアレイトレーシングをサポートした GPU をリリースしていない。  
つまり、オープンソース界隈はまだハードウェアレイトレーシングの波に乗れていない。ハードウェア、ソフトウェアの両方で。  
Intel は {{< xe class="hpg" >}}、AMD は [RDNA 2 アーキテクチャ](/tags/rdna_2)でハードウェアレイトレーシングをサポートするとしており、狭い範囲で言えば、それらに向けたパッチを読み解くのに役立つかもしれない。  

個人的な考えを言えば、最も疑問に思うのは、ハードウェアレイトレーシングを用いたソフトウェア/ゲームが Linux に向けてどれだけ開発されるか、レイトレーシングがどれだけメジャーなものになり得るか、だが。  

## その他 GPU 関連
### AMD GPU のプロファイリング機能

 * [Profiling on AMD GPUs using tracing](https://xdc2020.x.org/event/9/contributions/623/)

**RADV** には [Radeon GPU Profiler](https://github.com/GPUOpen-Tools/radeon_gpu_profiler) で確認可能なプロファイリングファイルを出力する機能がある。  
そこから分析し、実行性能を向上させる方法についての講演を **RADV** 開発者である Bas Nieuwenhuizen 氏が行なう。  

### AMD Zero Core power technology

 * [Don't bake your Graphics cards!](https://xdc2020.x.org/event/9/contributions/633/)

*GPU を焼くな!* というタイトルだが、講演の中身は AMD GPU に導入されている、アイドル時の消費電力を削減する省電力技術、*AMD Zero Core power technology* についてである。  
*AMD Zero Core power technology* はマーケティング的な名前であり、Linux Kernel 等では *BACO (Bus Alive/Active Chip Off)* の名で実装されている。  

講演は、AMD の Alex Deucher 氏と、同じく AMD の Rajneesh Bhardwaj 氏によって行なわれる。  

### Trust Memory Zone

 * [Secure Buffer Object Support with Trusted Memory Zone](https://xdc2020.x.org/event/9/contributions/630/)

コンテンツ保護においてメモリの暗号化は重要となる。AMD GPU に実装されているメモリ暗号化機能 *TMZ (Trust Memory Zone)* の詳細、実装に必要とされるハードウェア側の機能、制限について、AMD の  Ray Huang 氏が講演を行なう。  

*TMZ* は KMD(Kernel Mode Driver) 、UMD(User Mode Driver) 両方の対応が進められているが、まだオプションで有効可能という段階にあり、*TMZ* が必須となる状況にはまだぶつかっていない。  
将来的なDRM技術への対応に必要とされたりするのだろうか。  
または、正式に実装されれば DRM技術で保護されたゲームに実行が Linux 環境において可能になるかもしれない。  
