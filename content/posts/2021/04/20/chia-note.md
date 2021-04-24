---
title: "Chiaマイニングはどれだけストレージを消費するか"
date: 2021-04-20T19:36:04+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Software", "Note" ]
noindex: false
# summary: ""
---

[PC Watch](https://pc.watch.impress.co.jp) より、HDD/SSD が品薄になるとの予想が出され、一部では既に大容量HDDの購入制限を設けていることを知った。原因は仮想通貨「Chia」のマイニング需要によるものとされる。  

 * 参考: [HDD/SSDが品薄。今度は仮想通貨“Chia”のマイニング特需 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1319789.html)

アルゴリズムやブロックチェーンネットワークの仕組みとかは自分には難しすぎるのですっぱり諦め、あくまでも自分が気になった、どれだけストレージを使用するのかを整理する。  

## Chia

Chia では Plot と呼ぶファイル、あるいはブロックに事前に計算した大量のハッシュを格納する。  
ブロックチェーンが課題ブロックを送信した時、ユーザーはそれに最も近いハッシュが Plot 内に存在するか確認し、存在した場合にその分のブロックを得ることができる。  
Plot は土地であり、そこに大量のハッシュを格納することを種蒔き (seeding)、ユーザーはストレージスペースとそれらを行うコンピューターを提供することから農場主 (farmer) とも表現される。ハッシュを確認し、農場主に報告する者はハーベスタ (harvester) とされる。  

Plot のサイズは k=n のパラメーターで決まり、現在、最小パラメーターは k=32 に設定されている。  
Plot は CPU、DRAM、I/O リソースを大量に消費するプロセスであり、k=32 の場合、一時的には 332 GiB (356.5 GB)、最終的には 101.4 GiB (108.9 GB) の容量を使用する。  
ただそれは使用する容量であり、総書き込み量としては、ビットフィールド有効時は 1.6 TiB、無効時は 1.8 TiB にもなる。ビットフィールドを有効にすれば総書き込み量を 10% 近く減らせ、作業用の一時ディレクトリが HDD のように比較的遅いストレージにある場合は高速化という点でも効果的だが、使用するメモリ量は増加する。また、古い CPU ではビットフィールドを有効にしても効果がないこともある。  
k を 32より大きい値に設定することも可能だが、空きスペースをギリギリまで使いたい場合かベンチマーク用途以外に必要性は無いとされてる。  

1.6 TiB または 1.8 TiB の書き込み量を必要とするのだから、Plot には高速な SSD が向いていると言える。  
ただ、Plot では常時ほぼ 100% の使用率で書き込みを行うため、コンシューマー向けの SSD にある SLCキャッシュといった一時的な書き込み速度を向上させる機能は効果が小さくなる。  
また、これが SSD を使う上で最大の懸念点だが、耐久性を考えるとコンシューマー向けは Plot に向いておらず、総書き込み量が、メーカーの保証する TBW (Terabytes Written) を超えると性能やデータの保持に問題が出ると思われ、代わりに耐久性の高いエンタープライズ、データセンター向けの SSD が最適だとしている。[^ssd-dc]  
それらはコンシューマー向け製品よりも高価だが、中古のものが eBay や Craigslist でお得に手に入れられると紹介している。中古という点が不安に思われるが、それでもコンシューマー向けより高い耐久性を持った SSD を入手できるということなのだろう。  
高速なストレージでは他に DRAM 上にファイルシステムを作成する tmpfs や RAMディスク等が思い浮かぶが、DRAM だと容量単価が SSD の 10倍近くに上がる。それに 356.5 GB 以上 {{< comple >}} 近いメモリ容量で 384 GB (12x32GB) とかだろうか {{< /comple >}} のメモリを搭載できるシステムは構築のハードルが高い。
その上 Chia公式ブログの記事を読むに、大量のメモリを消費して 1つの Plot を作成するよりも、複数の SSD を用意して並行に Plot を作成した方が効率が良いと思われる。[^chia-blog]  

[^ssd-dc]: [SSD Endurance · Chia-Network/chia-blockchain Wiki](https://github.com/Chia-Network/chia-blockchain/wiki/SSD-Endurance)

Plot を保存し、ハッシュを確認する段階のプロセスでは CPU、DRAM、I/O 性能をさほど必要とせず、Arm CPU を搭載する SBC に大量の HDD を接続する構成が公式で紹介されている。[^farming-sbc]  
Plot を作成を作成するための作業用ディスクには SSD、Plot の保存先には HDD が向いていると言える。  

Chia は、Bitcoin や Ehereum のように 1基あたり 75W〜300W の電力を消費する GPU、それよりも大きく、物によっては 数千W 以上にもなる ASIC ではなく、10W 程度の HDD をマイニングに用いるため、省電力なブロックチェーンネットワークであることを特徴とする。  
また、誰もが持っているストレージの空きスペースを活用するため中央集権的になりにくいとされている。  
それでも Chiaマイニングが HDD/SSD の在庫に影響を与え、また専用の SSD が開発中というニュースが報じられていることに対し、公式Twitterアカウントではニュースのリンクをツイートしているが、まだ特別反応を示してはいない。[^chia-tw]  

[^chia-tw]: <https://twitter.com/chia_project>
[^farming-sbc]: [Reference Farming Hardware · Chia-Network/chia-blockchain Wiki](https://github.com/Chia-Network/chia-blockchain/wiki/Reference-Farming-Hardware)
[^chia-blog]: [Chia plotting basics](https://www.chia.net/2021/02/22/plotting-basics.html)

{{< ref >}}
 * [SSD Endurance · Chia-Network/chia-blockchain Wiki](https://github.com/Chia-Network/chia-blockchain/wiki/SSD-Endurance)
 * [Beginners Guide · Chia-Network/chia-blockchain Wiki](https://github.com/Chia-Network/chia-blockchain/wiki/Beginners-Guide)
 * [Chia plotting basics](https://www.chia.net/2021/02/22/plotting-basics.html)
 * [Network Architecture · Chia-Network/chia-blockchain Wiki](https://github.com/Chia-Network/chia-blockchain/wiki/Network-Architecture)
 * <https://www.chia.net/assets/proof_of_space.pdf>
{{< /ref >}}
