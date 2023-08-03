---
title: "【雑記】 RX 6600 の購入とバグ報告と更新頻度と"
date: 2023-08-03T15:26:36+09:00
draft: false
categories: [ "Diary", ]
tags: [ "RDNA_2", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

今の PC を組んでから 5, 6 年とそれなりに長い間 **Radeon RX 560 4GB (Polaris11)** を使い続けた自分だが、ついに **Radeon RX 6600 8GB (Navi23)** を購入し、GPU を更新した。  
最初は **RX 560** と同様に補助電源を必要としない **RX 6400 (Navi24)** あたりを買うつもりだったが、**RX 6400** が安くなってた時期を過ぎてしまい、また悩んでいた頃に **RX 6600** が約 3万円とコスパが良くなっていたため購入を決断。  
あまり興味が無かった、というよりは性能がかなり向上していることは分かりきっているため、ベンチマークを走らせて **RX 560** と **RX 6600** のスコアを比較するようなことはしていない。  
Valve が力を入れている [Proton](https://github.com/ValveSoftware/Proton) のおかげで自分も最近はゲームを楽しむようになり、一部のゲームでグラフィクス設定を上げても 60fps の維持に余裕が出たと感じてはいる。  

それと今回 GPU を更新した動機には、自分が開発している AMD GPU 向けのモニタリングツール、[amdgpu_top](https://github.com/Umio-Yasuno/amdgpu_top) の検証もあった。  
ありがたいことに `amdgpu_top` の不具合を報告し、検証してくれている人がいるが、やはり手元の環境で検証しながら開発、修正するのが手っ取り早い。  
`amdgpu_top` には AMD GPU を搭載しているシステム側の PCIe と GPU 側の PCIe の最大リンク速度とレーン数を表示する機能があるが、Navi 系 GPU では間違っていた情報を表示していたのをまず修正した。  
Navi 系 GPU は内部に Upstream Port, Downstream Port となる複数のポート、PCIe Switch を持ち、そこに GPU の PCIe エンドポイントが繋がっているという形式らしく[^navi-port]、GPU デバイス全体で見たときの実際の PCIe リンク速度とレーン数を取得するには、GPU の PCIe エンドポイントではなく Upstream Port から取得する必要があった。  
それと `gpu_metrics` から取得できる GFX, Memory Controller, Media Engine の使用率も修正した。  
`gpu_metrics` は一部の値が SMU (System Management Unit) から得られる値をそのまま用いており、そしてその単位は `gpu_metrics` のバージョンで異なるし、ごく一部は同じバージョンであっても GPU や内部ファームウェアのバージョンによって異なる場合がある。  
どの単位が使われているかはドキュメント化されていないため、AMDGPU ドライバーのソースコードを追っていくか、実際の GPU で検証していく必要がある。  

[^navi-port]: <https://gitlab.freedesktop.org/drm/amd/-/issues/1967#note_1347631>

そして `amdgpu_top` の検証をしている内、AMDGPU ドライバー側の不具合というかサポートが抜け落ちていた部分も見つけることができた。  
`gpu_metrics` には SMU が検出した GPU のスロットリングステータスを示す値 (`throttle_status, indep_throttle_status`) があり、立っているビットの位置からスロットリングのタイプを取得できるのだが、**RX 6600** だとアイドル状態であっても消費電力と温度に関するスロットリング (`PPT0, TEMP_EDGE, TEMP_LIQUID0 ...`) が発生していることになっていた。  
少し話がずれるが、Gentoo Wiki で `gpu_metrics` をデコードする Python スクリプトが紹介されているが、そのスクリプトはフィールド部分の定義が途中から誤っており、スロットリングステータスに関しては誤った情報を表示してしまう。  
同様の情報を表示するコードを Rust で書いたが、自分で書いたコードを Wiki 系のサイトで直接紹介、宣伝するのは良くないことと認識しているため何もできずにいる。  
閑話休題。  
とりあえず [drm/amd](https://gitlab.freedesktop.org/drm/amd/-/issues) に報告したあと、AMDGPU ドライバーのソースコードを追い始めた。  
*RDNA 2* GPU の電力管理機能に関するコードは `drivers/gpu/drm/amd/pm/swsmu/smu11/sienna_cichlid_ppt.c` にまとめられている。  
その中で SMU のファームウェアバージョンによって使用する構造体を変更し、スロットリングステータスを `gpu_metrics` 内の形式に直すコードがあるのだが、それが *Navi21* のみを対象としていた。  
*Navi22, Navi23, Navi24* は対象になっていないのを少し不思議に思い、とりあえず *Navi23* も対象とするように変更、ビルドしインストールしてみた。  
するとアイドル状態では何のスロットリングも発生していないことになり、電力制限に達するまで負荷を掛けると `PPT0` のステータスが示されるようになった。  
その後、*Navi23* だけでなく *Navi22, Navi24* も対象とするよう変更したパッチを作成し、issue に投稿した。今は AMDGPU ドライバーの開発者の反応待ち。  

[^issue]: <https://gitlab.freedesktop.org/drm/amd/-/issues/2720>
[^gentoo-wiki]: [AMDGPU - Gentoo Wiki](https://wiki.gentoo.org/wiki/AMDGPU#Viewing_current_metrics)

この問題は自分が最初に見つけた訳ではなく、フレームレートや CPU, GPU の情報をオーバレイ表示する [flightlessmango/MangoHud](https://github.com/flightlessmango/MangoHud) に 1年近く前に報告されていた。[^mangohud-issue0] [^mangohud-issue1]だが、[drm/amd](https://gitlab.freedesktop.org/drm/amd/-/issues) には報告されず、そのままになっていた。  
**Ryzen 5 5600G** を購入したときも、AMDGPU ドライバーが内部 GPU の ShaderEngine, ShaderArray, CU の数を誤って検出する問題を報告したことがある。インターネット上にアップロードされているログを確認する限り、少なくとも報告した当時から半年前からその問題は発生していた。  
何が言いたいのかというと、ハードウェアを購入し、問題を報告することはオープンソースドライバーとそのコミュニティにとって重要だが、それをやれる人、やろうとする人は少ないということだ。  

[^mangohud-issue0]: [is throttling option not working correctly · Issue #790 · flightlessmango/MangoHud](https://github.com/flightlessmango/MangoHud/issues/790)
[^mangohud-issue1]: [MangoHud always displays Throttling Power · Issue #748 · flightlessmango/MangoHud](https://github.com/flightlessmango/MangoHud/issues/748)

ついでにこのブログの更新頻度がひどく鈍くなった理由について書くと、ハードウェアやオープンソースドライバーについて、ただあれこれ語るより `amdgpu_top` のようなツールを開発した方が役立つと考えるようになったからだ。Linus Torvalds 氏も *Talk of tech innovation is bullsh\*t. Shut up and get the work done* と言っている (らしい)。[^linus-torvalds-says]  

これは以前から認識していたことではあるが、オープンソースドライバーへのパッチから将来のハードウェアに関する情報を探り、語ることは、時に開発者に迷惑を掛ける。  
比較的最近の事例だと *RDNA 3* のシェーダープリフェッチ機能に関する話題がそうだった。  

それに、ずっと前のことだが多くの人からリーカーとして認識されている人が、正式発表のイベントに対して、「リーク通りの情報しかなくてつまらなかった」と言っていた。それでもその人はリーカーとして振る舞うことをやめられなかった。  
今ではその人は観測できない場所で活動するようになったため、どうしているか自分は知ることができない。  
ただ自分もいつか同じようになってしまう気がした。  

そして、先のことについて知ろうとして楽しみを減らし、さらには開発者に迷惑を掛ける可能性がある、これまでの自分の活動はすべて間違っていたかもしれないと考えるようになった。  
以上が更新頻度がひどく鈍くなった理由となる。ブログを書く時間をゲームに使うようになったのもあるが。  
ブログの更新を完全にやめるつもりは現状無いが、1ヶ月に 1, 2回更新があれば良い方になるとは思う。  

[^linus-torvalds-says]: [Talk of tech innovation is bullsh*t. Shut up and get the work done – says Linus Torvalds • The Register](https://www.theregister.com/2017/02/15/think_different_shut_up_and_work_harder_says_linus_torvalds/)
