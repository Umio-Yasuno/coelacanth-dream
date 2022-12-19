---
title: "ActivityPub サーバー、honk をカスタマイズしてみる"
date: 2022-12-18T16:47:07+09:00
draft: false
categories: [ "Diary", "Note" ]
tags: [ "Honk", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

最近の Twitter の情勢を受けて、他の SNS、特に Mastodon サーバーに移行または独自のサーバーを構築する人が増えている。  
加えて自分の観測範囲では、以前に紹介した Ted Unangst 氏による Go言語製の ActivityPub サーバーの最小実装、`honk` を構築する人が増えている。[^honk-intro]  
`honk` は省メモリであるため、メモリ 512MB の VPS でも快適に動作する。  
メインでも予備でも、個人で運用可能な ActivityPub サーバーが欲しい人にとって `honk` は有力な選択肢になると思う。  
また、最小実装ということが手伝い、カスタマイズ、それによる再ビルドと再起動に必要とするコストは小さい。  
そこで、個人的に `honk` を運用しながらカスタマイズしていって得た知見を共有することで、誰かの `honk` を構築し、カスタマイズする切っ掛けになればと思い、この記事を書いている。  

 * [humungus - honk](https://humungus.tedunangst.com/r/honk)

[^honk-intro]: [Go言語製 ActivityPub サーバーの最小実装 honk を立ててみた | Coelacanth's Dream](/posts/2022/06/19/honk/)

カスタマイズにあたって先にオープンソースライセンス周りの話に触れると、有名な ActivityPub サーバー、Mastodon, Pleroma, Misskey, Pixelfed, PeerTube では AGPLv3 が採用されているが、`honk` では ISCライセンスが採用されている。  
ライセンスについて詳しい訳ではないため雑な解説となるが、AGPLv3 ではソフトウェアの利用にあたってコピーレフトが適用されるが、ISCライセンスにコピーレフト条項はない。  
こうした点でも `honk` はカスタマイズが容易だと言えるかもしれない。  

## Customization {#custom}
### CSS {#css}
まず一番簡単な `honk` のカスタマイズは CSS による配色の変更だろう。  
`honk` では `{datadir}/views/` 下に `local.css` がある場合に追加で読み込むため、ソースファイルを編集しなくとも、`local.css` を追加するだけで CSS は変更できる。  
`honk` では主に使う色を `views/style.css` で CSS 変数を使って指定しており、CSS 変数には `--bg-page, --bg-dark, --fg, --hl, --fg-subtle, --fg-limitied` が宣言されている。  
`--bg-page` は背景色、`--bg-dark` は一部の要素に使われる背景色、`--fg` はテキストやボーダー、アウトラインの色、`--hl` はボタン要素等を強調するのに使われる色、`--fg-subtle` はリプライ等に使われる色、`--fg-limited` はパブリックではない投稿に使われる色となっている。  
合わせて、`honk` では上記 CSS 変数を上書きするための `local.css` を追加するだけで独自の配色、テーマを適用することができる。  

自分の `honk` サーバーでは以下の内容だけの `local.css` を追加して、このブログと近い配色にしている。  

 >         html {
 >           --bg-page: #145;
 >           --fg: #ffe;
 >           --hl: #fa3;
 >           --bg-dark: #023;
 >         }

他には `views/style.css` を編集してマージン、フォントサイズ、ボーダーのスタイルを変更しているが、これも `local.css` にルールを上書きする形で記述すれば適用される。  

### HTMLテンプレート {#html-template}
`honk` では `html/template` [^html-template] パッケージを採用しており、`{viewdir}` にある HTMLテンプレートを起動時に読み込む。  
少し注意が必要なのが、HTMLテンプレートは `{viewdir}` 下にあるものを読み込むが、`local.css, local.js` は `{datadir}/views/` 下にあるものを読み込む点となる。  
また、HTMLテンプレートの変更には golang の `text/template, html/template` に関する知識が必要になる。と言っても、変数や分岐の概念が追加されるだけで、後はほとんど HTML/CSS/JS がそのまま使われている。  
それと、あまり悪く言うつもりはないのだが、`honk` の HTMLテンプレートでは `<p>` 要素をスペース目的で使っていたり、`<p>` 要素の閉じる前に他のブロックレベル要素が見つかった場合は自動で閉じる仕様を活用した部分が多く、`</p>` が HTMLテンプレート内になかったり、そしてインデントがほとんどされていないため、正直読みにくかった。  
HTMLテンプレートを変更する際は、まず好みのコードフォーマットを適用することをオススメする。  

[^html-template]: [template package - html/template - Go Packages](https://pkg.go.dev/html/template)

HTMLテンプレートの変更は、それを前提とした CSS セレクタや JavaScript コードの動作に影響を与える可能性があるため、ブラウザの開発者ツールで確認しながら試し、それから HTMLテンプレートに変更を適用することを推奨。  

### JavaScript {#js}
`honk` の WebUI では JavaScript が使われており、例えば投稿に対する Boost (`bonk`)、リプライ (`honk back`)、削除 (`bonk`)、編集 (`edit`)、ブックマーク (`save`) をサーバーにリクエストする部分に JavaScript が使われている。  
主な JavaScript コードは `views/honkpage.js` にまとめられているが、グローバル変数は HTMLテンプレート `views/honkpage.html` に記述されている。  

### Go Code {#go-code}
見た目だけでなく処理部分もカスタマイズしようとすると Go で書かれたソースコードの変更が必要となる。  
`honk` の各ソースコードファイルの短い説明は `docs/vim.3` にまとめられている。  
自分は `honk` のソースコードを読み込んでいる訳ではないが、カスタマイズし、自分の fork を作っていく中で触れた部分について簡単な解説を書いていく。  

`activity.go` には ActivityPub 形式との変換、他 ActivityPub 実装との相互運用のための処理がまとめられている。  

`fun.go` は *All sorts of fun stuff.* とあるように、投稿に対する色々な処理がまとめられてある。  
具体的には投稿の種類の判定、ユーザー名の正規化、HTML 要素からの不要な属性の削除、フィルターの適用、画像リンクの処理、データベースへの添付ファイルの保存といった処理がある。  
`honk` では投稿内のインライン `<img>` 要素も処理し、画像を保存するが、これを無効化するには `fun.go` 内の `func inlineimgsfor` を常に `return ""` となるように変更すればいい。[^disable-inline-img]  
画像リンクの処理は `func replaceimgsand` で行われる。自分の fork では WebUI 上ですべての `<img>` 要素に `loading="lazy"` を追加するため、その関数内のフォーマット文字列を変更した。[^loading-lazy]  

[^disable-inline-img]: [disable inline img tag · Umio-Yasuno/honk-fisherman@d01a918](https://github.com/Umio-Yasuno/honk-fisherman/commit/d01a918dd411ff3e7de21a989aa36942a5c0764d)
[^loading-lazy]: [add `loading="lazy"` to img tag · Umio-Yasuno/honk-fisherman@10c694e](https://github.com/Umio-Yasuno/honk-fisherman/commit/10c694e4dfae1106427ad99f57965c1fddd02c33)

データベースからの取得は `database.go` に実装されている。何日以内の公開投稿を表示するか等も `database.go` 内で定義されているため、その部分を適切に変更することで、通常は一週間以内の表示期間を変更することができる。  

`sensors.go` には `/about` ページで表示される `honk` のメモリ使用量 (`syscall Maxrss`)、起動時間 (Uptime)、CPU 使用量を取得する処理が実装されている。  

`honk` のユーザーごとに設定可能なフィルター機能、Honk Filtering and Censorship System (hfcs) は `hfcs.go` に実装されている。  
hfcs では正規表現を使うことができるが、ユーザーが設定した文字列の前後に `\\b` が追加したものをフィルタリングに用いるため、日本語等をフィルター対象に指定する場合は `\\b` を追加しないようコードを変更する必要があった。[^hfcs]  
投稿の表示を折り畳むかどうかの判定も `hfcs.go` 内の `func unsee` で行われる。  
`honk` ではフィルター対象は折り畳むが、`sensitive` フラグが有効な投稿はそのまま表示する。自分の fork では簡単な変更を適用して `sensitive: true` な投稿も折り畳むようにしている。[^fix-dz]  
ちなみに `sensitive: true` でも折り畳まないのは意図的なものらしく、その理由を Ted Unangst 氏に聞いた所、Python のインストールに関するスレッドを読むためだけに 20回もクリックして展開することにうんざりしているから、とのことだった。[^thread]

[^hfcs]: [remove `\\b` from filter for non-ASCII word · Umio-Yasuno/honk-fisherman@2b4a010](https://github.com/Umio-Yasuno/honk-fisherman/commit/2b4a010421c9c98cc2874bdbb365e06bc8804144)
[^fix-dz]: [fix DZ: (sensitive) · Umio-Yasuno/honk-fisherman@d0bd6b8](https://github.com/Umio-Yasuno/honk-fisherman/commit/d0bd6b86bb60a83bf83daba5ec25895ac7272adb)
[^thread]: <https://honk.tedunangst.com/u/tedu/h/Dm52M7sZbbT8fGCq16>

`honk` では投稿時に Markdown 記法が使えるが、Markdown パーサーは Ted Unangst 氏が別に公開している `webs` パッケージに実装されている。  
そのため、Markdown パーサー部を変更するには `go.mod` を編集し、ローカルにある `webs` パッケージを参照するようにする等の方法が必要になる。  

## fork of honk {#fork}
レポジトリが公開されている `honk` の fork には以下のようなものがある。  

 * [benjojo/honk-benjojo: The benjojo.co.uk fork of honk](https://github.com/benjojo/honk-benjojo)
 * [mascaldotfr/honk: **customized** honk for https://honk.thebus.top](https://github.com/mascaldotfr/honk)
 * [timkuijsten/honk: Some patches to tedu's honk](https://github.com/timkuijsten/honk)
 * [~petersanchez/honk - netlandish hg](https://hg.code.netlandish.com/~petersanchez/honk)
 * [zevweiss/honk: Miscellaneous patches for Ted Unangst's honk: https://humungus.tedunangst.com/r/honk -- tweaks branch probably rebased semi-often](https://github.com/zevweiss/honk)
 * <https://git.icyphox.sh/honk>
 * [statianzo/honk: fork of honk - honk - Gitodome](https://git.jxs.me/statianzo/honk)
 * [~petersanchez/honk - netlandish hg](https://hg.code.netlandish.com/~petersanchez/honk)
 * [~ols/yeet - sourcehut git](https://git.sr.ht/~ols/yeet)

Ben Cartwright-Cox (benjojo) 氏の fork では、外部ユーザーのアバター画像のサポートと WebUI 上での表示、MP4 と GIF のサポートが追加されており、また外部からの `Like` をログに出力するよう (記録はしない) 変更されている。  
`honk` のクセが強い (と思う) 部分が緩和され、使いやすくなっているように思う。  

 * [benjojo/honk-benjojo: The benjojo.co.uk fork of honk](https://github.com/benjojo/honk-benjojo)

自分の fork は [Umio-Yasuno/honk-fisherman](https://github.com/Umio-Yasuno/honk-fisherman) で公開している。いくつかの変更を各ブランチで分けているが、それらをまとめ、実際に運用しているのが `dream` ブランチとなっている。  
変更は HTMLテンプレートが主で、処理部は先に挙げた細かい修正くらいしかしていない。  
投稿に表示される時刻フォーマットを `2006-01-02 15:04:05` に変更したり、`honkpage.js` を書き直したり、一部レイアウトを flex に置き換えたり、くらい。  

好みの fork をそのまま運用してもいいが、それぞれの変更内容を一部取り込んでいって独自の `honk` を構築する手もある。  
上で挙げた fork の中にもそのようなレポジトリがいくつかある。  

{{< ref >}}
 * [misskey-dev/misskey: 🌎 An interplanetary microblogging platform 🚀](https://github.com/misskey-dev/misskey)
 * [mastodon/mastodon: Your self-hosted, globally interconnected microblogging community](https://github.com/mastodon/mastodon)
 * [Pixelfed - Decentralized social media](https://pixelfed.org/)
    * [pixelfed/pixelfed: Photo Sharing. For Everyone.](https://github.com/pixelfed/pixelfed)
 * [Pleroma — a lightweight fediverse server](https://pleroma.social/)
    * [Pleroma / pleroma · GitLab](https://git.pleroma.social/pleroma/pleroma)
 * [PeerTubeとは？ | JoinPeerTube](https://joinpeertube.org/)
    * [Chocobozzz/PeerTube: ActivityPub-federated video streaming platform using P2P directly in your web browser](https://github.com/Chocobozzz/PeerTube)
 * [さまざまなライセンスとそれらについての解説 - GNUプロジェクト - フリーソフトウェアファウンデーション](https://www.gnu.org/licenses/license-list.ja.html)
{{< /ref >}}
