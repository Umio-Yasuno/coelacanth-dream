---
title: "AMD、HBM2 を採用した Navi GPU、Radeon Pro 5600M を 16インチMacBook Pro向けに発表"
date: 2020-06-15T22:46:34+09:00
draft: false
tags: [ "Navi12", "GFX10", "gfx1011" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMD は 2020-06-15付で、16インチ MacBook Pro向けの **Radeon Pro 5600M** を発表した。  
{{< link >}}[New AMD Radeon™ Pro 5600M Mobile GPU Brings Desktop-Class Graphics Performance and Enhanced Power Efficiency to 16-inch MacBook Pro for Users On-the-Go | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/new-amd-radeontm-pro-5600m-mobile-gpu-brings-desktop-class){{< /link >}}
**Radeon Pro 5600M** は、モバイルフォームファクタにおいてデスクトップクラスのグラフィクス性能を実現するように設計されたとしている。  

*AMD RDNAアーキテクチャ (Navi1x)* であり、[Navi10](/tags/navi10)、[Navi14](/tags/navi14)が採用した GDDR6メモリとは違う HBM2 を採用するGPU。  
このことから、**Radeon Pro 5600M** のベースとなるのは、これまで名前が出てきつつも具体的な姿を現さなかった、*Navi12* と考えられる。  

## インデックス

 * [スペック](#spec)
   * [Navi12 答え合わせ](#navi12-grading)
 * [HBM2採用において効率的な内部構造](#design-for-hbm2)

## スペック {#spec}
上記のニュースリリースでは情報が少ないが、MacBook Pro向けの **Radeon Pro 5000Mシリーズ** のページには詳細が記述されている。  
{{< link >}}[Radeon™ Pro 5000M Series for Apple MacBook Pro | AMD](https://www.amd.com/en/graphics/radeon-apple-5000m-series){{< /link >}}

| Radeon Pro 5600M | |
| :--- | :---: |
| WGP(CU) | 20(40) |
| &emsp;Stream Processors | 2560 |
| Peak Engine Clock | 1035 MHz |
| Memory Type | HBM2 |
| Memory Interface | 2048-bit |
| Memory Speed | 1.54 Gbps |
| Memory Bandwidth | 394 GB/s |
| FP32 (TFLOPS) | 5.3 |
| TGP | 50 W |

HBM2 はダイあたり2chとなるため、実質的なメモリ速度は 1.54Gbps とされているが、メモリクロックは 770MHz となる。  

クロックが最大でも 1035MHzと **Radeon Pro 5300M/5500M** よりも低めだが、CU数が多いため、ピーク演算性能では勝っている。  
そして電力効率に優れた HBM2 をメモリに採用したこともあり、倍近いメモリ帯域を確保しながら、TGP(Total Graphis Power) は他と同じ 50Wを保っている。  

RevisionID はパターンから Apple向けSKUに使われやすい `0x40` か `0x41` だろうか？[^1]  

[^1]: [AMDGPU Database: Device ID/ Revision ID/ Product Name | Coelacanth's Dream](/posts/2019/12/30/did-rid-product-matome-p2/#navi12-gfx1011)

### Navi12 答え合わせ {#navi12-grading}
これまで限られた情報から推測していた *Navi12* の、一部答え合わせをすると、まあほとんど外れてた。  
自分の考えていた、*Navi12* はサーバ向けとかクラウドゲーミング向けのGPUというのは、まだ外れたとはっきりしてはいないが可能性としては薄くなり、  
対して、その考えと衝突モバイル向けというのは合っていて、自分ではソースを見つけられなかった HBM2採用という話も合ってた。  

自分が情報に正確性を求めるのは、基本的な部分で間抜けだからなんじゃないかという気になってくる。いや実際そうなのだろう。  
それでも正確性が大切という基本は変わらないし、情報を追い、あれこれ考えを巡らすのはそれなりに楽しいため、懲りずに続けていきたい。  

そう言えば末尾の数字が同じ *Vega12* も HBM2、Apple MacBook Pro向けだったなと。  
`12` にはそういった意味を持たせているのだろうか？  
いや *Polaris12* は違うか……  

それと [Navi12](/tags/navi12) について初めてまとめたのが 7ヶ月前ということに恐怖を覚えた。  

## HBM2採用において効率的な内部構造 {#desing-for-hbm2}
気を取り直して、  
AMD は **Radeon Pro 5600M** の内部構造を示す画像を公開している。  
{{< link >}}<https://www.globenewswire.com/NewsRoom/AttachmentNg/5b3001ca-447c-4b58-921e-919895b6afc9/en>{{< /link >}}

[Renoir](/tags/renoir)の件があるため、画像全てを鵜呑みには出来ず、実際詳細部分は誤魔化されているだろうが、大まかな配置までは嘘を付いていないはずだ。  
それを念頭に置き、じっくり噛むように見てみると、まず HBM2 PHY と思われるユニットが左右端中央付近に配置されているのが目につく。  
そしてその周囲に RenderBackend、L1cache、L2cache らしきユニットがあり、他よりも大きい HBM2 PHY がダイサイズを肥大化させないよう、注意深く、効率的に配置されているように見て取れる。  
ここが個人的に結構興奮した部分で、それまでの HBM2採用GPU はダイがデカくなりがちだった。  

*AMD GCNアーキテクチャ* では ShadeEngine を単位とし、その中で CU をずらっと並べ、その塊の周囲に RenderBackend、L2cache を配置するような設計を取ってきた。  
CU を並べて配置するのは、L1 Data /Instruction Cache を最大 4CU で共有するというのが理由の1つではないかと思う。  
{{< link >}}[Polaris10/Polaris11のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/03/30/polaris10-polaris11-dieshot-guess/){{< /link >}}
{{< link >}}[Vega10/Vega20のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/03/24/vega10-vega20-dieshot-guess/){{< /link >}}
それによって、*Vega10* は ShaderEngine 等のコア部から離れた場所に HBM2 PHY がぽつり、ぽつりと配置されていた。  
反対側の ShaderEngine も等しい距離でアクセスできるよう、HBM2 PHY 2基の内、片方を反対側に配置してしまうと、ただでさえ大きめな *Vega10* のダイサイズがさらに大きくなってしまうだろう。  

しかし、*AMD RDNAアーキテクチャ* からは L1 Data /Instruction Cache を共有する CU数が 2基で固定され、WGP が新たな単位となった。  
これにより配置の自由度が増し、他よりも大きい HBM2 PHY ユニットがダイサイズを喰わないよう効率的に配置され、それでいて左右対称的な美しい内部構造が可能となったのではないだろうか。  
L1cache、L2cache、HBM2 PHY の流れるような配置もイイ。  

繰り返しとなるが、あくまで **大体の** 内部構造を示す画像、あくまでそれっぽい画像であるため、自分が言っていることがどこまで正しいかはわからない。  
それでもやはりあの画像は美しく、興奮を覚えるものだ。  
