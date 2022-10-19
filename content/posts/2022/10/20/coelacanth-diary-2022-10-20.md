---
title: "[雑記] 2022/10/20"
date: 2022-10-20T00:29:13+09:00
draft: false
categories: [ "Diary", ]
# tags: [ "", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

個人的などうでもいいことは Mastodon か honk で出力していたため、雑記を書く機会を見失っていた。  

少し前の話だが、はてなブログで以下のような企画が行われていた。  

 * [あの頃みたいに、インターネットに書き続けよう - 週刊はてなブログ](https://blog.hatenablog.com/entry/2022/06/13/180000)

「あの頃」のサイトにあったような、独特の雰囲気や独自の時間の流れが生み出していた、アングラ感とでも言うべきものが好きで、それを感じられるようにサイトのデザインを心掛けているが、最近は少し複雑化しているようにも思えて、一部を削ったりもしている。  

このサイトを開設した時と比べると、ハードウェア情報を集めるという趣味に対する考え方が変わったように思う。  
開設してから、オープンソースコミュニティでハードウェアのサポート追加や性能を引き出すため、実際に尽力している方々や、一方で企業や開発者を揶揄することに力を入れてしまっているようなコミュニティの一部を見てきたことを思えば、それも当然なのかもしれない。  
毎世代ハードウェアを買い換えるエンスージアストでもない自分が、製品に対してあれやこれやと言うのも違うだろう。  
まだそこまではっきりとはしていないが、今はハードウェアの特徴や改良点、魅力と言えるものを伝える方向でサイトを続けたいと思っている。  

## unofficial-amdgpu-firmware-repo
`unofficial-amdgpu-firmware-repo` というレポジトリ名で、AMDGPU-PRO ドライバや ROCm に同梱されている Linux Kernel 向けの `amdgpu` ファームウェアを再公開していたが、先日削除した。  

レポジトリを作成した動機は、`linux-firmware.git` [^linux-firmware] の更新が製品のリリースに間に合わず、先行してファームウェアが同梱されている AMDGPU-PRO ドライバが必要になるケースがたまにだが発生し、それを減らすためというものだった。  
ファームウェアは `linux-firmware.git` も AMDGPU-PRO ドライバも同じライセンスとなっている。  
今はそうした問題はなくなり、Alex Deucher 氏により AMDGPU-PRO ドライバより先に `linux-firmware.git` が更新されることもある。  
本来の目的が達成されたから削除した、というのもあるが、それ以外の問題の解決策として挙げられるようなことがあったのも理由にある。  
繰り返すが、自分はファームウェアを再公開していただけで、独自の改変等は一切していない。ファームウェアの内容も `linux-firmware.git` と AMDGPU-PRO で同じであり、違うのは更新タイミングだけだと思われる。  
それなのに解決策の 1つとして異なるレポジトリのファームウェアを試すことは時間の無駄だし、危険な行為と言える。  
警告メッセージを消す目的で使われたこともあったが、`update-initramfs` の実行時に表示される警告メッセージはファームウェアファイルが無いことを示すもので、丁度新しい AMD GPU に交換するタイミングでもなければファームウェアが無いことによる問題が顕在化することはない。  

Linux Kernel における AMDGPU ドライバ、`amdgpu` ファームウェア周りで問題が発生したのであれば、同様の問題が報告されていないか調べ、それから freedesktop.org の `drm/amd` レポジトリに報告するのが良いと思う。  

 * [Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues)

[^linux-firmware]: [kernel/git/firmware/linux-firmware.git - Repository of firmware blobs for use with the Linux kernel](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/log/)

正直、今では `unofficial-amdgpu-firmware-repo` レポジトリを作ったことを後悔している。  
バイナリを配布することの危険性と、問題に対して誤った方向に導いてしまう可能性を考えるべきだった。  

## cpuid_dump_rs
Rust の勉強にと一年半近く前に作り始めたシンプルな CPUID Dump ツール `cpuid_dump_rs` だが、今も更新を続けている。  

 * <https://github.com/Umio-Yasuno/cpuid_dump_rs>

初期と比べると全体的にすっきりした構成になり、拡張もしやくすくなっている。  
`CPUID` 命令を使う関係で、途中までインラインアセンブリのサポートを有効化するため nightly リリースを使う必要があったが、Rust のバージョンが進んだことで stable でもビルドできるようになった。  

全スレッドでの実行もサポートしている。  
全スレッド分の実行結果を出力すると、どうしても長くなってしまうため、`CPU1` からは `CPU0` との差分のみを出力する機能を後から追加したが、思ってたよりあっさり実装できたため驚いた。  
具体的には、同じ構造体の値比較とその構造体の `Vec<T>` から異なる値のみを残す処理だが、比較は `#[derive(PartialEq, Eq)]` の追加だけでいいし、異なる値のみを残す処理は `retain` メソッド [^retain] を使うだけで済んだ。  
経験が浅いから、勝手に複雑な実装が必要だと思っていただけかもしれないが。  
自分なんかでも、安全でそれなりにまともなコードを書ける Rust は素晴らしい言語だと思う。  

ドキュメントや他のコードを読む動機にもなるため、これからも `cpuid_dump_rs` の更新を続けるつもりでいる。  

[^retain]: [Vec in std::vec - Rust](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.retain)
