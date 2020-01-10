---
title: "Intelが開発者向けのPCIeカード型DG1を公開"
date: 2020-01-10T10:11:56+09:00
draft: false
tags: [ "Intel", "Gen12", "DG1" ]
keywords: [ "Intel", "Gen12", "DG1" ]
categories: [ "Hardware", "GPU", "Intel" ]
---

 >  　Intelは1月7日～1月10日(現地時間)に米国ネバダ州ラスベガス市で開催されている世界最大のデジタル関連展示会「CES 2020」の開幕前日となる1月6日に記者会見を行ない、同社が開発している次世代GPUアーキテクチャとなる「Xe」(エックスイー)に基づいたクライアントPC向け単体GPUとなる「DG1」を搭載したノートPCのデモを行なった。  
　そして、9日には、そのDG1の開発用ボード「DG1:Software Development Vehicle」を、第1四半期中にソフトメーカーに出荷する計画であることを明らかにするとともに、PCI Expressのアドインカード形状になっているビデオカードを公開した。 

(引用元: [Intel、同社初の単体GPU「DG1」搭載ビデオカードを初披露。開発者向けボードは第1四半期中に出荷 - PC Watch](https://pc.watch.impress.co.jp/docs/news/event/1228480.html))

ISV（Integrated Software Vendors）向けということで残念ながら自作PC市場で販売されることは今の所予定にないと思われる。  
実機デモが行われたことからGPUとして至って普通に使えるはずで、何かの拍子に流れてくることを期待する。  
<span class="hide">エンジニアサンプリングという訳でもないから比較的流れやすそう</span>

Intelが公式に公開している高解像度画像を見ると、インターフェイスこそx16だが、データリンクはx8までであることがわかる。  
[Images: Intel DG1 SDV - Intel Newsroom ](https://newsroom.intel.com/image-archive/images-intel-dgi-sdv/)  
これはNVIDIAやAMDのGPUでもよくあり（例:RX 5500 XT）、PCI Expressの電力規定でx16インターフェイスでないと75Wをスロットからカードへ供給できない。  
PCI Expressの規格ではx4/x8だと25Wまでとなっているため、DG1の消費電力は25〜75Wの範囲内と推測される。  
{{% xe %}}を統合したTigerlake-U GT2はTDP 25W[^1]ということがわかっており、DG1はそれよりも高いクロックで動作していることとなる。  

[^1]:[2019 Intel Investor Meeting - 2019_Intel_Investor_Meeting_Bryant.pdf#page=11](http://intelstudios.edgesuite.net/im/2019/pdf/2019_Intel_Investor_Meeting_Bryant.pdf#page=11)

カバーとバックプレートに覆われメモリは不明。  

#### DG2 推測

<del>IntelはDG1-SDV（Software Deveiopment Vehicle）によって{{% xe %}}アーキテクチャへの最適化が可能になるとしている。  
また、Intelは{{% xe %}}アーキテクチャがスケーラブルであると語っており、モバイル向けからExascaleのHPCまでカバーする。  
最適化が可能、ということから細部は統一されるはずだ。  
SlicesがNVIDIAで言うGPC、AMDではShaderArrayに該当することになるだろう。</del>  

<del>これらを合わせて考えると、DG1――Gen12LP――{{% xe class="LP" %}}――96EU――が{{% xe %}}アーキテクチャにおける単位となるのではないだろうか。
要は、Gen12LPは1 Slicesあたり6 Dual Sub-Slices、そしてDual Sub-Slicesあたり16EUという構成を取っているが、それより上の{{% xe class="HP" %}}ではSlicesを増やし、固定機能部やL3キャッシュ等も合わせて増やすと考えている。</del>  

<del>大雑把にDG2（{{% xe class="HP" %}}?）のEU数を予想すると、288/384EUとかだろうか。  
ShadingUnitsは2304/3072となり、単純な規模で言えばNavi10に近くなる。  
まあ完全に予想なので自身でも信頼度10%くらいの感覚。</del>  

[Breaking: Intel DG2 Discrete Graphics Details Exposed In 10nm Rocket Lake Driver Leak - HotHardware](https://hothardware.com/news/intel-dg2-discrete-graphics-rocket-lake-driver-leak)  
{{% xe class="HP" %}}には3種の構成があり、それぞれ128/256/512 EU（1024/2048/4096 ShadingUnits）になる、という**話**。  
この場合、1 Slicesあたり8 Dual Sub-Slices（128EU）か、4 Dual Sub-Slices（64EU）となるだろうか。  
個人的な予想では前者にして出来るだけSlices内で処理を回しそうな気がする。  
IntelはGen11からSlices数を増やすのではなく、Slices内のSub-Slicesを増やす方向性を取り、それまではL3キャッシュに含まれていたShared Local Memory（SLM）64KB をSub-Slices内に移動させ、実行効率を向上させた。  
同L3キャッシュバンクにアクセスできるEUが多ければ性能、効率で有利になるのではないだろうか。（バランスの問題はあるが）  
[Architecture Overview for Intel® Processor Graphics Gen11 - Intel® Software](https://software.intel.com/en-us/download/architecture-overview-for-intel-processor-graphics-gen11)（PDF Page15）  

また、Gen11で16 Pixel/Clockとなり、Gen12でもそれを踏襲するならば4 Slicesの512EUは64 Pixel/Clockと他社のハイエンドGPUと同じバランスになる。  

ただ、グラフィック性能を最低限確保するため128EUでは4 Dual Sub-Slices（64EU）構成にするということも考えられる。  

<br>
結局の所素人考えなため、当たってたらいいよね程度に受け止めてほしい。  

#### 外観感想
最後にどうでもいい話。  
カバーを見て最初に思ったのは結構おとなしめだということ。  
Intelは前々からイメージとしてド派手な見た目をしたコンセプトアートを公開していたため、その印象が頭に残っていた。  
開発者向けというのを考えれば無骨な方があっているのかもしれない。  
ただIntelも派手さを一切考えなかった訳ではなく、  
画像を見ると映像端子（スロットカバー）側に小さな穴があり、そこからLEDの光が入るとカバーの流線的なデザインが一気に映える。  
DG1より上のカードがどんなデザインになるか楽しみだ。  
