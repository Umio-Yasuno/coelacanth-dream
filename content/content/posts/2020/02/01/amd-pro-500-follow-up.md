---
title: "AMD 「PRO 500」チップセット続報 & まとめ"
date: 2020-02-01T21:12:42+09:00
draft: false
tags: [ "PCH",  ]
keywords: [ ]
categories: [ "Hardware", "Chipset" ]
noindex: false
---

先日書いた「PRO 500」及び「PRO 560」について、もう少し情報を手に入れることができた。  

### 「PRO 500」 ≠  「PRO 560」
まず、他に「PRO 560」を搭載したシステムがないか調べたところ、HP EliteDesk 705 G5 Small Form Factorが見つかった。  
[HP EliteDesk 705 G5 Small Form Factor Business PC Specifications | HP® Customer Support](https://support.hp.com/us-en/document/c06461848)  

そしてありがたいことに、HPはマニュアルとしてEliteDesk 705 G5のマザーボード画像を解説付きで公開している。  
[HP EliteDesk 705 G5 Small Form Factor PC Manuals | HP® Customer Support](https://support.hp.com/us-en/product/hp-elitedesk-705-g5-small-form-factor-pc/27015959/manuals)  
(リンク先 PDFファイル:[HP EliteDesk 705 G5 Small Form Factor PC - Motherboard Viewer](http://h10032.www1.hp.com/ctg/Manual/c06466792))  

それを見ると、そこには（たぶん）多くの人が見慣れているであろう外部チップセットの姿が。  
![PRO 560 external chipset](/image/2020/02/01/amd-pro-560-external-chipset.webp)  

つまり前回の記事で自分が書いた、「PRO 560」は外部チップセットレス、というのは間違っていたこととなる。  

AMD公式サイトにあった以下の文言と矛盾するかもしれないが、  
[Socket AM4 Platform | AMD](https://www.amd.com/en/products/chipsets-am4)

 > To satisfy customers who value the smallest form factors, AMD’s X300, A300, and PRO 500 chipsets provide processor-direct access for excellent performance.

それとは別に、HPに「PRO 500」を搭載したシステムがあった。  
[HP Desktop Pro A G3 Specifications | HP® Customer Support](https://support.hp.com/sg-en/document/c06522206#AbT2)  

Lenovoでは「PRO 560」を搭載したシステムのみだったことから、思い違いをしてしまったが、  
「PRO 500」と「PRO 560」は**別物**、ということだろう。  

「PRO 500」はA/X300同様に外部チップセットを必要としないプラットフォームであり、  
「PRO 560」は外部チップセットを必要とする。  

### 「PRO 500」 = 「B300」 ?
またさらにややこしい話となってしまうが、  
HP Desktop Pro A G3の資料が *6390ENL* と *6362EEAP* の2バージョン見つかり、前者の方が古く、最下部に 2019 October 2 とある。  
[4AA7-6390ENL.pdf](https://www8.hp.com/h20195/v2/GetPDF.aspx/4AA7-6390ENL.pdf)  
[4AA7-6362EEAP.pdf](https://www8.hp.com/h20195/v2/GetPDF.aspx/4AA7-6362EEAP.pdf)  

そして古い *6390ENL* では、「PRO 500」ではなく「B500」とあり、  
さらには注釈を辿ると、

 > AMD B300 with AMD Ryzen™ 5 PRO 2400G & AMD Ryzen™ 3 PRO 2200G processors.

 と記載されている。  

簡単に表すと、「B300」 = (B500) = 「PRO 500」、となる。  

最下部にJanuary 2020 と記された、最新バージョンと思われる *6362EEAP* では「PRO 500」になっていることから、{{< comple >}}どんな理由があったかは知らないが{{< /comple >}}名称変更がAMDの方からあったのだろう。  
AMD公式サイトでは何故かハブられていた「B300」がまさか最新のチップセットとして復活していたのも驚きだ。  

そのまま「B300」の名を使わなかった理由だが、  
元々「A300」と「B300」の違いがわかりにくい、もしくはほとんどなかったため、まずラインナップをシンプルにするため「B300」を消し、  
その後、OEM向けにRyzen 3000シリーズを正式にサポートしているチップセット(規格)として出そう、という時に「B300」を再利用しよう  
となったの**かもしれない**。あくまで推測。  
ただ、それならPCIeGen3までのサポートとなった理由も納得いくのではないだろうか。  
しかし「B300」は外部チップセットを必要としないため、仕様変更が一部加えられている可能性もある。  

### 「PRO 560」の正体は
そうなると気になるのは「PRO 560」の正体だ。  

まず、パッケージ外観からしてX570ではない。PCIeGen4もサポートしていないことからも、それはわかる。  
既存の300/400シリーズチップセットのどれかか、新設計かとなる。  

PCIeGen3 x1があるが、300/400シリーズでも一応仕様としてはPCIeGen3 x2まで出せるはずであり、  
「PRO 560」もOEM向けにRyzen 3000シリーズを正式にサポートしているチップセットとしてリネームしたもの**かもしれない**。  
もしくは、以前より名前が出ているOEM向けの「B550A」チップセットと近い存在である可能性もある。  
そちらもチップセット側のPCIe Gen4はサポートされないという話だ。  
