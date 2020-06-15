---
title: "いかに Zen APU は複雑か 補足 (Winston)"
date: 2020-02-19T08:43:56+09:00
draft: false
tags: [ "Raven", "Renoir", "Picasso", "Raven2", "Dali", "Pollock" ]
keywords: [ "", ]
categories: [ "Hardware", "APU" ]
noindex: false
---

この前の記事では説明が冗長になってしまうため、あえて飛ばしたが、  
[いかに Zen APU は複雑か | Coelacanth's Dream](https://umio-yasuno.github.io/posts/2020/02/16/raven-family-complex/)  
Ravenファミリー内のコードネームには、*Raven* 、*Picasso* 、*Raven2 /Dali /Pollock* 、*Renoir* の他に、  
*Winston* というのが存在する。  
製品名にはMicrosoft Surface専用の、Ryzen 7 3780U、Ryzen 5 3580Uがあげられる。  

しかし、専用品と言っても *Picasso* から大きな違いは見られず、プロプライエタリなドライバーの中身を見ると、一部で Picasso2 との記述も確認できる。  
それでも *Winston* というコードネームが付けられている。  
何故か？  

AMDの真意ははかれないが、ヒントらしきものはある。  

最近よく見るようになったChromium OSへのパッチだが、コミットメッセージにRaven2ベースである *Dali* と *Pollock* について書かれている。  

 > Dali only has 3 i2c controllers, while Pollock has 5. Both boards use  
 > the same socket. So we determine at runtime how many i2c controllers are  
 > available.  
 >
 > 引用元: <cite>[soc/amd/common: Determine # of i2c controllers at runtime (I397b074e) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2057468)</cite>  

i2c[^1]コントローラーの数が *Dali* と *Pollock* で違い、*Pollock* の方が多いとされている。  
どちらも *Raven2* ベースであるため、実際にそれしか持たないのではなく、仕様構成の違いだと推察される。  
またどちらも同じソケット（恐らくFP5）を使用したとあるため、ソケットの仕様ではない。  
*Dali* と *Pollock* の違いがこれだけとは限らないが{{% comple %}}Linux Kernelのamd-gfx、ディスプレイ関連に名があるため、Raven2、Dali、Pollockでそれぞれディスプレイコントローラーの仕様が違う可能性がある{{% /comple %}}、少なくとも i2cコントローラー、仕様構成の違いが新たにコードネームを付けるだけの理由にはなると言える。  

[^1]:[I2C（Inter-Integrated Circuit）とは - IT用語辞典 e-Words](http://e-words.jp/w/I2C.html)

そして *Winston* だが、専用Ryzen搭載Surface発表時のアピールに、以下のものがあった。  

 > Dynamic responsiveness: A fully optimized pen interface delivers unparalleled precision through the revolutionary on-die pen controller.  
 >
 > 引用元: [Microsoft Takes Pole Position in Laptops based on AMD Technology - AMD](https://community.amd.com/community/amd-business/blog/2019/10/03/microsoft-takes-pole-position-in-laptops-based-on-amd-technology)  

on-board でも on-package でもなく、**on-die** とあり、ひとまずこれを信じるにしても、  
いくらMicrosoftのためとは言え、コンソール機ならまだしもラップトップ向けにわざわざカスタム品を設計、生産するとは考えにくい。  
実際、パッケージはRyzen Mobileに通常用いられるFP5で、  
[AMD Ryzen™ 7 3780U Microsoft Surface® Edition Processor | AMD](https://www.amd.com/en/products/apu/amd-ryzen-7-3780u-microsoft-surface-edition#product-specs)  

ダイ自体も他の *Picasso* と同一という見方が世間では強い。  
そのため、元々 *Picasso* 内にあった何らかのコントローラーを利用する形を取ったと思われる。  
これは仕様構成の違い、と言えるはずだ。  

そして、AMDはRyzen 7 3780U、Ryzen 5 3580Uに *Winston* というコードネームを *Picasso* とは別に付けた……

<br>
という推測。  

そう何種類も出ないため、GPUのコードネームが実質SoCをも指し示すことが多いが、  
Zen APUには、GPUのコードネームとして、*Raven* 、*Picasso* 、*Raven2* 、*Renoir* が存在し、  
SoCのコードネームにはGPUのそれとは別に、*Winston* (*Picasso*)、*Dali* / *Pollock* (*Raven2*) が存在すると考えれば、少しはわかりやすくなるかもしれない。  
