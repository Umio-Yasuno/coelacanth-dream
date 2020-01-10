---
title: "公式が常に正しいとは限らない"
date: 2019-11-09T04:34:12+09:00
draft: false
tags: [ "CPU", "GPU" ]
categories: [ "Hardware", "AMD" ]
---

[official-athlon-3000g-spec](https://www.amd.com/en/products/apu/amd-athlon-3000g#product-specs)  
![official-athlon-3000g-spec](/image/2019/11/09/official-athlon-3000g-spec.webp)  
<br>
[gigabyte-athlon-3000g-spec](https://www.gigabyte.com/Ajax/SupportFunction/Getcpulist?Type=Product&Value=6236)
![gigabyte-athlon-3000g-spec](/image/2019/11/09/gigabyte-athlon-3000g-spec.webp)  
<br>
どっちなのさ。  
少し前に見た時はThreadripper 3960X/3970X が12nmということになっていたが、さすがに修正されていた。  
ついでに言うとRX590の Peak Pixel Fill-Rate も間違っていたりする。  
公式では49.54 GP/sと記載されているが、49.44 GP/sのはず。  
( 1.545[GHz] * 32[ROP] = 49.44[GP/s] )  
いやこれはどうでもいいか。  
<br>
ただタイトルは結構重要だと思う。  
初期の資料での内容が後期では変わっていたりとか、ある資料では簡略化しすぎて、詳細に解説にしたものだとズレが生じていたり。  
そんなことはよくあると思う。  
最近ではZen2のIOダイが14nmだったのが、今ではcIODもsIODも12nmとなっている。  
ある程度柔軟でいることが大切なのだと思う。  
そんな時、その時正しいと思った情報を書き、後から間違っていると気付いたらそれも書けるこのブログ?は役に立つはず。  
自分限定になってしまうだろうけど。  
<br>
mstdn.jp復旧までネタが持ちそうにない。