---
title: "クールとかスマートとか"
date: 2019-11-18T00:00:57+09:00
draft: true
tags: [ "Hugo", ]
keywords: [ "Hugo", "Theme" ]
categories: [ "Diary", ]
---

あえて古風な見た目をしたものを作るとき、その人は動機として  
クール、スマートといったものを挙げることが多い、と思う。  
例えば、  
[retrosmart-x11-cursors - Github](https://github.com/mdomlop/retrosmart-x11-cursors)  
[cool-retro-term - Github](https://github.com/Swordfish90/cool-retro-term)  
とか。  
retrosmartシリーズは最近のとにかく丸いテーマやアイコンが好きになれないため、  
重宝している。  

だがウィンドウテーマは所々、サイズ調整バーとかタイトル部分が好みに合わなかったため、  
自分でHaikuOS(BeOS)の見た目をほぼ完コピした haikuppoi-openbox-theme を作り、使っている。  
[haikuppoi-openbox-theme - Github](https://github.com/Umio-Yasuno/haikuppoi-openbox-theme)  

 * ボタンは左の閉じるボタンだけ
 * タイトルバーの高さを少し大きくとっている
 * 非アクティブになってもタイトルのフォントが変化しない

というのが特に使いやすいと感じる。（自画自賛）  
とにかく、この場合はスマート（使いやすさ）が動機だ。  

そして、自分でHugoのテーマを作ろうとしたときは、クールが主な動機となった。  

じゃあ古風でクールなものって何だと考え、ブラウン管に行き着いた。  
そうなったのは serial experiments lain が好きというのが大きいだろう。  
ハードウェア関連の話をしたいこのブログにも合ってる。  

そして、CSSの勉強をしながら頑張った途中結果が以下の画像。  
![Theme Test](/image/2019/11/18/theme-test.webp)  

いやあCSSって高機能だなと思わされた。  
最初は画像でなんとかしようとしたが、四隅の影も、ブラウン管と言ったら外せない走査線も、  
CSSのコードだけで実現できた。  
表のAntec行にかかっているのが走査線（っぽいもの）だが、ちゃんとアニメーションする。  
文字のぼやけ加減も、 text-shadow でいけた。  

ただここまで作って思ったのは、見る分には楽しいけど文章としてはただただ読みづらいな、というもの。  
もっとtext-shadowによるブラーの加減、色を調整する必要がある。  
あとスクロールバーが邪魔くさい。  
消えろとまでは言わないが、もう少し存在感をなくしたい。  
baseof.html、list.html、single.htmlの調整もしなければ。   

もっと時間が掛かりそうだが、クールとスマートを両立したものを目指していく。  


ある程度完成したらGithubで公開したいが、サイトタイトル部分に使っている LoveLetterTypewriterフォント（lainのタイトルにも使われている）を再配布していいのかが心配だ。  
ライセンスとしては、シェアウェアで個人使用としては無料、商用としてはライセンス購入、作者に連絡する必要があるらしい。  
[Love Letter Typewriter Font - FontSpace](https://www.fontspace.com/dixies-delights/love-letter-typewriter)  
フォントだけ、使いたい人がそれぞれでダウンロードしてもらう形にすればいける？  

（追記）  
だいぶクールにはなったと思う。  
![Theme Test Cool](/image/2019/11/18/theme-test-cool.webp)  
ボーダーラインが少し目立つがテスト用なので無視。  

lainっぽさを出そうと lightsteelblue を背景色に選んでいたが、  
ブラウン管らしくないため捨て去った。  
結果として、テキストのシャドウも活きるし見やすい。  
○○らしくしたいときは、まず忠実にあるべきなのかもしれない。  

あと気になる大きな部分はページタイトルとスクロールバーぐらいだろうか。  
（追記終了）  


### 追伸
<https://mastodon.cloud/@dsno/103141242935858992>  
当初の見込みより4日遅れて言うことがそれかい。  
どこか予想していた気はするけど。  
mstdn.jp復旧とケースが届くのと、どっちが先か。  
一応ケースの到着予定は12月5日あたり。  
