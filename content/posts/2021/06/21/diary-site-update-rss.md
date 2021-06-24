---
title: "【雑記】最近のサイトアップデートと RSS"
date: 2021-06-21T18:14:11+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Diary", ]
noindex: true
# summary: ""
---

最近このサイトに追加した機能を振り返る、もとい紹介して使ってもらうための雑記。  

{{< pindex >}}

 * [引用部にコピーボタン設置](#copy-btn)
 * [RSS 復権の流れ?](#rss)

{{< /pindex >}}

## 引用部にコピーボタン設置 {#copy-btn}

引用部 (`<blockquote></blockquote>`) にテキストと引用元のタイトル、URL をコピーするボタンを実装した。ボタンは引用部にカーソルを置いた状態 (`:hover`) で表示される。  
切っ掛けは Github ですべてのコードブロックにコピーボタンを追加する変更を行ったことで、自分でもあったら便利だと思い、実装してみた。  
自分は記事の元ファイルからコピーすればいいため、自分以外のこのサイトを見てくれている人が再利用したい時に便利な機能、なはず。  

 * [What’s new from GitHub Changelog? May 2021 Recap | The GitHub Blog](https://github.blog/2021-06-10-whats-new-from-github-changelog-may-2021-recap/#markdown)

ボタンを追加する処理については、とりあえず引用元を示す時に使う Short Code に要素を追加したという雑な方法を取っている。Javascript で追加する方法も試したけど、どっかで不具合が出そうなので安定した方法を取った。そのうち Javascript版に移行すると思う。  

以下はテストとして引用したが、引用元のパッチは [Lunar Lake](/tags/lunar_lake) のコードネームが明かされた時と同様の形式で、Intel Gigabit Ethernetドライバー *e1000e* に次世代の Intel Client Platform **「RPL」** に向けた初期サポートを追加するもの。  
{{< link >}} [Intel Meteor Lake の次世代は「Lunar Lake」、Ethernetコントローラーの初期サポートパッチが投稿される | Coelacanth's Dream](/posts/2021/03/07/intel-lunar_lake/) {{< /link >}}
*RPL* が何の略かは記載されておらず、変に噂やら憶測で記事をかさ増しするのも嫌だったため、こうした場で取り上げる。  
何の略か明かされていた所で自分で語れることは少なかっただろうし、正直次世代のコードネームだけを積み上げても仕方がない。  
次世代次世代と自分でもよく言う (書く) が、果たしてどこまで未来を見据えればいい？  
Intel が *Alder Lake* を 2021年後半にリリース予定にあること、 *Meteor Lake* が初の 7nmプロセスで製造されるクライアント向け CPU となり、2023年に市場へ投入される予定であることを公式的に発表している。[^mtl]  
*Lunar Lake* が *Meteor Lake* の次世代、*RPL* が *Lunar Lake* の次世代だとして、実物が出てくるのは何年後か。  

こうした考えを持っているから、自分はリークとかと相性が悪いことをこのブログを続けていて知った。  
その点 OSS へのパッチは良い。社内である程度が開発が進んで、公開してもいい段階となっているからパッチが公開されている訳で、サポートを追加するということはそのプラットフォーム、ハードウェア (CPU/GPU) のリリースが (ある程度) 近いことも示している。  
自分の情報元はほとんど公式のリリースや、ソフトウェアエンジニア方が公開してくれているパッチであるから、リーカー (leaker) なんて蔑称で呼ばれずに済んでいる、と思いたい。  
雑記と言っても完全に話がずれてしまった。  
結局何に価値を見出すかは人それぞれだ。  

[^mtl]: [Intel CEO Announces ‘IDM 2.0’ Strategy for Manufacturing, Innovation,...](https://www.intel.com/content/www/us/en/newsroom/news/idm-manufacturing-innovation-product-leadership.html?wapkw=%22Meteor%20Lake%22)

 > 		 #define E1000_DEV_ID_PCH_TGP_I219_V14		0x15FA
 > 		 #define E1000_DEV_ID_PCH_TGP_I219_LM15		0x15F4
 > 		 #define E1000_DEV_ID_PCH_TGP_I219_V15		0x15F5
 > 		+#define E1000_DEV_ID_PCH_RPL_I219_LM23		0x0DC5
 > 		+#define E1000_DEV_ID_PCH_RPL_I219_V23		0x0DC6
 > 		 #define E1000_DEV_ID_PCH_ADP_I219_LM16		0x1A1E
 > 		 #define E1000_DEV_ID_PCH_ADP_I219_V16		0x1A1F
 > 		 #define E1000_DEV_ID_PCH_ADP_I219_LM17		0x1A1C
 > 		 #define E1000_DEV_ID_PCH_ADP_I219_V17		0x1A1D
 > 		+#define E1000_DEV_ID_PCH_RPL_I219_LM22		0x0DC7
 > 		+#define E1000_DEV_ID_PCH_RPL_I219_V22		0x0DC8
 > 		 #define E1000_DEV_ID_PCH_MTP_I219_LM18		0x550A
 > 		 #define E1000_DEV_ID_PCH_MTP_I219_V18		0x550B
 > 		 #define E1000_DEV_ID_PCH_MTP_I219_LM19		0x550C
 >
 > {{< quote >}} [[Intel-wired-lan] [PATCH v1 1/1] e1000e: Add support for the next LOM generation](https://lists.osuosl.org/pipermail/intel-wired-lan/Week-of-Mon-20210607/024597.html) {{< /quote >}}

## RSS 復権の流れ? {#rss}

2つとも Impress の記事だが、RSS に再度注目が集まっているらしい。  
RSS 復権の流れは自分にとって嬉しく、というもの自分は積極的な記事の更新通知を行っていないからだ。  
一時期 Twitter で更新通知をやろうとしたが、それだけを行おうとすることは難しく、どこかでそこから外れたことをしてしまうだろうし、見たくないものを見ることになる。  
RSS であれば Hugo でページをビルドする中で生成される `index.html` を配信するだけでいい。毎回自分から通知する必要もない。  
Hugo ではタグ、カテゴリーごとの RSS も生成されるため、興味があるタグ、カテゴリーの RSS だけをフォローする、といった使い方もできる。  

 * [RSSが復権!? 「Google Chrome」にお気に入りのサイトを「フォロー」する機能 - 窓の杜](https://forest.watch.impress.co.jp/docs/news/1325951.html)
 * [RSS復権の流れか。ウェブブラウザー「Vivaldi」の新バージョンにRSSリーダー機能追加【やじうまWatch】 - INTERNET Watch](https://internet.watch.impress.co.jp/docs/yajiuma/1330396.html)

Twitter で思い出したが、記事タイトル下に置いていて Tweetボタンを削除した。  
今自分は Twiiterアカウントを持っておらず、Tweetボタンが正常に機能するか検証できない。そんなものを置き続けるのもどうかと思い、削除した。  
代わりという訳でもないが、URL、Title&URL のコピーボタンを押した時、対応している場合は Web Share API (`Navigator.share()`)[^share] を用いるようにした。  
ただこれもデスクトップPCからは検証できないため、もう少し調節する必要があるかもしれない。  

[^share]: [Navigator.share() - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share)

後は半年振りにアイコンを微調整したり、CSS を修正したり。  
CSS はその時の気分で感じるものが違ったりしていつまでも安定しない。  
