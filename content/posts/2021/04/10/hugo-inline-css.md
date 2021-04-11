---
title: "Hugo で SCSS から Inline CSS に変換する"
date: 2021-04-10T23:09:17+09:00
draft: false
tags: [ "Hugo", ]
# keywords: [ "", ]
categories: [ "Diary", "Software" ]
noindex: false
# summary: ""
---

このサイトの支えている静的サイトジェネレーター [Hugo](https://github.com/gohugoio/hugo/) のメモ。  

## SCSS -\> Inline CSS

このサイトでは一部の CSS をレンダリング開始速度、CSSファイルがダウンロード、パースされるまでの表示のため HTMLファイルにインライン、`<style>...</style>` 内に記述しているが、  
他の CSS は SCSS形式で記述し、サイズ削減のため Hugo の Minify機能を活用している中、インラインCSS だけベタに書いてた。  
それをどうにかしたくて試行錯誤した結果見付けた方法のメモ。  

Hugo のドキュメントにあるように、通常 SCSS から CSS に変換、CSSファイルを生成する場合は以下のように記述する。  

 * [Hugo Pipes Introduction | Hugo](https://gohugo.io/hugo-pipes/introduction/)
 * [SASS / SCSS | Hugo](https://gohugo.io/hugo-pipes/scss-sass/)

 > 		{{- $main_style := resources.Get "css/main.scss" | toCSS | minify -}}
 > 		  <link rel="preload" href="{{ $main_style.Permalink }}" as="style">
 > 		  <link rel="stylesheet" href="{{ $main_style.Permalink }}">

それをベースに、変換結果をインラインに展開したい場合は以下のようになる。1行にまとめることもでき、下がその例。  
`<styke>...</style>` 内に置く場合は `safeCSS` をやらないと正常に変換結果が出力されないので注意。  

 >      <style>
 >        ...
 > 		  {{- $main_style := resources.Get "css/main.scss" | toCSS | minify -}}
 > 		  {{- $main_style.Content | safeCSS -}}
 >        ...
 >      </style>
 >      ~~~
 > 		  {{- (resources.Get "css/main.scss" | toCSS | minify).Content | safeCSS -}}

これをテンプレート部に使うことで、SCSS形式で記述しながらインラインCSSとして展開でき、レンダリング開始速度を向上させることができる。速度向上には後に表示させたい要素をインラインCSS側で表示しないよう設定する必要があるが。  

SCSSの変換結果はキャッシュされるらしく、個々の SCSSファイルをテンプレートで取得、変換する方法だとファイル処理のためかビルドに時間が掛かるようになる。  
そのため、メインとする SCSSファイルを決め、そこで `@import "./other.scss";` という風にしてまとめた方が吉。  


