---
title: "ActivityPub サーバー、honk v0.9.91 がリリース"
date: 2023-03-21T06:21:34+09:00
draft: false
categories: [ "Diary", "Note", ]
tags: [ "Honk", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Ted Unangst 氏により、2023/03/17 付で honk v0.9.90/91 がリリースされた。  
`honk` は Go 言語で書かれた、最小実装、軽量であることを特徴とした ActivityPub サーバー。  

 * [humungus - honk](https://humungus.tedunangst.com/r/honk/d)

v0.9.90 がリリースされた直後に、フォロー周りの機能で変数のミスがあり、すぐに修正した v0.9.91 がリリースされた。  
v0.9.90/91 にはいくつかの修正と機能の追加、削除が含まれる。その一部を紹介したい。  

まず Tim Kuijsten 氏のパッチが取り込まれ、honk に CSP ヘッダが導入された。  
CSP (Content-Security-Policy) は読み込むリソースの種類を制限することで、XSS といった脆弱性とそれを用いた攻撃の影響を軽減するセキュリティレイヤーとなる。  
honk では以下のルールが適用し、インラインスクリプトや CSS は HTMLテンプレートから別ファイルに移された。  

 >         "Content-Security-Policy", "default-src 'none'; script-src 'self'; connect-src 'self'; style-src 'self'; img-src 'self'; report-uri /csp-violation"

 * [timkuijsten/honk: Some patches to tedu's honk](https://github.com/timkuijsten/honk)
 * [1469: apply CSP patches. no more inline script or css. from timkuijsten](https://humungus.tedunangst.com/r/honk/v/8e69552fa0bc)
 * [1470:  add an emu peeker to the honkform. from petersanchez, adapted to new csp rules](https://humungus.tedunangst.com/r/honk/v/6bb39edcea30)

次に wonk 機能が削除された。  
wonk は v0.9.7 (2022-02-12) で追加された、Wordle 風の単語を推測するゲーム。だがどのように使うかは自分には不明であり、分からないまま削除された。  
Ted Unangst 氏のサーバーで使われているのは見たことがあり、外部に配信しない wonk を投稿するユーザーがいるといった感じだった。  

また、外部の投稿をインラインで引用として展開する機能が追加された。  
実装としては、ユーザーオプションで `InlineQuotes` が有効かつ、投稿内に `>https://[^\s<]+<` の正規表現に一致するリンクがある場合、その投稿を読み込み、末尾に `<blockquote>%s</blockquote>` として追加する形となっている。  
投稿単体へのリンクは各 ActivityPub サーバーの実装で異なり、現状では Mastodon, Misskey, Pleroma, honk に対応している。  

 * [1464: some basic qt support](https://humungus.tedunangst.com/r/honk/v/30d874501a1d)
 * [1471: make quote inlining an option](https://humungus.tedunangst.com/r/honk/v/7ad586912de2)

 >         var re_mast0link = regexp.MustCompile(`https://[[:alnum:].]+/users/[[:alnum:]]+/statuses/[[:digit:]]+`)
 >         var re_masto1ink = regexp.MustCompile(`https://[[:alnum:].]+/@[[:alnum:]]+/[[:digit:]]+`)
 >         var re_misslink = regexp.MustCompile(`https://[[:alnum:].]+/notes/[[:alnum:]]+`)
 >         var re_honklink = regexp.MustCompile(`https://[[:alnum:].]+/u/[[:alnum:]]+/h/[[:alnum:]]+`)
 >         var re_r0malink = regexp.MustCompile(`https://[[:alnum:].]+/objects/[[:alnum:]-]+`)
 >         var re_roma1ink = regexp.MustCompile(`https://[[:alnum:].]+/notice/[[:alnum:]]+`)
 >         var re_qtlinks = regexp.MustCompile(`>https://[^\s<]+<`)

Emoji ピッカーも今回のリリースで追加されている。  
honk でのカスタム Emoji は `<dataDir>/emus/` 下に PNG もしくは GIF ファイルを置くだけで追加でき、Emoji の指定には拡張子を除いたファイル名が使われる。  

自分のファークでは全体メニューのサイドバー化や、投稿の表示やメニューの整理を行った。  
それと `/about` ページなどに表示するメッセージを変更するのに、honk 独自のエディタで操作する必要があるのが面倒だったため、Markdown ファイルをメッセージ部に使えるようにする簡易的な変更を適用した。  

honk は Go言語で書かれた ActivityPub サーバーの最小実装であることを特徴とするが、あくまでも Ted Unangst 氏の個人的なプロジェクトであり、機能の変更は好みに左右される、と考えている。  
また、自分のフォークでは UI を主に変更しているが、今回のリリースにはそうした見た目に関わる変更はほとんど無い。そのため受け入れられる可能性は低いと思い、UI を変更するパッチや提案は送らないつもりでいる。  

{{< ref >}}
 * [コンテンツセキュリティポリシー (CSP) - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/CSP)
{{< /ref >}}
