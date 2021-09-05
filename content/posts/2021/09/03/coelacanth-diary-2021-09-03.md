---
title: "【雑記】そっと踏んでくれ"
date: 2021-09-03T21:58:49+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Diary", ]
noindex: true
# summary: ""
---

それは私の夢だから。  

まずはっきりと言っておきたいのは、自分は現時点で *Aldebaran/MI200* が GPUダイ 2基を合わせて 110CU、各ダイ 55CU の構成ではないか、と考えているということだ。  
{{< link >}} [Aldebaran/MI200 はダイあたり 55CU、トータル 110CU か? | Coelacanth's Dream](/posts/2021/09/01/aldebaran-gfx90a-cu/) {{< /link >}}

| CDNA | Arcturus/MI100 | Aldebaran/MI200 |
| :-- | :--: | :--: |
| GPUID | gfx908 | gfx90a |
| Active CUs | 120 CU | 110 CU?<br> (2x 55 CU?) |
| FP32:FP64 Rate <br> (Arcturus FP32 == 1) | 1 : 0.5 | 2 : 1 ? |
| Memory | HBM2 (2.4Gbps) | HBM2e |
| Memory Bus width | 4096-bit | 8192-bit ? <br> (2x 4096-bit ?) |
| Memory Size | 32 GB | 128GB ? <br> (2x 64GB ?) |

これは雑記で、単なる愚痴であるから、これを書いている背景とか関係するリンクは省くが、書いておく必要があると思ったから書いている。  
ただこういうのは得意ではないから、後からこの雑記を削除するかもしれない。  

機械翻訳の結果は引用にはならないと自分は思う。特に日本語->英語は翻訳結果に怪しい部分が多いし、自分は機械翻訳に向くような短い文ではなく、だらだらと長くかくから尚更だ。  
機械翻訳が吐き出した怪しい文から都合の良いように読み取ったのなら、責任は自身で取ってくれ。そこにこのサイト名は置かないでくれ。  

自分が絶対的に正しいという保証は無いから、こうして吐き出すだけにしておく。前は感情的になりすぎて馬鹿げたことをした。  
このサイトを開設してからもうすぐで 2年になるが、誰かに踏みつけられたような思いをすることは別に初めてではない。  
例えば表テーブルを記事中におくと、そういったものはパッと見て分かりやすく、意味があるように勘違いしやすいからか {{< comple >}} あくまで文の補足として自分は使っている {{< /comple >}}、海外サイトから丸々そのまま使われてたりする。  
それが書く上で抵抗にならないと言えば嘘になるが――いつか自分の側が克服すべきものなのだろう。  
そもそもやっぱり日本語で書かれたオタクのブログが参考にされることがおかしい。自分が情報元にしてるコード、パッチはすべてオープンソースで公開されている。  
ハードウェア情報をメインにした海外サイトの多くが、Twitter や Reddit をそのまま載せただけのものばかりになったと思うのは自分だけか？

 > Tread softly because you tread on my dreams.
 >
 > W. B. Yeats - 1865-1939, Aedh Wishes for the Cloths of Heaven  
 > [Aedh Wishes for the Cloths of Heaven by W. B. Yeats - Poems | poets.org](https://poets.org/poem/aedh-wishes-cloths-heaven)
