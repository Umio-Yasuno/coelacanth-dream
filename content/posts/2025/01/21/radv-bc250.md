---
title: "RADV ドライバーで Cyan Skillfish/BC-250 のサポートが進められる"
date: 2025-01-21T11:16:49+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Cyan_Skilfish", "gfx1013" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

オープンソースで開発されている AMD GPU 向け Vulkan ドライバー **RADV** に、*Cyan Skillfish/BC-250/gfx1013* のサポートを追加するマージリクエストが Ivan Avdeev 氏によって追加されている。  
*Cyan Skillfish/gfx1013* は 2021年に AMDGPU ドライバーや LLVM にパッチが登場し始めた APU であり、*Navi10/RDNA 1* をベースとしているが、レイトレーシング関連の命令をサポートしていることが非常に特徴的な GPU アーキテクチャである。  
今になって RADV でのサポート作業が進められるのは意外だが、要望があったことと、それ程大きなコードを追加する必要がなかったことが要因かもしれない。  

BC-250 はマイニング向けの製品であり、Zen 2 6-Core を持つ APU と GDDR6 16GB を搭載し、各種 I/O や映像出力として Display Port を持った、単体で動作するカード型の PC とされている。  
GPU は 12 CU が有効化されているとするドキュメントもあるが、24 CU が認識されている BC-250 のセットアップ動画もある。  

 * [mothenjoyer69/bc250-documentation: Information on running the AMD BC-250 powered ASRock mining boards as a desktop.](https://github.com/mothenjoyer69/bc250-documentation)
 * <https://youtu.be/4dFmwjDCdLw?si=P4f0ppuF260k_EZ-&t=574>

*Cyan Skillfish/BC-250/gfx1013* は PS5 の APU を転用したものとも言われており、そのためかチップを指す名前が複数ある。Navi10_LITE、Navi12_LITE、Ariel、Cyan Skillfish、Robin…… しかしドライバーのコードとしてはそういった名前や APU の素性を列挙したところで分かりにくいだけであり、今回のマージリクエストに関しても、AMD の Marek Olšák 氏は単に製品名の BC250 を使うことを推奨している。  

 * [Draft: radv: add experimental support for CYAN_SKILLFISH (!33116) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33116)

既知の問題として、コンピュートキューを有効するとテストに失敗することが挙げられている。しかし、フリーズはしないため誤った結果を返しているのかもしれないが、原因は不明である。  
BC-250 はマイニング向け製品であり、ROCm の OpenCL ランタイムでもサポートされているため、コンピュートキューが機能しないとは考えにくい。  
処理可能なキュー数に関して、ハードウェア側での制限があるか、ファームウェアかレジスタのセットアップに問題がある可能性も考えられる。  

レイトレーシング性能に関してはそれ程速い訳ではなく、Quake2 RTX の実行速度はレイトレーシングをソフトウェアエミュレーションした場合の 3-4 倍程度とされている。  
