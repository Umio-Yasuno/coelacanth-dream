---
title: "【雑記】 AMD GPU のモニタリングツールを作ってみた"
date: 2023-03-08T04:30:01+09:00
draft: false
categories: [ "Diary", "Note" ]
tags: [ "Rust", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

TUI (Text User Interface) ライブラリ `Cursive` を使って Rust 言語で AMD GPU のモニタリングツール `amdgpu_top` を作った記録。  

 * [Umio-Yasuno/amdgpu_top](https://github.com/Umio-Yasuno/amdgpu_top)

以前から練習も兼ねて `libdrm_amdgpu` の Rust バインディングを作っていたのだが、ふとそれで [clbr/radeontop](https://github.com/clbr/radeontop) や AMD の Tom St Denis 氏が開発している [umr (User Mode Register Debugger for AMDGPU Hardware)](https://gitlab.freedesktop.org/tomstdenis/umr/) のような AMD GPU のモニタリングツールを作れることに気付いた。  

まず、AMD GPU のメモリ使用量や一部センサーの値は `libdrm_amdgpu` 経由で読み取ればいいため簡単だが、GPU 内部の各ユニットの使用率については特殊なレジスタを読み取ってサンプリングする必要がある。  
`libdrm_amdgpu` を経由したレジスタの読み取りには `amdgpu_read_mm_registers` 関数が用意されている。引数には、デバイスハンドル、レジスタオフセット、GPUインスタンスを指定するビットマスク、値を書き込む先のポインタを渡す必要がある。レジスタオフセットから連続で読み出す数やフラグも引数にはあるが、自分の用途では特に気にする必要はなかった。  
レジスタオフセットはどんな値でも良いという訳ではなく、AMD GPU ドライバー側で許可されていないレジスタオフセットの場合はエラーが返されるようになっている、  
のだが *Vega/GFX9* とそれ以降の世代向けだとチェック処理に間違いがあり、どんなレジスタオフセットでも成功するようになっていた。このバグを報告したところ、すぐに AMD の Alex Deucher 氏により修正パッチが作成された。[^patch]  
また、自分が読んだ限りだが `umr` では `libdrm_amdgpu` を使わずに MMIO レジスタから読み取っているため AMD GPU ドライバーによる制限はない。(ただし root 権限が必要となっている)  

[^patch]: [[PATCH 1/2] drm/amdgpu: fix error checking in amdgpu_read_mm_registers for soc15](https://lists.freedesktop.org/archives/amd-gfx/2023-March/090189.html)

内部ユニットの使用状況を示す特殊なレジスタには主に `GRBM (Graphics Register Block), SRBM (System Register Block), SRBM2` がある。  
`GRBM` は GFX部の各ユニット、`SRBM` は `UVD (Unified Vide Decoder/Encoder)`、`SRBM2` は `VCE (Video Compression Engine)` と `SDMA (System DMA)` の使用状況を示すようになっている。  
他にも `CP_STAT (Command Processor Status?)` などがあるが、ドキュメントが公開されていない。AMD GPU ドライバーや `umr` にある定数をまとめた部分から各ビットの対応は分かるものの、定数では内部ユニットを略称で表記しているため、何の略称かまで分からないとほとんど意味が無い。  
`SRBM` と `SRBM2` については、*Vega/GFX9* とそれ以降の世代では AMD GPU ドライバー側で許可されておらず、`amdgpu_read_mm_registers` からは読み取ることができない。  
バグを報告した際についでで Alex Deucher 氏に質問したところ、*Vega/GFX9* とそれ以降ではそもそも `SRBM` と `SRBM2` を持たないとの回答だった。  

ユーザーが内部ユニットの使用状況を知って嬉しい場面は少ないかもしれないが、ともかくとして特殊なレジスタから知ることができる。  

以下は動作中の `amdgpu_top` のスクリーンショット。  
色とテーマは `Cursive` のデフォルトそのまま。最初は変えるつもりだったが、変にいじるよりこのままの方が良いんじゃないかと考えている。  

{{< figure src="../amdgpu_top.png" >}}

TUI ライブラリも `Cursive` も自分は初めてだったが、少し慣れてくると `Cursive` が非常に使いやすいことが分かってきた。  
`amdgpu_top` ではキー入力によって表示設定を動的に変更可能にしているが、`set_user_data/with_user_data` で `Arc<Mutex<T>>` を変更可能にし、`add_global_callback` で各フラグを反転させるキーを設定、そして更新用スレッドが参照する形にするだけで実装できた。  
内容の更新やレイアウトの管理もやりやすいと感じた。  

`/sys/kernel/debug/dri/{instance}/amdgpu_gem_info` をパースして VRAM を使用しているプロセスとその使用量も表示可能にしている。  
他の情報はユーザースペースから取得可能だが、`amdgpu_gem_info` は DEBUGFS にあるため root 権限が必要となる。  

正直に言えば `amdgpu_top` は機能的には `umr` の下位互換だが、Rust 言語でシンプルに書くことができたことに満足している。  
それに `amdgpu_top` の作成の過程で、微力ではあるが AMD GPU ドライバーに貢献することができた。  

{{< ref >}}
 * [gyscos/cursive: A Text User Interface library for the Rust programming language](https://github.com/gyscos/cursive)
 * [Umio-Yasuno/libdrm-amdgpu-sys-rs: libdrm_amdgpu bindings for Rust, and some methods ported from Mesa3D](https://github.com/Umio-Yasuno/libdrm-amdgpu-sys-rs)
{{< /ref >}}
