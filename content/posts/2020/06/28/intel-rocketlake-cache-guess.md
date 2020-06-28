---
title: "Intel Rocket Lake のキャッシュ構成は Ice Lake と同様か & 推測"
date: 2020-06-28T18:42:51+09:00
draft: false
tags: [ "Rocket_Lake", "Guess", "Gen12" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU" ]
noindex: false
---

先日、*Rocket Lake* の Geekbench 実行結果が登場、それを [APISAKさん (@TUM_APISAK) / Twitter](https://twitter.com/TUM_APISAK) 氏が発見し、話題となった。  
{{< link >}} <https://twitter.com/TUM_APISAK/status/1276482336683511810> {{< /link >}}
{{< link >}} [Intel Corporation Rocket Lake Client Platform - Geekbench Browser](https://browser.geekbench.com/v5/compute/1124595) {{< /link >}}

この実行結果の見るべき点は、*Rocket Lake* の CPUコア数/スレッド数、そしてキャッシュ構成にあるだろう。  
*Rocket Lake S* には 8-Core/16-Thread の構成があり、  
キャッシュ構成は *Ice Lake (Client)* 同様に L1命令キャッシュ 32KB、L1データキャッシュ 64KB、L2キャッシュ 512KB、L3キャッシュはコアあたり 2048KB(2MB) となっている。  

GPU に関しては既出であるため、今回は特に触れない。  
{{< link >}} [Intel、オープンソースドライバーに DG1 と Rocket Lake の関するコードを追加 ――DG1 は 96EU、RKL は 16EU または 32EU | Coelacanth's Dream](/posts/2020/05/08/intel-add-dg1-rkl-oss-driver/) {{< /link >}}

## 推測 {#guess}
あくまでこうなる可能性があるという **推測** であり、*Rocket Lake* に関しては示した以上の情報は持ち合わせていない。  

### コアあたりの容量が増やされていない L3キャッシュ {#rkl-l3cache}

*Rocket Lake S* は L3キャッシュが全体で 16MB、コアあたり 2MBと、*Skylake (Client)* 、*Ice Lake (Client)* から増量された様子はない。  
*Rocket Lake* は MCM構成になるという噂があるが、このことから *Rocket Lake* は単一のチップ構成、もしくはメモリコントローラとメモリの物理層は別チップではなく、CPUコアと同一のチップに搭載されている、と考えられるのではないだろうか。  

AMD は、*Zen 2アーキテクチャ* を採用したプロセッサにおいて、サーバ/ワークステーション/デスクトップ向けには、CPUコア+L3キャッシュと、メモリを含む主要 I/O部を別チップに分ける MCM構成を取った。  
その中で、L3キャッシュを前世代 *Zen アーキテクチャ* から倍の容量に増やしている。  
AMD自身は増やした目的を明らかにしていないが、Hot Chips 31 にて、メモリアクセスレイテンシを減らすのに効果的と発表している。[^hc31-amd-zen2]  
目的を推測するならば、別チップに分けたことで増えたメモリのアクセスレイテンシを出来る限り減らすためと考えられる。  

[^hc31-amd-zen2]: <https://www.hotchips.org/hc31/HC31_1.1_AMD_ZEN2.pdf#page=17>

しかし、*Rocket Lake* はそういったキャッシュ構成の変更が見られない。  
実行結果の詳細でL4キャッシュが検出されていないため、キャッシュ階層が増やされたというのも無いだろう。  

### Rocket Lake は AVX512 に対応するか？ {#rkl-avx}

キャッシュ構成が *Ice Lake (Client)* と同じだとして、アーキテクチャ、実行ポートの構成もまた同じかは気になるところだ。  

*Ice Lake (Client)* は AVX512命令が Port0 で実行可能となっており、[^icl-arch]  
L1データキャッシュの帯域が、ロード 2x 64Byte、ストア 1x 64Byte(or 2x 32Byte) となっている。[^icl-arch]  

[^icl-arch]: [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)

L1データキャッシュ帯域は、AVX512 実行においてデータ供給をスムーズに行なうためと考えられ、*Skylake (Client)* では、ロード 2x32Byte、ストア 1x32Byte だったのが、AVX512命令に対応した *Skylake (Server)* ではそこから倍の帯域、*Ice Lake (Client)* と同様のものとなっている。  

しかし、キャッシュ帯域の増加は性能向上に繋がるが、消費電力の増加にも繋がり、  
[テクニカルライターの大原雄介](http://yusuke-ohara.com/) 氏は、*Skylake (Server)* アーキテクチャを採用する *Skylake-SP* に対し、以下のように述べている。  

 > こうした帯域の増加は、性能の向上にも貢献する一方で確実に消費電力も底上げしてしまう。このあたりが、Skylake-SPにおける消費電力の多さに繋がっているのではないかという気がする。
 >
 > 引用元: [新Xeonで何が変わったか - 内部構造を解説 (1) 既存のSkylakeコアからの変更点 | マイナビニュース](https://news.mynavi.jp/article/20170725-xeon/)

実際、L1データキャッシュ帯域の増加のみが原因とは言い切れないが、*Skylake-SP* 同様に *Skylake (Server)* アーキテクチャである *Skylake-X* では消費電力、消費電力比性能の悪化が確認されていた。[^sky-x-1][^sky-x-2]  

[^sky-x-1]: [Core i9-7900XとCore i7-7820Xを試す - いち早く登場する10コアと8コアCPUのパフォーマンスを徹底検証 (18) ベンチマークテスト「消費電力測定」 | マイナビニュース](https://news.mynavi.jp/article/20170710-corex/18)
[^sky-x-2]: [ASCII.jp：Core i9-7900X＆Core i7-7800Xレビュー、全コア4.5GHz OCで見えた新世界 (3/3)](https://ascii.jp/elem/000/001/514/1514627/3/)

*Ice Lake (Client)* アーキテクチャの CPUを 14nmプロセスで製造するとしたら、同様の問題が発生すると考えられ、*Rocket Lake* はデスクトップ向けとしては高いTDP枠を取るか、動作クロックを抑える必要が出てくるだろう。  
AVX512 に対応しないのであれば、L1データキャッシュ帯域を *Ice Lake (Client)* や *Skylake (Server)* と同じものにする必要性は薄まり、そして *Skylake (Client)* と同じ帯域にすれば問題は回避できる。  
欠点として、当然 AVX512 に対応しないのだから、*Ice Lake* 等に用いたアピールを使えなくなる。  
ただそれに関しては前例があり、*Sunny Cove* コアを搭載する *Lakefield* は対応しているはずの AVX/2/512 命令への対応が無効化されていた。  
そして、*Ice Lake* の特徴の1つである、`AVX512-VNNI` 命令による高速な推論 **Intel DL Boost** の代わりか、GPU による推論の実行性能がアピールされていた。  
{{< link >}} [Intel、Lakefiled を正式発表 & AVX命令には非対応？ | Coelacanth's Dream](/posts/2020/06/13/intel-lakefiled-without-avx/) {{< /link >}}
*Rocket Lake* は GPU部が [Gen12](/tags/gen12)アーキテクチャとなり、前世代から強化されるため、同様のアピールは可能だろう。  
デスクトップ向けプロセッサとして、どこまで AVX512 への対応が求めてられているか、という疑問もある。  

よって、*Rocket Lake* が AVX512 に対応せず、実行ポート、キャッシュ帯域がそれに合わせて調節される可能性もあると考えられる。あくまで推測。  

*Rocket Lake* にはまだまだ謎があり、気になる点も多い。今後出てくる情報が楽しみだ。  
個人的には、*Rocket Lake* への最適化のためのコードがコンパイラ、バックエンドに追加されれば対応する命令範囲も判明するため、それを待ち遠しく思っている。  


{{< ref >}}

 * [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)
 * <https://twitter.com/TUM_APISAK/status/1276482336683511810>
 * <https://www.hotchips.org/hc31/HC31_1.1_AMD_ZEN2.pdf>
 * [新Xeonで何が変わったか - 内部構造を解説 (1) 既存のSkylakeコアからの変更点 | マイナビニュース](https://news.mynavi.jp/article/20170725-xeon/)

{{< /ref >}}
