---
title: "Intel、Alder Lake Developer Guide を公開"
date: 2021-10-16T12:58:53+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Intel は 2021/10/04 付でゲーム開発者向けに *Alder Lake* への性能最適化手法をまとめた *「Intel® Codename Alder Lake (ADL) Developer Guide」* を公開した。  

 * [Intel® Codename Alder Lake (ADL) Developer Guide](https://www.intel.com/content/www/us/en/developer/articles/guide/alder-lake-developer-guide.html?wapkw=alder%20lake)

*Alder Lake* は Big/Performance Core (Golden Cove/Core) と Small/Efficient Core (Gracemont/Atom) の異なる 2種類の CPUアーキテクチャを組み合わせたハイブリッドアーキテクチャを採り、最適化のためにはそれを意識したスレッド化が必要となる。  

また Intel は現地時間 2021/10/27-2021/10/28 に [Intel Innovation](https://www.intel.com/content/www/us/en/events/on-event-series.html) と題したイベントを予定しており、Alder Lake Developer Guide の公開はそれに合わせたものと思われる。  
イベントでは *「Performance Tuning Games for Alder Lake and Hybrid Architectures」* 、 *「Alchemist Graphics Architecture and Features for Game Developers」* というセッションが予定されている。  

 * [Game Development](https://www.intel.com/content/www/us/en/developer/topic-technology/gamedev/overview.html?wapkw=alder%20lake)

## 異なる 2種類の CPUアーキテクチャ {#diff-cpu-arch}

*Golden Cove* と *Gracemont* ではキャッシュ構成が異なり、*Golden Cove* は各コアが L2キャッシュ 1.25 MiB を持ち、L3キャッシュ/LLC (Last Level Cache) に接続されるが、*Gracemont* は 4コアで L2キャッシュ 2 MiB を共有し、モジュール/クラスターを構成する。  
*Golden Cove* は Intel Hyper-Threading が有効化されているため 2スレッドで L1キャッシュを共有することになるが、*Gracemont* では無効化されている、あるいはサポートされていないため 1コア/1スレッドが独自の L1キャッシュを持つといった違いもある。  
*Golden Cove* と *Gracemont* との間でキャッシュを共有する場合は L3/LLC でのコヒーレンスに制限される。

ハイブリッドアーキテクチャでは対応命令/ISAレベルでは対称的となり、そのため *Golden Cove* の一部機能は制限される。  
*Golden Cove* では AVX512系命令に対応しているが、*Gracemont* は AVX2 までのサポートであるため、*Gracemont/E-Core* が有効化されている場合、AVX512 は無効化される。  
同時に *Gracemont/E-Core* を BIOS/UEFI から無効化する機能が存在するが、それをユーザーが切り替え可能とするかは OEM に委ねられるとしている。  
{{< link >}} [Alder Lake では BIOS から一方の CPUタイプを無効化可能 | Coelacanth's Dream](/posts/2021/04/29/adl-cpu-type/) {{< /link >}}
ただ、AVX512 が無効化されている理由に *Gracemont/E-Core* を挙げているが、*Gracemont/E-Core* をすべて無効化すれば AVX512 が有効化されるかどうかについては明言されていない。  
Intel Architecture Day 2021、Hot Chip 33 で語られたところでは、*Gracemont/E-Core* を無効化しても AVX-512 等は有効化されず、また AVX-512 ユニットを *Alder Lake* の *Golden Cove/P-Core* には物理的に搭載していないという。  

 * [Alder Lake DeepDive - HotChipsで垣間見たIntel「Alder Lake」の細部 | マイナビニュース](https://news.mynavi.jp/article/20210825-1955663/)
 * [Instruction Sets: Alder Lake Dumps AVX-512 in a BIG Way - Intel Architecture Day 2021: Alder Lake, Golden Cove, and Gracemont Detailed](https://www.anandtech.com/show/16881/a-deep-dive-into-intels-alder-lake-microarchitectures/5)

SKU (Stock-Keeping Unit) の節で、モバイル向け SKU ではすべてで *Gracemont/E-Core* が有効化されているが、デスクトップ向けでは *Gracemont/E-Core* を無効化し、*Golden Cove/P-Core* のみを有効化した SKU が存在するとしている。  
仮に *Gracemont/E-Core* の無効化で AVX512 のサポート有無が変わると SKU 間で大きな性能差が生まれることになるため、そうした点からも *Alder Lake* では AVX512 が存在しない可能性が高い。  

### E-Core の活用 {#e-core}

*Alder Lake* の性能を最大限引き出すには *Gracemont/E-Core* の活用が必要となる。  
しかし単一のスレッドプールを *Gracemont/E-Core* を含めたトポロジに割り当てた場合、スレッドの同期部分で *Gracemont/E-Core* が足を引っ張り全体の性能が低下する恐れや、スレッドの切り替えによるオーバーヘッドの発生が予想される。  
そこで高性能を必要とする重要度の高いスレッドプールと、非同期的な、性能に影響するクリティカルパスの外にある重要度の低いスレッドプールを用意することが推奨されている。  
それによって ITD (Intel Thread Director) と OSスケジューラーはより適切にスレッドを *Golden Cove/P-Core* 、*Gracemont/E-Core* に割り当てることができる。  
重要度の高いスレッドプールには *Golden Cove/P-Core* のみを割り当てるよう、アプリケーション、ゲーム側で設定できる。  
*Gracemont/E-Core* に向いた処理には、シェーダーコンパイル、音声処理、アセットストリーミング、展開処理等が挙げられている。  

モバイル向けでは最大で *Golden Cove/P-Core* 6コア + *Gracemont/E-Core* 8コアという構成で、SKU によっては 2コア+8コアという *Gracemont/E-Core* の比率が大きい構成が用意されているため、こうした *Alder Lake* 向けの最適化はモバイル向けでは特に重要となる。  

| uArch     | Golden Cove | Willow Cove   | Sunny Cove | Gracemont | Tremont   |
| :--       | :--:          | :--:          | :--:      | :--:      | :--: |
| L1 Data size   | 48 KiB<br>/12-way | 48 KiB<br>/12-way | 48 KiB<br>/12-way | 32 KiB<br>/8-way | 32 KiB<br>/8-way 
| L1D BW<br>(L: Load, S: Store) | 256B?<br>(L: 3x32B? or 2x64B?)<br>(S: 2x64B?)  | ? | 192B<br>(L: 2x64B)<br>(S: 1x64B or 2x32B) | 128B???<br>(L: 2x32B? + S: 2x32B?) | 64B???<br>(L: 1x32B + S: 1x32B)<br>(L: 2x32B or S: 2x32B)
| L1 Inst. size   | 32 KiB<br>/8-way  | 32 KiB<br>/8-way | 32 KiB<br>/8-way | 64 KiB<br>/8-way | 32 KiB<br>/8-way
| L1I BW | ? | ? | ? | 32B?<br>(2x 16B) | 32B<br>(2x 16B) |
| L2 size       | 1.25 MiB/10-way (Client)<br>2 MiB (Data Center) | 1.25 MiB<br>/20-way | 512 KiB<br>/8-way | 2 MiB<br>/16-way<br>(per Module) | 1.5-4.5 MiB<br>/12-way<br>(per Cluster/Module) |
| L2 BW Peak/Sustained | ? | ? | 64B/48B | 64B/? | 
| L3 size       | 12 MiB<br>/12-way<br>(3 MiB per Core) | 12 MiB<br>/12-way<br>(3 MiB per Core) | 2 MiB<br>/12-way  |12 MiB<br>/12-way<br>(3 MiB per Cluster/Module) | 4 MiB<br>/16-way

 * 表参考リンク
    * [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)
    * Tremont - [Tremont: A Wider Front End and Caches - Intel's new Atom Microarchitecture: The Tremont Core in Lakefield](https://www.anandtech.com/show/15009/intels-new-atom-microarchitecture-the-tremont-core/2)
    * Willow Cove/Sunny Cove - [Cache Architecture: The Effect of Increasing L2 and L3 - Intel’s Tiger Lake 11th Gen Core i7-1185G7 Review and Deep Dive: Baskin’ for the Exotic](https://www.anandtech.com/show/16084/intel-tiger-lake-review-deep-dive-core-11th-gen/4)
    * [Config_tools: Update board xml for acrn 2.5 · projectacrn/acrn-hypervisor@918ec7a](https://github.com/projectacrn/acrn-hypervisor/commit/918ec7aaf316b49aba5f58ffed6513f84cc1f96f)
        * [acrn-hypervisor/adl-rvp.xml at 918ec7aaf316b49aba5f58ffed6513f84cc1f96f · projectacrn/acrn-hypervisor](https://github.com/projectacrn/acrn-hypervisor/blob/918ec7aaf316b49aba5f58ffed6513f84cc1f96f/misc/config_tools/data/adl-rvp/adl-rvp.xml)

{{< ref >}}
 * [Intel® Codename Alder Lake (ADL) Developer Guide](https://www.intel.com/content/www/us/en/developer/articles/guide/alder-lake-developer-guide.html?wapkw=alder%20lake)
 * [GameTechDev/HybridDetect: Heterogeneous & Homogeneous CPU Detect for Intel Processors](https://github.com/GameTechDev/HybridDetect)
 * [Alder Lake DeepDive - HotChipsで垣間見たIntel「Alder Lake」の細部 | マイナビニュース](https://news.mynavi.jp/article/20210825-1955663/)
 * [Instruction Sets: Alder Lake Dumps AVX-512 in a BIG Way - Intel Architecture Day 2021: Alder Lake, Golden Cove, and Gracemont Detailed](https://www.anandtech.com/show/16881/a-deep-dive-into-intels-alder-lake-microarchitectures/5)
{{< /ref >}}
