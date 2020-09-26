---
title: "【雑記】シーラカンスが見た夢　―― コードネーム云々"
date: 2020-09-26T04:14:12+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Diary", ]
noindex: false
# summary: ""
toc: false
---

## コードネーム云々
### Lucienne {#lcn}

初めて *Lucienne APU* の話題を取り上げた時、学の無い自分は Lucianne という誤字から *Lucienne Bisson* を連想することが出来ず、[AMD Renoir の兄弟？ 現れるは Lucienne](/posts/2020/06/20/amd-lucianne-apu/) なんてタイトルにしたが、コードネーム元の *Lucienne Bisson* は女性であり、*Pierre-Auguste Renoir* の実の娘という。  
ただ、個人的には APU にもこの関係性を適用するには抵抗を覚える。  
APU の世代で言えば、(恐らく) CPU と GPU 部には *Renoir* から変わらず、変わるとしてもプロセスくらいに思う。  
だから兄弟姉妹、*Renoir* の妹と言う方がしっくり来る。  
意味は無いけど。  

それと今でこそ情報が少ないため、*Renoir Refresh* と言われることが多いが、*Picasso* をわざわざ *Raven Refresh* と言い換えないように、変更点が明らかになれば *Lucienne* が定着するのかなと思う。  

### RDNA 2 {#rdna_2}
以前に [AMD RDNA 2 のコードネーム良し悪し](/posts/2020/07/27/rdna_2-codename-good-and-bad/) という記事を書いたが、AMD の Linux ソフトウェア開発チームの方が Reddit でしたコメントによると、X11 Color と魚をランダムに組み合わせたもので、テーマを決めて星の名前から取るよりも商標の認定が通過しやすいという。[^codename-random]  {{< comple >}} 懺悔じみて来たな。{{< /comple >}}  
*Vega* からの *Arcturus* といった、コードネームから象徴的なものがなくなってしまうのは少し寂しいが、生物的なコードネームは GPU のイメージをユニークにしてくれるため好きだ。  

[^codename-random]: <https://www.reddit.com/r/Amd/comments/ixu9xc/amd_adds_dimgrey_cavefish_navi_23_to_linux_opengl/g69175c?utm_source=share&utm_medium=web2x&context=3>

## Phoronix による NCNN ベンチマーク結果 {#ncnn}

自分が以前に [ACOバックエンド](/tags/aco) の検証に用いた [Tencent/ncnn](https://github.com/Tencent/ncnn) のベンチマークプロファイルが OpenBenchmarking に追加され、Phoronix の Michael Larabel 氏が多くの GPU で実行、結果を公開している。  
{{< link >}} [NVIDIA GeForce vs. AMD Radeon Vulkan Neural Network Performance With NCNN - Phoronix](https://www.phoronix.com/scan.php?page=article&item=realsr-ncnn-vulkan&num=1) {{< /link >}}

Linux とオープンソースに理解があり、大規模なベンチマークを実行できるのは Phoronix の Michael Larabel 氏しかいないため、価値ある記事だと言える。  
ベンチマークにおいて難しいのはシステム環境を揃えることだと思う。それに左右されないベンチマークソフトだと、単にプロセッサのピーク性能を測るものとなり、実使用とはかけ離れてしまう。  

結果はと言うと、**Radeon VII** が **TITAN RTX** を追い越してトップに立っている。  
ただピーク性能では勝っているはずの **RX Vega 56** が **RX 5700 /XT** よりも遅く、このあたりは画像の入力出力、ネットワークモデルの読み込み等でメモリバウンドなベンチとなっているような気がする。  
しかし、メモリ帯域が同じ 448GB/s でありピーク性能で優っている **RTX 2080** よりも、**RX 5700 /XT** が速く、他社 GPU との比較ではただメモリ性能で決まる訳でもないようだ。  
ここでは [ACOバックエンド](/tags/aco) が性能に大きく寄与しているように思える。  

