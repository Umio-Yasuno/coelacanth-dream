---
title: "URL変更のお知らせ"
date: 2020-04-04T01:24:50+09:00
draft: false
#	tags: [ "", ]
keywords: [ "", ]
# categories: [ "Diary", ]
noindex: true
---

このサイトのURLを <https://umio-yasuno.github.io> から <https://blog.coelacanth-dream.com> へ変更しました。  
Github Pageのサービスが優秀で、以前のURLにアクセスしても 301ステータスを返して新しいURLへリダイレクトしてくれるため、ユーザ側で特別修正する必要はありません。  

## ドメイン購入
[Namecheap](https://www.namecheap.com/)にて comドメインを購入。価格は初回1年分で $8.88 + ICANN $0.18 で計 $9.06、日本円で &yen;1,100。  
海外サイトでの購入だったが、個人情報を英語のフォーマットに合わせるのに手間取っただけで、支払いはVISAデビットで行なったため非常に簡単だった。  
設定も、ダッシュボードがとても見やすく、そこでつまづくことはほとんどなかった。  
わざわざ Namecheap で購入したのも、いつかに購入した人のブログを見て、その時の設定画面がわかりやすく、魅力的だと感じたからだ。  

ほんとは netドメインにしたかったけれども、$11.98 と comドメインより高めだったため見送った。*ドリームコム* より *ドリームネット* の方が響きが良い。  
<span class="hide">今[お名前.com](https://www.onamae.com/)を見たら netドメインの登録料金が &yen;599。まあこんなこともある……。</span>

## サイト移行
まず[Configuring a custom domain for your GitHub Pages site - GitHub Help](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site)を参考に独自ドメインへ移行。  
HTTPS認証もGithub側が [Let's Encrypt](https://letsencrypt.org/) で発行処理をしてくれる親切仕様。  

当然これらの操作は反映されるまで少しの時間が掛かる。それを知らずに自分みたく変更を戻したりすると余計に時間が掛かるため注意。  

## これから
独自ドメインを購入したことで、やれることは広がったはずだが{{< comple >}}レンタルサーバで動的サイトを立ち上げたり、マストドンインスタンス立てたり、独自ドメインのメールサーバを立てたり{{< /comple >}}、今の所手を広げるつもりはない。  
やるにしても徐々に学んでいきたい。ドメイン購入だけでも自分にとっては結構な挑戦だった。  
