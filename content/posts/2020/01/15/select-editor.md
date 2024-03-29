---
title: "行き着く先はVim"
date: 2020-01-15T04:36:18+09:00
draft: true
tags: [ ]
keywords: [ "", ]
categories: [ "Diary", ]
noindex: true
---

このブログ的なサイトを始めてからまだほんの2ヶ月しか経っていないが、その間に記事の編集に使用するテキストエディタが2回変わった。  
それらエディタの軽い紹介と使ってみての所感。  

#### ReText
[retext-project/retext: ReText: Simple but powerful editor for Markdown and reStructuredText](https://github.com/retext-project/retext)  

最初に選んだシンプルなMarkdownエディタ。  

まず何故Markdownエディタなのかと言うと、このサイトの静的なHTMLを生成するのに[Hugo](https://gohugo.io)を使用しており、Hugoでは記事自体をMarkdownで記述する。  
Markdownをまだまともに習得していなかった私には、ReTextのライブプレビュー機能が必須だった。  
書いたことはあったが、それは基本的なことを書いてあるサイトを見ながら手探りでようやく改行や見出し、画像リンクができた、程度のものだった。  
そんな私は、Markdownエディタとして信頼できるシンタックスハイライトと、ReTextがエディタとして至って普通に使えたことのおかげでMarkdownを**ほぼ**習得できたと言ってもいい。  

ReTextから離れた理由は、プレビュー機能はHugoにも備えられていたというのが大きい。  
ReTextは編集ウィンドウ内を2つに分割し、右にプレビューを表示する形で、  
Hugoの場合は、Hugoの機能でローカルサーバーを立て、Webブラウザでそれにアクセスし生成結果を確認するというもの。  
Hugoの方が画面使用の自由度が高く、実際に生成した結果を見られるのだから問題点に気付きやすい。さらに言えば、ReTextとHugoとでMarkdownの解釈に微妙な違いがあり、ReTextではプレビューの意味を成さなかった。  
もっとショートカットキーを使いたいというのもある。  

つまりは巣離れだ。  
いざ飛び立って次に選んだエディタだが、だいぶ迷った後に[Gedit](https://wiki.gnome.org/Apps/Gedit)に落ち着いた。  
それも少しの間だが。  

#### Gedit
シンプルでありながら多くの機能を備えたパワフルなテキストエディタ。  
初めてUbuntuを使った頃よくお世話になった思い出。  

使っていて一番イイと感じたのはファイル関連の機能。  
最初から入っている「ファイル参照パネル」プラグインを有効にすれば気軽に、素早くファイルを新規タブに開くことができ、日付ごとに管理しているのと相性が良かった。  
タイトルバー内左にある開くボタンからの履歴検索も強力だった。  

それなのにGeditからも離れたのは、一言で言えばGNOMEアプリケーション故だ。  
GNOMEアプリは必要とする表示領域が大きい。  
サイドパネルを使おうとすればそれはさらに大きくなり、大体FullHDの横半分、960pxがないと辛い。  
そもそもサイドパネルが便利と言っても、編集の内、ファイルを開くのに使う時間なんてわずかなものであり、そこをちょっと速くするためだけに表示領域を割くのは効率が悪い。  
タブ部分も大きく、表示できて3タブ、4タブ以上になると切り替えが一気に面倒くさくなる。  

FullHDのモニター1枚しかないため、調べながら編集をしているとキーボードとマウスの移動が増える。  
他の記事からコピペしたい時は、まず探し出して開いて範囲を選択してコピー、編集中の記事に戻ってペースト。  
これをマウスと少しの有用なショートカットキーでしなければならない。  
きっと手のキーボードとマウス間の移動に嫌気がさしたのだと思う。  

そして履歴検索が強力なことが災いし、起動する度に外部ストレージのHDDにアクセスが発生、ｺﾞｺﾞという音と共にもたつきを感じる羽目になる。  

GNOMEのテーマを変更するとかGeditの便利なプラグインを探すといった解決策もあったのかもしれないが、そこまでするほど気に入ってはいなかったため、さっさと次のエディタを探しに行った。  

そして行き着いたのがVimだった。  

### Vim
言わずと知れた長い歴史を持つ伝統的なエディタ。  
VimはそれまでにもこのサイトのテンプレートHTML、CSSや他にちょっとしたBashスクリプトを書くのに使っていて慣れていたし、気に入ってた。それなのに記事には別のエディタをあてていたのは、ひとえにVimが日本語に向いてないと広くに言われ、自分自身それに納得していたからだ。  
例えば、Vimのコマンドを打つ時にIMEが有効になっていると、日本語で入力されてしまい、エンターを押してもかき消されて何も起きないということがままある。  

だが全くできないということではなく、偉大な先駆者たちが使いやすくする方法を残してくれている。  

[Vimで日本語を編集するいくつかの方法 - Qiita](https://qiita.com/murashitas/items/f2be0dda2a4498cb7985)  
[vim/gvimで日本語を使いやすくする - fudist](https://sites.google.com/site/fudist/Home/vim-nihongo-ban/vim-japanese)  
[日本語のカーソル移動の改善: 文節単位のWORD移動(W,E,B)プラグインと、句読点に移動するmap](https://gist.github.com/deton/5138905)  

#### 使用環境
それと、自分はIMEにfcitx+mozcを使っているが、デフォルトだと日本語キーボードの変換/無変換キーがIMEの有効/無効の切り替えに割り当てられており、これを変換をIME有効に、無変換をIME無効に割り当て直すとVimでの誤爆がぐっと減る。  
Macの英数/かなのようなものだ。  
この設定は Mozcの設定 > キー設定の選択 から行なうことができる。  

後はターミナルエミュレーターにはIMEが対応したフレームワークを使用されてある必要があり、メジャーなGTK3、QTならば基本問題はない。  
自分の場合、GPUによる高速なレンダリングを特徴とした[kitty](https://sw.kovidgoyal.net/kitty/)を主に使っていたため、引っかかってしまった。  
今はLXTerminalをメインに据えている。<span class="hide">設定を覚えれば xterm でいいかも。</span>  

フォントには Migu 2M Regular を設定している。  
これが一番英字と日本語文字のバランスが良いように感じた。  
それまでは Dejavu Sans Mono がお気に入りで設定したが、やはり日本語対応したフォントでないと厳しかった。  

Vim自体の設定では最初、Markdownで改行を意味する行末の二重スペースにシンタックスが対応していないのに悩まされたが、以下を ~/.vimrc に追加することで解決した。  

	syntax on  
	highlight MarkdownTrailingSpaces ctermbg=green guibg=green  
	match MarkdownTrailingSpaces "\s\s$"

参考: [syntax highlighting - Highlight double space in markdown - Vi and Vim Stack Exchange](https://vi.stackexchange.com/questions/3539/highlight-double-space-in-markdown)  

そしてVimでやはり良いと思うのは、豊富で強力なコマンドさえ覚えればキーボードから手を離すことなく快適に編集を行えることだ。  
そこに関してReText/Geditと比べると雲泥の差がある。  
あるファイルを編集中に他のファイルからコピペしたいという時は、
	
	:new <ファイルパス>

でさくっと開いて、コピペすればいい。  
Vim内のタブ移動ももちろんキーボードで完結する。  


結局の所、エディタはVimに行き着くのだろう。  
LinuxのディストリビューションにおけるDebianのように。（これは自分がUbuntu -> Xubuntu -> Lubuntu -> Sparky Linux -> Debian という遍歴を持つことの経験と偏見による。）  
ある程度それに慣れ、自分好みにカスタムする手段を覚えると、最低限に抑えたものを好むものだ。  
このサイトのテーマだって最初こそ他者が作成したのをありがたく使わせてもらっていたが、そう経たない内に自身でテーマを作り始めた。  
性格というよりは本能に近いものだと思う。  
現在このサイトはGithub Pagesサービスを利用しているが、いつかは自分で用意したドメインとサーバーで運用するかもしれない。  
ただ、それをもう一歩先考えるのはGithubから警告が届くリポジトリサイズ 1GB（らしい）に到達してからだろうし、それまでこのサイトが続いているかはやはり怪しい。

VimやDebianはツールとしてのソフトウェアよりも、その人にとっての居場所だと実感する。  
静かに佇み、集まる鳥たちの鳴き声を巨大な幹で柔らかく受け止める巨木のようでもある。  
