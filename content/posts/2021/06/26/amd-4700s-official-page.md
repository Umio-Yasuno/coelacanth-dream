---
title: "AMD 4700S Desktop Kit の公式ページとユーザーマニュアルが公開"
date: 2021-06-26T08:54:17+09:00
draft: false
tags: [ "Zen_2", "AMD_4700S" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

未だにチップの素性は明らかでない **AMD 4700S** の AMD公式ページとユーザーマニュアルが公開された。  
**AMD 4700S** は CPU に *Zen 2 アーキテクチャ* を採用し、コア/スレッド数は 8-Core/16-Thread、メモリには通常 GPU 等に用いられる GDDR6メモリを搭載する。  
その仕様から **Xbox Series X|S** 、**PS5** といったゲーム機向け APU/SoC を転用した CPU ではないかと思われるが、やはりはっきりとはしていない。  
{{< link >}} [AMD 4700S の正体は GPU部を無効化したゲーム機向け APU だったか | Coelacanth's Dream](/posts/2021/04/26/amd-4700s-identity/) {{< /link >}}
公式ページとユーザーマニュアルが公開されたが、**AMD 4700S** 単体のページは用意されておらず、仕様はコア/スレッド数のみが記載されている。  

 * [AMD 4700S 8-Core Processor Desktop Kit | AMD](https://www.amd.com/en/desktop-kits/amd-4700s)
 * [AMD 4700S 8コア・プロセッサー・デスクトップ・キット | AMD](https://www.amd.com/ja/desktop-kits/amd-4700s)
 * [AMD 4700S 8-COREDESKTOP KIT USER MANUAL](https://www.amd.com/system/files/documents/21791450-a-4700s-dtkit-user-manual.pdf)
 * [AMD 4700S 8-COREDESKTOP KIT ユーザーマニュアル](https://www.amd.com/system/files/documents/21791450-a-4700s-dtkit-user-manual-ja.pdf)

## AMD 4700S Desktop Kit の製品仕様 {#spec}

**AMD 4700S Desktop Kit** は Mini-ITXマザーボードに **AMD 4700S (Zen2, 8-Core)** と GDDR6メモリ 8GB または 16GB をオンボード実装、そこに CPUクーラーを搭載した仕様で構成される。  
Desktop Kit としてそれらが最初から組み込まれていることに対し、公式ページではコンポーネントの互換性を心配する必要がなくなる、とアピールしている。 **Xbox One SoC** を転用した APU に **A9-9820** があるが、 **Xbox One** では DDR3メモリをオンボード実装していたのをスロット化したこともあってかメモリ相性が厳しいという話があり、確かにシステムの安定性を得る上では重要かもしれない。[^a9-9820-review]  
メモリの拡張性は無くなるが、AMD は **AMD 4700S Desktop Kit** を自宅、オフィス向けのデスクトップソリューションとしており、そうした一般用途程度ならば最低でも 8GB、最大で 16GB あれば十分だということなのだろう。  

[^a9-9820-review]: [A9-9820搭載マザーボード: 思いつくがままのぶろぐ](https://ktyk.seesaa.net/article/478245701.html)

マザーボードには PCIe x16スロットが実装されているが、PCIe Gen4 ではなく、それどころか Gen3 でもなく Gen2 で、さらには信号としては x4 までとなっている。  
互換性のある GPU のリストも公開されているが、AMD GPU では **RX 500シリーズ (Polaris)** 、NVIDIA GPU では **GT 710/730** 、**GTX 1060** までの Pascal世代が対象となる。  
*Zen 2 アーキテクチャ* 8-Core/16-Thread という CPU に対し、PCIeスロットの仕様は数世代前に遡るような最低限の仕様であり、ゲーミングPCにするのは難しく、AMD が言うような自宅、オフィス向けに活用するのが最適だろう。  
I/O は 2x SATA 6Gb/s のみで、M.2スロットは実装されておらず、PCIeスロットは映像出力用の dGPU で埋まるため、それ以上ストレージを接続するには USB接続を用いることになる。  

FCH には **AMD A77E Fusion Controller Hub** を搭載しており、同チップセットは **A9-9820** 搭載マザーボードにも搭載されていた。[^a77e]  
また、ファームウェアTPM (fTPM) をサポートしているとするが、同時にマザーボード上に TPMピンヘッダーが実装されている。  

[^a77e]: [AMD A77E FusionController Hub Databook - 53830 A77E Databook.pdf](https://www.amd.com/system/files/TechDocs/53830%20A77E%20Databook.pdf)

Geekbench 5 の実行結果に **AMD 4700S** が登場したのと同時に公開されていた AMD のドライバー&サポートページは、その後非公開にされていたが、今回の公式ページ公開のタイミングで復活している。  

 * [AMD 4700S Desktop Kit Drivers & Support | AMD](https://www.amd.com/en/support/desktop-kit/amd-4700s-desktop-kit/amd-4700s-desktop-kit)

BIOS がアップデートされたらしいが (Release Date : 2021-06-24)、Release Note には *「Reliability fix」* とあるだけで具体的な内容は不明。  

**AMD 4700S Desktop Kit** の公式ページ、ユーザーマニュアルは各言語版が用意されており、各種ドライバーは公式から提供されている。  
**A9-9820** が APU であり内蔵GPUが有効されていたのにドライバーが提供されておらず、メモリ相性等もありシステムとして一般用途には厳しい面があったのに対し、  
**AMD 4700S** はメモリがオンボード実装されているためメモリ相性が問題になることは無く、dGPU を必要とする分、公式の最新ドライバーの導入が容易で、他各種ドライバーとユーザーマニュアルもしっかり用意されているというのは、意識したものではないだろうがどこか対照的だ。  
各言語版のページとユーザーマニュアル、それとサポートがしっかり用意されているあたり思ってたよりも **AMD 4700S Desktip Kit** を広くに展開するつもりなのかもしれない。  
PCIeスロットが Gen2 x4 仕様というのはハイエンドユーザーにとってかなり惜しい部分かもしれないが、GDDR6メモリ 8GB or 16GB を搭載した *Zen 2 アーキテクチャ* CPU という魅力に惹かれる人はきっと多いだろう。  

