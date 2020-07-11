---
title: "デフォルト放置は時に悪、マニュアル読まずは常に悪"
date: 2019-11-15T00:19:57+09:00
draft: true
tags: [ "Hugo" ]
keywords: [ "Hugo", ]
categories: [ "Diary" ]
---

まったくもってその通りだと思う。  
経緯としてはまずGithubのコミットログだ。  

私はコミットログを更新履歴として利用しようと考えていたのだが、  
いざ見るとcss部のリンクが毎回変わり、結果すべてのindex.htmlが書き換わることとなり、  
それがすべて表示されるため、とても更新履歴には使えなかった。  

原因として、このサイトに使っている[Solar-Theme](https://github.com/bake/solar-theme-hugo)のbaseof.htmlが、  

     {{ $style := resources.Get (printf "css/colors-%s.scss" (.Site.Params.scheme | default "dark")) | toCSS | minify | fingerprint }}
     <link rel="stylesheet" href="{{ $style.Permalink }}">

となっており、scssをcssに変換する際、パイプライン最後段でフィンガープリントを付与するようになっていた。  

そこをさくっと削除し、css名は固定され、平穏が訪れた。  

それと同時に、自動的に生成される tags と categories を無効化、削除した。  
理由としては、tagsがこのサイトで一番容量が多いディレクトリと化していた。
自分で使おうにも目的のタグに辿り着きにくく、必要ないと感じられる。  
自前で tags を用意するのか、諦めてまた自動的に生成させるかはまだわからない。  

あとテーマ、というよりcssも出来れば誰も見られていない内に自分で作っておきたい。  
