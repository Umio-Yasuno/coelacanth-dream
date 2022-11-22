---
title: "Go言語製 ActivityPub サーバーの最小実装 honk を立ててみた"
date: 2022-06-19T02:30:21+09:00
draft: false
categories: [ "Diary", "Note" ]
tags: [ "Honk", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

タイトル通り、個人用に `honk` を立ててみたからその紹介。  
セットアップの流れ等は、`nginx` 周りの知識が必要だが簡易でありドキュメントに従えば問題は無く、また自分自身そこまで詳しい訳でもないため省く。  
`honk` のユニークな部分をメインに紹介していきたい。  

## honk
`honk` は [OpenBSD](https://www.openbsd.org/) の開発者として知られる Ted Unangst 氏が開発した Go言語製の ActivityPub サーバーの最小実装。データベースには SQLite を使用する。  
フォロー数やアカウント数等の接続規模、画像処理時で変わるとは思うが、最小実装のため軽量であり、メモリ使用量は小さい。  
個人用として運用しているが、メモリ 512MB の VPS でも問題なく稼働している。  

 * <https://humungus.tedunangst.com/r/honk>

`honk` は ActivityPub サーバーとして、同じく軽量であることを特徴とする [Pleroma](https://pleroma.social/) との連携を意識した部分があり、chat 機能が実装されている。  

`honk` は ActivityPub に対応し、Fediverse に参加可能なソフトウェアでありながら、SNS らしい機能を一部削っていることも特徴と言える。  
その時々、SNS によって意味合いは変わるが、`honk` で *Like* に対応していない。このことは特に強調されており、ドキュメントには *"Don't be ridiculous."* と書いてあるくらいだ。  
フォロワー情報は `honk` の WebUI 上からは確認できないようになっている。  
通知機能も自アカウントに対するリプライのみに限られている。  
こうした点から `honk` は SNS に対して一歩引いた ActivityPub サーバー、WebUI だと言える。  

用語周りも少し変わっていて、Mastodon の toot や Twitter の tweet に相当するのが *honk(s)*。  
*honk(s)* に対して Actions が用意されており、boost や re-tweet 相当が *bonk*、reply が *honk bonk* とされている。  
他の Actions、*mute* は *honk(s)* とそのスレッドをタイムライン上で非表示に、*zonk* は *honk(s)* の削除、*save* は *honk(s)* の保存、*untag me* はその *honk(s)* に対する replay を隠すが、*honk(s)* 自体は表示したままにする機能となる。  
*edit* は *honk(s)* の編集だが、他サーバーに確実に反映されるかは対応状況によると思われる。*ack* はリプライに対して既読であることを示すが、どのように機能するかはまだ自分が分かっていない。  

 * <https://humungus.tedunangst.com/r/honk/m/honk.1#Viewing>

`honk` の WebUI 上では一週間以内の *honk(s)* だけ通常表示され、長い間表示させたい場合は `/longago` から投稿する必要がある。  
ユーザーごとに RSS を出力する機能があるが、これも一週間以内の *honk(s)* のみが表示される。  
*saved* で保存した *honk(s)* は `/saved` から確認できる。*saved* していない古い投稿を探す時には検索機能が使える。  

投稿には Markdown記法を使うことができ、`honk` の WebUI 上では対応する HTMLタグが反映されて表示される。他サーバーの WebUI、クライアントから見た場合は、そこでの HTMLタグの対応状況による。  
ハッシュタグ機能に対応しており、そしてサーバー内のハッシュタグ一覧が `/o` に表示されるため、`honk` はアーカイブ寄りのマイクロブログとしての運用に向いていると思っている。  

`honk` は Twitter と Mastodon、それぞれからエクスポートされた投稿データをインポートする機能を持っている。  
Mastodon からのインポートではフォローリストに対応しているが、フォロワーは引き継げない。Mastodon のアカウント引っ越し機能と異なるため、`honk` に移行する場合は注意が必要となる。  
tweet を引用する機能があり、`hoot:` の後に投稿への URL を付けることで引用として投稿内に含めることができる。tweet への反応を投稿する方法がはっきりしており、個人的にかなり便利な機能だと思っている。  
他サーバーとの連携関連で言えば、`honk` では投稿内のリンクすべてに `class="mention u-url"` を付与するが、これは意図的なものであり、他 ActivityPub サーバーからのリンクプレビュー生成、それに伴うプリフェッチを防ぐための処置となる。[^preview]  

[^preview]: <https://humungus.tedunangst.com/r/honk/v/fb1d23b65fb0>
[^import]: <https://humungus.tedunangst.com/r/honk/m/honk.8#Import>

`honk` で外部 ActivityPub サーバーのユーザーをフォローしたい場合には、`/honkers` でユーザー URL を入力して `add honker` ボタンを押すか、`/xzone` の `xid` に投稿単体の URL を入力し、読み込まれた投稿から経由して `add honker` をする方法がある。  
`/xzone` は他 ActivityPub サーバーの投稿やユーザーの URL をインポートする場となっている。  

カスタマイズ要素に少し触れると、`./honk admin` からページに表示される固定メッセージの変更と、生成されるアバターアイコンに用いられる 4色を設定できる。  
色は 32-bit hex colors、`RRGGBBAA` の形式で記述する必要がある。末尾 `AA` は省略可能。  
生成されるパターン画像以外をアカウント (`avatar`) に設定することもできるが、一般的な設定方法と異なり、`honk` の起動時に指定した `datadir` 下の `memes` ディレクトリに置かれた画像を、`/account` のプロファイル (`about me`) に `avatar: filename.png` の形式で記述を追加して指定する。  
`memes` 下に置いた画像等は `/funzone` から確認することができる。  
また、`banner: image.jpg` の形式でアカウントのバナー画像を指定できる。  

`avatar: filename.png` の形式でアバター画像を設定できるが、`honk` の WebUI 上では生成される画像へのリンク (`/a?={user_url}`) を用いているため、WebUI 上では反映されない。  
Mastodon 等、他の ActivityPub サーバーから反映されているか確認する必要があるだろう。  

## 感想
立ててから一ヶ月近く経ったが、以前からの Mastodon アカウントが別にあるため、そこまで使い込んではいない。  
それでも、誰にも反応されず、反応があってもそれを知ることができない、ただ趣味に関するつぶやきを投稿する場というのはそれなりに居心地が良いと感じる。  
このブログも同じような場となっているが、投稿の手間は `honk` の方がずっと手軽だ。  
上で書いたように、ハッシュタグや検索機能を用いたアーカイブ寄りの使い方にも向いている。  

先に書いたことから少し外れるが、リプライで反応がもらえれば通知も来るため、こちらから反応を返すことができる。メインのブログに対する反応をこちらでもらえると自分が喜ぶ。  
試しで立てたため数ヶ月の命かもしれないが、他にも活用方法を探っていきたい。  

 * <https://fisherman.coelacanth-dream.com/u/coelacanth>
