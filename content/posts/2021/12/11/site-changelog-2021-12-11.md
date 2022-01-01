---
title: "[雑記] Site Changelog 2021-12-11"
date: 2021-12-11T22:16:01+09:00
draft: false
tags: [ "Hugo", ]
# keywords: [ "", ]
categories: [ "Diary", ]
noindex: false
# summary: ""
---

サイトテンプレート (HTML/CSS) を全面的に見直して修正した。その記録。  

{{< pindex >}}
 * [パーサーを goldmark に](#goldmark)
    * [Render Hook](#render-hook)
 * [CSS](#css)
    * [色の統一](#color)
    * [エフェクト](#effect)
 * [その他](#other)
    * [サイト内検索](#site-search)
    * [RSS](#rss)
{{< /pindex >}}

## パーサーを goldmark に {#goldmark}

Markdownパーサーに、Hugo を使い始めた時のデフォルトだった blackfriday を使い続けてきたが、v0.87.0 からついに非推奨となった。[^hugo-v0_87_0]  
そこで仕方なく goldmark に移行したが、苦労する点がいくつかあった。  

[^hugo-v0_87_0]: [Release v0.87.0 · gohugoio/hugo](https://github.com/gohugoio/hugo/releases/tag/v0.87.0)

まず blackfriday からフットノート周りが変わった。クラス名と構造の変更はわずかで、CSS側の修正に時間はそう掛からなかった。  
バックリンクが blackfriday の \[return\] から &#x21a9;&#xfe0e; (`&#x21a9;&#xfe0e;`) に変更され、goldmark 側にそれを変更するオプションはあるようだが、Hugo では有効化されていないため、CSS の `:before { content: '[return] ' }` で以前の状態に近づけている。  

地味に苦労したのが blackfriday と解釈の異なる部分で、blackfriday では空行を挟んで 2つの引用部がある場合、1つの引用部 (`<blockquote>...</blockquote>`) を出力していたのだが、goldmark では個別の引用部を出力する。  
自分は引用部と引用元を 1つのブロックにまとまるようにしているが、その 2つの間に空行があると分かれて出力されてしまう。  
これは元の Markdownファイルを修正して対応した。曖昧な部分に依存して Markdown を書いていたのだから、これに関しては自分が悪い。  

また、goldmark では HTML を直接 Markdownファイルに書いた場合、HTML部直後に空行を挟まずに文章を置くと、文章が段落と解釈されず、テキストそのままを出力する。  
その部分は何の要素にもならないため、レイアウトが崩れることになる。  
これも Markdown側を修正して対応。発生しているのが段落間の調節のために `<br>` を置いているケースがほとんどだった。  

修正のため、1、2年前のファイルを確認、編集することになったが、その頃は文体とか全体的なテンションに今と違いがあり、それを読まなければならないことの方が辛かった。  

### Render Hook {#render-hook}

goldmark では 一部 Markdownレンダリングを上書きする Render Hook が使えるのだが、外部リンクへの `target="_blank"` 等を設定するためにリンク部分 (`render-link.html`) は必ず用意しなければならない。blackfriday では設定部分で切り替えできていた。  
とはいえ Hugo の公式サイトに例が用意されてあり、それで十分事足りる。  
 
 * [Configure Markup | Hugo](https://gohugo.io/getting-started/configuration-markup/#markdown-render-hooks)

だがテンプレートを扱う関係か、そうして上書きされたリンク部の後に改行が差し込まれることがあり、ブラウザはそれを空白として扱うため、意図しない空白、ずれが発生した。  
これは下記のように、最初と最後に空の `{{- "" -}}` を置くことで回避できる。  
Goテンプレートでは `{{ }}` で変数、関数にアクセスでき、ハイフンを追加すると、追加した側をトリミングがトリミングされる。[^go-template]

[^go-template]: [Introduction to Hugo Templating | Hugo](https://gohugo.io/templates/introduction/)

 > 		{{- "" -}}
 > 		<a href="{{ .Destination | safeURL }}"
 > 		  {{- with .Title }} title="{{ . }}" {{ end -}}
 > 		  {{- if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener" {{ end -}}>
 > 		  {{- .Text | safeHTML -}}
 > 		</a>
 > 		{{- "" -}}

それと Shortcode 内では今まで Markdown からの変換に `markdownify` を使っていたが、goldmark ではその場合 Render Hook が適用されない。適用したい場合は `.Page.RenderString` を使う必要がある。  

今の所 goldmark への移行で発生した問題で目に付いたものは修正できているが、他にも気付いていない問題があるように思える。  
移行にあたって予想される問題とその対応方法を記したガイドなんかがあったりはしないため、手探りで進めなければならないのも対応を難しくしている。  

## CSS {#css}
### 色の統一 {#color}

テキスト、ボーダー、背景……それらの色を出来る限り統一し、1つの SCSSファイルにまとめた変数で管理するようにした。  
色もまずベースの色を決めて、そこから SASS/SCSS のビルトイン関数 `lighten(), darken()` 等を使って調整するようにした。  
色の決定が今まで雑だっただけだが、色の管理と決定がかなり楽になった。  
ファイル自体も `@import` で用途別に分割して管理するようにした。  

SASS/SCSS を使う人にとっては当然のことなのかもしれないが、勉強しながら継ぎ足していったこともあり、無駄な部分は多かった。  
大体のレイアウトはほとんど変えていないが、細かい部分で最適化している、できているはず。  

### エフェクト {#effect}

このサイトではそれっぽさを出すために、CSS で走査線っぽいエフェクトと中央部をわずかに明るく見せるエフェクトを追加している。  
これがないと全体がのっぺりしたものになるため、個人的には重要な部分なのだが、一時期スクロールが遅い原因になっていた。  
デスクトップでは感じないが、モバイルだと明らかに遅く、アドブロッカーを使ってエフェクトを適用している要素を消すと解消した。  
削除以外の解決方法を探すのに時間が掛かったが、結局原因は `mix-blend-mode` を指定していながら、`isolation` は指定していないことが原因だった。  
`mix-blend-mode` はエフェクトを重ねる上で色に影響を与えるが、自分の使い方では影響が小さかったため、そのプロパティ自体を消した。値を調節するのを面倒に感じていたのもある。  

## その他 {#other}
### サイト内検索 {#site-search}

Github API を利用したサイト内検索を実装した。ログを確認したところ公開したのは 2021/07 だが、未だに (Beta) を付けている。  

中身としては、Github Pages のレポジトリとは別に管理している Markdownファイルのレポジトリに検索を掛けて結果を JSON形式で受け取る。そしてパスをキーに、Hugoテンプレートとして用意、出力した JSONファイルから記事のタイトルと時刻を取得し、検索結果として表示している。  

### RSS {#rss}

最初はサイトマップ同様にすべての記事分を出力させていたが、参考に他のサイトの RSS を見たところ最近の記事のみを出力しており、そこでようやく更新チェックならすべては必要ないことに気付いた。今は 10記事分だけ出力するようにしている。  


{{< ref >}}
 * [mix-blend-mode - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode)
 * [isolation - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/isolation)
{{< /ref >}}
