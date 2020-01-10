---
title: "もっとクールを"
date: 2019-11-18T19:06:20+09:00
draft: false
tags: [ "Hugo", ]
keywords: [ "Hugo", "Theme" ]
categories: [ "Diary", ]
---

画像だとロッシー圧縮を使ってるため、かなりぼやけているが実際はもっとマシ、だと思う。  

![Theme Test Cool 1](/image/2019/11/18/more-cool_1.webp)  

![Theme Test Cool 2](/image/2019/11/18/more-cool_2.webp)  

改善点:  

 * scrollbar-width を thin に設定
 * スクロールバー背景を透過
 * テキストの色/ブラー調整
 * タイトル部のカーソル点滅をちゃんとステップで変化するように
 * ページ対応

課題点:

 * スクロールバー周りの設定はFirefoxのみ有効なため、他ブラウザ（Webkit）の設定もしなければいけない
 * Firefoxの開発ツールを使って、モバイル向けの画面も見て調整してはいるが不完全
 * SCSSをあまり有効に使っていない
 * タイトルフォント部を簡単にいじれるようにする（タイトル部にショートコードを使ってスタイル指定でいける？）