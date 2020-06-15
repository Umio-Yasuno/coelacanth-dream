---
title: "Athlon 3000Gの正体"
date: 2019-11-27T20:14:16+09:00
draft: false
tags: [ "Athlon", "Radeon", "GCN", "GFX9", "Raven", "Picasso", "Raven2", "gfx909", "gfx902" ]
keywords: [ "Athlon" ]
categories: [ "Hardware", "AMD", "APU" ]
---

Athlon 3000GにRaven2とPicasso、どちらのダイを使っているかが明らかとなった。  

さっさと答えを言うと、Raven2だった。  
<https://twitter.com/momomo_us/status/1199354469227487232>  

### 謎

Athlon 3000Gは名前が出た当初は、ソース元のM/BのサポートCPUリストからPicasso 12nmだとされていた。  
そのため、多くは精々Ryzen 5 3200G/3400Gからスペックを削った程度のものと思い込んでいたはずだ。  
だが公式発表では14nmとのこと。  
そして私を含め、思い込んでいた人たちは、何だこいつはとなった。  
Athlon 200/220/240GEがいるのに、今更OC解禁したくらいの製品を追加するとも考えにくい。  
それにAthlon GEもBIOSのバージョンによっては可能だったため、ユーザーにとってもさほど魅力的でもない。  

この謎は発売されてからも少しの間続き、一部国内メディアからはこう発表される程だった。  
[Athlon 3000Gの国内販売、開始日と価格が判明 - 11月23日に格安の6千円台で - マイナビニュース](https://news.mynavi.jp/article/20191119-925936/)  

 > 製造プロセスが12nmのCPUと、14nmのGPUを組み合わせた、Zen+アーキテクチャをベースとしたPicasso世代の新APUと見られる。

ハードウェア情報を取得、表示するソフトウェアではPicassoとなっていたこともあり、謎と混乱に拍車をかけた。  

ASCII.jpではその点をAMDに確認しており、14nmが正しいという回答を得ている。  
[4GHzオーバーが狙える！OC可能なAPU「Athlon 3000G」の実力とは - ASCII.jp](https://ascii.jp/elem/000/001/977/1977161/)  

正直、それでも実はPicassoなんじゃないかと、私はAMDよりもM/Bメーカー等の情報を信じていた。  
ごめんなさい。  

ただ、公式の仕様ページでは当初GPUクロックが1100MHzではなく、1000MHzと記載されてたりと、初っ端から鵜呑みにするわけにもいかないのだ。  
柔軟でいることが大切――というのは[過去の記事](/posts/2019/11/09/spec-wrong)で書いたことだ。  

混乱は確実な情報が出るまで続く。
確実にAPU/GPUの正体を知る方法は私が知っている限りでは２つあり、そのどちらかが明らかにされ、混乱が（少なくとも個人的に）終結することを期待していた。  

#### 殻割り
１つは、殻割りしてダイを直接確認する、ハードウェア的な判別方法。  
Raven2とPicassoはダイサイズが違っており、Raven2の方が小さい。Picassoが長方形なダイ形状をしているのに対し、Raven2は正方形に近い。  
そのため、これが手っ取り早い方法だった。  

#### chip family
安全なソフトウェアでの判別方法。  

Linux環境において、ユーザーモードで使うドライバ（RadeonSI、RADV）では詳細なハードウェア情報を表示することができる。  

ターミナルエミュレーターにて、まず

	 AMD_DEBUG=info
		RADV_DEBUG=info

を入力。  
それに続いて、AMD_DEBUGならOpenGLを必要とする（呼び出す）コマンドを、  
RADV_DEBUGならVulkanを必要とする（呼び出す）コマンドを入力する。  

    例:
		AMD_DEBUG=info glxinfo -B
		RADV_DEBUG=info vkcube

そうするとコマンド結果の前に、このようなハードウェア情報が出力される。  
![AMD_DEBUG=info result](/image/2019/11/27/amddebug-info-result.webp)  
family の値を[amd_family.h](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/common/amd_family.h)と照合。  
今回の場合は、 CHIP_POLARIS11 とわかる。  

Athlon 3000Gなら、値が70なら CHIP_RAVEN（Picasso）、71ならCHIP_RAVEN2と判断することができた。  

#### 結果
Twitterにてハードウェアの検証や情報収集を積極的におこなっている[188号@momomo_us - Twitter](https://twitter.com/momomo_us)氏が  
Athlon 3000Gの殻割りを実行したことでRaven2だと判明した。  
とても届くとは思えないのが悲しいが、私を含め多くの人の好奇心を満たしてくれたことに感謝を述べたい。  

### おまけ
#### Raven(Picasso)とRaven2の違い
情報が多く公開されている組み込み向けのV1000（Raven）、R1000（Raven2）のを元に比較してみる。  

| RV | V1000 | R1000 |
| :--- | :---: | :---: |
| CMOS | 14nm | 14nm |
| Chip | Raven | Raven2 |
| CPU Core/Thread | 4/8 | 2/4 |
| CPU L3$ | 4MB | 4MB |
| GPU CUs | 11 | 3 |
| GPU ROPs | 8 | 4 |
| GPU L2$ | 1MB | 0.5MB |
| Memory Channnel | 2 | 2 |
| Memory Speed (MT/s) | 3200  | 2400 |
| Display Output | 4 | 3 |
| PCIe Gen3 Lane | 16 | 8 |
| USB 3.1 Gen2 (10Gb/s) | 4 | 4 |
| Type-C | 2 | 1 |
| USB 3.1 Gen1 (5Gb/s) | 1 | 0 |
| USB 2.0 | 1 | 2 |
| SATA Port | 2 | 2 |
| 10 Gigabit Ethernet | 2 | 2 |
| TDP Range | 12-54 W | 12-25 W |
| DieSize  | 209.78 mm2 | 154.68? mm2 |

わかりやすい違いとしては、CPUのコア/スレッド数、GPUのCU数、PCIeレーン数、Typc-C数、USB 3.1 Gen1の有無が挙げられる。  
ダイサイズの削減には、GPU CU数の減少（11->3）、PCIeレーン数（16->8）が貢献していると思われる。  
TDP的にはV1000も12-25Wの製品があるが、[V1202B](https://www.amd.com/en/product/7291) [V1605B](https://www.amd.com/en/product/7281)  
そのTDPを求める層に売るには、V1000（Raven）のコストがきついため再設計し、ダイが小さめなR1000（Raven2）を出したのだろうか。  
ダイが小さい分、Ravenよりも消費電力が低くなり、Ravenであった一部のバグも修正されているため、電力効率ではより優秀である、というのもあるかもしれない。  

<br>
どーでもいい話だが、R1000シリーズ発表と同時にAtari VCSに採用、2019年7月に出荷開始されることになっていたが、  
延期され2020年中までとなった。  
[The Atari VCS retro console can now be preordered from GameStop and Walmart - The Verge](https://www.theverge.com/2019/6/11/18661879/atari-vcs-gamestop-walmart-preorders-price-available)  
さらには2019年10月にシステム設計担当者が半年以上の給料未払を理由に辞任していると報じられた。  
[Atari VCS設計担当が給料未払いを理由に辞任 - IGN](https://jp.ign.com/atari-vcs-tvgame/39072/news/atari-vcs)  
いろいろと大丈夫だろうか。  
