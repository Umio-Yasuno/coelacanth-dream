---
title: "【シーラカンスノート】 A9-9820 のさらなる詳細、RB+ の謎、新たな Picasso APU SKU【2020/11/10】"
date: 2020-11-10T16:49:25+09:00
draft: true
tags: [ "Cato", "RDNA_2", ]
# keywords: [ "", ]
categories: [ "Note", ]
noindex: false
# summary: ""
toc: false
---

どこまでをノートとし、どこから記事とするかは曖昧。  


## A9-9820 のさらなる詳細 {#cato-detail}

ダイの素性は分かっても、チップセットや GPU部に謎を残していた **A9-9820 (Cato)** だが、Twitter で活動する [188号 (@momomo_us) / Twitter](https://twitter.com/momomo_us) 氏が実際に購入、分析を行なったことにより、それらの詳細が明らかとなった。  
{{< link >}} [A9-9820 は Xbox One SoC と同一ダイ | Coelacanth's Dream](/posts/2020/10/14/a9-9820-silicon/) {{< /link >}}

氏によると、チップセットは **Xbox One** と同一のものではなく、組み込み向けの第二世代 R-Series *Bald Eagle* [^bald-eagle] と組み合わされる A77Eチップセット[^a77e] を搭載しているようだ。[^tw-a77e]  
互換性については、**Xbox One** のサウスブリッジ (チップセット) も A77Eチップセットも APU とは PCIe で接続する形であるため、そう問題は無いのだろう。[^hc25-xbox-one]  
**Xbox One SoC** では APU側の PCIe から Gigabit Ethernet に接続していたが、**A9-9820** でもそのようになっているのか、チップセット側から接続しているのかは少し気になる所だ。  

[^hc25-xbox-one]: [HC25.26.121-fixed- XB1 20130826gnn.pdf](https://www.hotchips.org/wp-content/uploads/hc_archives/hc25/HC25.10-SoC1-epub/HC25.26.121-fixed-%20XB1%2020130826gnn.pdf)
[^tw-a77e]: <https://twitter.com/momomo_us/status/1323239972913049600>
[^bald-eagle]: [2nd_gen_rseries_product_brief.pdf](https://www.amd.com/system/files/documents/2nd_gen_rseries_product_brief.pdf)
[^a77e]: [AMD A77E FusionController Hub Databook - 53830 A77E Databook.pdf](https://www.amd.com/system/files/TechDocs/53830%20A77E%20Databook.pdf)

そして GPU については、前回は **R7 350** とスペックを合わせた 8CU (512SP), 16ROP 構成としているのではないかと推測したが、  
実際には 6CU (384SP), 8ROP と、フルスペックの 14CU, 16ROP から半分以上も削った構成となっているようだ。[^tw-cato-gpu]  

ただ、既にある GPU (R7 350) とスペックを同じにするから、名前を **R7 350** としている、というのはそれほど外れてはいなかったようで、  
再度調べてみると、**R7 350** には 2種類存在し、その内の OEM向けモデルが 6CU (384SP), 8ROP という構成となっているらしい。[^r7-350-oem]  

[^tw-cato-gpu]: <https://twitter.com/momomo_us/status/1325423345790169091>
[^r7-350-oem]: [AMD Radeon R7 350 OEM Specs | TechPowerUp GPU Database](https://www.techpowerup.com/gpu-specs/radeon-r7-350-oem.c2682)

それでも微妙に謎は存在していて、まず **Xbox One SoC** はジオメトリエンジンを 2基持つことから、GPUの各コア部をある程度まとめたクラスタである Shader Engine もまた 2基と考えられる。  
そして、各 Shader Engine は CU を 7基ずつ持つ。  
6CU, 8ROP という構成を取るとき、各 Shader Engine の CU を 3基ずつ、RB (Render Backend, ROP 4基相当) は 1基ずつ有効にする方法と、  
片方の Shader Engine を無効化し、もう片方の、CU 6基と RB 2基を有効にするという方法がある。  

どちらの方法を取っているかは、Linuxのドライバーで確認するのが手っ取り早いように思うが、`DeviceID: 0x154C` というのはコードには無く、Linux環境でまともに動作するか、認識するかは怪しい。  
Windowsドライバーでは *Kaveri APU* として認識させているようなので、Linux でもドライバーに一部変更を加えてビルドすれば動作させられるかもしれないが、あくまで可能性の話である。  


