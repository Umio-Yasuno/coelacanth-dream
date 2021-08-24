---
title: "Intel Architecture Day 2021 個人的まとめ　―― Gracemont, Golden Cove, Alder Lake"
date: 2021-08-24T02:56:55+09:00
draft: false
tags: [ "Alder_Lake", "Sapphire_Rapids" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU", "GPU" ]
noindex: true
# summary: ""
---

時間と体力の関係で遅れた上に一部だけを取り上げた、個人的な Intel Architecture Day 2021 のまとめ。  
{{< xe class="hpg" >}}、{{< xe class="hpc" >}} はその内に、たぶん。  

 * [Intel Architecture Day 2021](https://www.intel.com/content/www/us/en/newsroom/resources/press-kit-architecture-day-2021.html)
    * [Presentation Deck: Intel Architecture Day 2021](https://download.intel.com/newsroom/2021/client-computing/intel-architecture-day-2021-presentation.pdf)

## Alder Lake
### Gracemont (Efficient-core) {#gracemont}
*Gramont* はスループット、マルチスレッドにおけるスケーリング性能を重視して設計された高効率マイクロアーキテクチャ (Atom/Small)。  
前世代の *Tremontアーキテクチャ* から大きく進化した点には、L1命令キャッシュの倍増 (32KiB -\> 64KiB)、実行ポート数が 8ポートから 17ポートに、AVX2 と AVX-VNNI命令への対応等が挙げられる。ロード/ストアユニットも倍に増やされており、L1データキャッシュの帯域も向上していると思われる。  
デコーダーが 3-wide デコードのクラスタを 2基持つという構成は *Tremont* から引き継がれている。  従来の方法でデコード幅を増やそうとすると、必要なリソース量が指数関数的に増えて効率が悪化するが、デコーダーのクラスタリングを用いることでリソース量は線形となり、効率の低下を防げる。[^decode-cluster]  
*Tremont* では。デコーダーのクラスタリングによって発生する負荷分散は分岐予測に依存していたため、分岐の存在しない長いコード (浮動小数点演算等) では片方のデコードクラスタしか使用できず、ボトルネックとなる可能性があった。  
[Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html) では、次世代の Atomプロセッサではそうしたケースを検出し、ボトルネックを軽減するハードウェアを含む予定だとしている。  
その次世代の Atomプロセッサが *Gracemont* を指し、かつ実際にそうしたハードウェアが導入されているならば、クラスタリングによるリソース量の節約という恩恵を受けながら、性能と電力効率がさらに向上していることとなる。  

[^decode-cluster]:  [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)

### Golden Cove (Performance-core) {#golden_cove}
*Golden Cove* はシングルスレッド性能、低レイテンシを重視して設計された高性能なマイクロアーキテクチャ (Core/Big) となる。  
*Golden Cove* は、キャッシュ構成は前世代の *Willow Cove* をほとんど踏襲したものとなっているが、フロントエンド部、アウト・オブ・オーダーエンジン、実行ユニット数を大幅に増強している。  
特にデコード幅は従来の 4-wide から 6-wide に増やされ、命令のフェッチサイズも 16B から 32B となった。  
ただ、*Skylake, Sunny Cove* では 1x Complex decoder + 4x Simple decoder という構成であったため (*Willow Cove* に構成は公開されていない)、*Golden Cove* で増やされたのがどれで、具体的な構成については不明だが、従来から増やされたことは確かなのだろう。  
6-wide という点だけ見れば、*Tremont, Gracemont* と同じデコード幅だが、*Golden Cove* はデコーダーのクラスタリングを採用していない。クラスタリングの利点から考えれば、*Golden Cove* のデコーダーにはかなりのリソースが割かれていることになる。  
Intel が示す *Alder Lake* の CGイメージでは、*1x Golden Cove == 4x Gracemont* というサイズで描かれているが、これは *Gracemont* が小さいというよりは、*Golden Cove* が巨大なコアであることを示しているような気がしなくもない。  
*Golden Cove* は AVX512 に対応している。スライド内の CGイメージから、2x FMA (256-bit?) + 1x FMA (512-bit) という構成で、*Skylake (Server), Sunny Cove (Server)* と同様に、2x FMA (256-bit) を束ねるポートフージョンと FMA (512-bit) 1ポートそれぞれで AVX-512 が処理可能になっていると思われる。  
また、*Golden Cove* では新たな AVX-512系命令、`AVX512_FP16` に対応している。  
{{< link >}} [Intel Sapphire Rapids は AVX512_FP16 をサポート | Coelacanth's Dream](/posts/2021/01/11/intel-spr-avx512_fp16/) {{< /link >}}

*Sunny Cove (Client)* ではポートフージョンには対応せず、FMA (512-bit) で AVX-512 を処理する形を採っていた。  
*Golden Cove (Client)* も同様の構成を採るかどうかは気になる所だが、それ以前に *Golden Cove (Client)* を採用する *Alder Lake* では AVX-512 が有効化されないことが複数のメディア、ライターから語られている。[^pcwatch][^anand-ian_cutress]  
Intel のハイブリッドアーキテクチャでは、*Lakefield* の例から Core (Big) / Atom (Small) 両方が有効な場合はどちらも対応している命令のみが有効化されることが分かっていたが、[AnandTech](https://www.anandtech.com) の [Dr. Ian Cutress](https://www.anandtech.com/Author/140) 氏によれば、Atom (Small) 側を無効化しても、Core (Big) 側のみが対応する AVX-512 命令等は無効化されたままだとしている。  

 * [Instruction Sets: Alder Lake Dumps AVX-512 in a BIG Way - Intel Architecture Day 2021: Alder Lake, Golden Cove, and Gracemont Detailed](https://www.anandtech.com/show/16881/a-deep-dive-into-intels-alder-lake-microarchitectures/5)

以前、Linux Kernel へのあるパッチ内で Intel のソフトウェアエンジニアより、*Alder Lake* では UEFI/BIOS、またはカーネルパラメーターから一方の CPUタイプを無効化するオプションが提供されることが語られていた。  
{{< link >}} [Alder Lake では BIOS から一方の CPUタイプを無効化可能 | Coelacanth's Dream](/posts/2021/04/29/adl-cpu-type/) {{< /link >}}
その時は Atom (Small) 側の無効化により、AVX-512 命令の対応が有効化されるのではないかと自分は考えたが、その推測は外れてしまった。  
技術的な問題か、それとも性能に関係するマーケティング的な問題か。理由が明かされる望みは薄いが、多くの人が惜しいと思う部分になるのではないだろうか。  
最近リリースされた Intel のクライアント向け新型プロセッサでは、*Ice Lake, Tiger Lake, Rocket Lake* と AVX-512 命令に対応した CPU が続き、同時にそのことをアピールしてきた。中でも `AVX512_VNNI` 命令を活用した高速な推論処理、マーケティング的には Intel DL (Deep Learning) Boost と呼ばれ、目玉機能として扱われてきた。  
その流れが *Alder Lake* で途切れてしまうことになるが、*Alder Lake* に採用される *Golden Cove* と *Gracemont* は新たな `AVX_VNNI (128/256-bit)` 命令に対応しており、これが Intel DL Boost として含まれることになると思われる。  
ベクタ幅が異なるため、性能に影響が出ると考えられるが、AVX-512 に対応したクライアント向けプロセッサは最大 8-Core (*Tiger Lake-H, Rocket Lake-S*) だったのに対し、*Alder Lake* では両方のアーキテクチャが `AVX_VNNI (128/256-bit)` に対応しているため、デスクトップ向けでは最大 16-Core で処理可能となる。  
それでもやはりスケーリング性能、スケジューリング、Atom (Small) 側の最大動作周波数等が性能に悪影響をもたらす懸念は出てくる。  
その影響がどれ程のものになるかは、実際に *Alder Lake* が登場してくるまでは分からない。  

[^pcwatch]: [Intel次期CPU「Alder Lake」はWindows 11に最適化されたスレッド割り当て機能を搭載 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1344898.html)
[^anand-ian_cutress]: [Instruction Sets: Alder Lake Dumps AVX-512 in a BIG Way - Intel Architecture Day 2021: Alder Lake, Golden Cove, and Gracemont Detailed](https://www.anandtech.com/show/16881/a-deep-dive-into-intels-alder-lake-microarchitectures/5)

| uArch     | Golden Cove | Willow Cove   | Sunny Cove | Gracemont | Tremont   |
| :--       | :--:          | :--:          | :--:      | :--:      | :--: |
| L1 Data size   | 48 KiB<br>/12-way | 48 KiB<br>/12-way | 48 KiB<br>/12-way | 32 KiB<br>/8-way | 32 KiB<br>/8-way 
| L1D BW<br>(L: Load, S: Store) | 256B?<br>(L: 3x32B? or 2x64B?)<br>(S: 2x64B?)  | ? | 192B<br>(L: 2x64B)<br>(S: 1x64B or 2x32B) | 128B???<br>(L: 2x32B? + S: 2x32B?) | 64B???<br>(L: 1x32B + S: 1x32B)<br>(L: 2x32B or S: 2x32B)
| L1 Inst. size   | 32 KiB<br>/8-way  | 32 KiB<br>/8-way | 32 KiB<br>/8-way | 64 KiB<br>/8-way | 32 KiB<br>/8-way
| L1I BW | ? | ? | ? | 32B?<br>(2x 16B) | 32B<br>(2x 16B) |
| L2 size       | 1.25 MiB/10-way (Client)<br>2 MiB (Data Center) | 1.25 MiB<br>/20-way | 512 KiB<br>/8-way | 2 MiB<br>/16-way<br>(per Module) | 1.5-4.5 MiB<br>/12-way<br>(per Cluster/Tile) |
| L2 BW Peak/Sustained | ? | ? | 64B/48B | 64B/? | 
| L3 size       | 12 MiB<br>/12-way<br>(3 MiB per Core) | 12 MiB<br>/12-way<br>(3 MiB per Core) | 2 MiB<br>/12-way  |12 MiB<br>/12-way<br>(3 MiB per Cluster/Tile) | 4 MiB<br>/16-way

 * 表参考リンク
    * [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)
    * Tremont - [Tremont: A Wider Front End and Caches - Intel's new Atom Microarchitecture: The Tremont Core in Lakefield](https://www.anandtech.com/show/15009/intels-new-atom-microarchitecture-the-tremont-core/2)
    * Willow Cove/Sunny Cove - [Cache Architecture: The Effect of Increasing L2 and L3 - Intel’s Tiger Lake 11th Gen Core i7-1185G7 Review and Deep Dive: Baskin’ for the Exotic](https://www.anandtech.com/show/16084/intel-tiger-lake-review-deep-dive-core-11th-gen/4)
    * [Config_tools: Update board xml for acrn 2.5 · projectacrn/acrn-hypervisor@918ec7a](https://github.com/projectacrn/acrn-hypervisor/commit/918ec7aaf316b49aba5f58ffed6513f84cc1f96f)
        * [acrn-hypervisor/adl-rvp.xml at 918ec7aaf316b49aba5f58ffed6513f84cc1f96f · projectacrn/acrn-hypervisor](https://github.com/projectacrn/acrn-hypervisor/blob/918ec7aaf316b49aba5f58ffed6513f84cc1f96f/misc/config_tools/data/adl-rvp/adl-rvp.xml)
