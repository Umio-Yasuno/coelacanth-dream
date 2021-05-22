---
title: "Alder Lake (Golden Cove + Gracemont) のキャッシュ詳細"
date: 2021-05-22T01:08:09+09:00
draft: false
tags: [ "Alder_Lake", "Sapphire_Rapids" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

組み込みデバイス向けのハイパーバイザー [ACRN Hypervisor](https://github.com/projectacrn/acrn-hypervisor) へのプルリクエストの中で *Alder Lake-P* の構成ファイルが投稿された。投稿したのは Intel の [Kunhui-Li](https://github.com/Kunhui-Li) 氏。  
ACRN Hypervisor は Intel が推進するオープンソースプロジェクトの一つであり、Intel はメンバーとして参加、自社プロセッサ用のコードを提供している。[^intel-acrn]  
そして構成ファイルには CPUキャッシュの詳細情報が含まれており、既にキャッシュサイズはベンチマーク結果から知られていたが、今回キャッシュの way数も公開されたことになる。  
{{< link >}} [Intel Alder Lake-S のベンチマーク結果から読むキャッシュ構成　【追記 2020-10-07】16-Core/24-Thread? | Coelacanth's Dream](/posts/2020/10/06/intel-adls-benchmark-cache/) {{< /link >}}

 * [Config_tools: Update board xml for acrn 2.5 · projectacrn/acrn-hypervisor@918ec7a](https://github.com/projectacrn/acrn-hypervisor/commit/918ec7aaf316b49aba5f58ffed6513f84cc1f96f)
    * [acrn-hypervisor/adl-rvp.xml at 918ec7aaf316b49aba5f58ffed6513f84cc1f96f · projectacrn/acrn-hypervisor](https://github.com/projectacrn/acrn-hypervisor/blob/918ec7aaf316b49aba5f58ffed6513f84cc1f96f/misc/config_tools/data/adl-rvp/adl-rvp.xml)

[^intel-acrn]: [Intelが推進するEdgeの仮想化、ACRNとは？ | Think IT（シンクイット）](https://thinkit.co.jp/article/13695)

{{< pindex >}}
 * [構成ファイルからキャッシュ情報を読み取る](#cache)
 * [Golden Cove, Gramont](#table)
{{< /pindex >}}

## 構成ファイルからキャッシュ情報を読み取る {#cache}

まず構成ファイルからキャッシュ情報を読み取る方法を書くと、構成ファイルは基本以下のように記述されている。  
`cache level` とその値はそのまま意と思われ、`<processor>..</procssor>` にあるのはそのキャッシュを持つ CPUコアの ID と思われる。  
そして `type` だが、これは CPUID からキャッシュパラメーターを読み取るのに使われる定義と同じと思われる。  
[Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/content/www/us/en/develop/download/intel-architecture-instruction-set-extensions-programming-reference.html) の Table 1-3 に記載されているが、 Leaf (EAXレジスタ) に 0x4、Sub-leaf (ECXレジスタ) にキャッシュレベルに対応した値をセットして CPUID命令を実行すると得られる 32-bit の値では、Bit 4-0 がキャッシュタイプを示すフィールドとなっており、1 は Data Cache、2 は Instruction Cache、3 は Unified Cache、0 は Null でその他は予約領域。  
これを踏まえると、以下の上側は L1 Data Cache の構成を、下は L1 Instruction Cache の構成を示していることがわかる。早速ツールを自作する時に得た CPUID の知識が役立った。  

また、ここにおける *Alder Lake-P* は 2x Core (Golden Cove) + 8x Atom (Gracemont) の構成となっており、Core ID: 0x0, 0x4 に *Golden Cove* 、Core ID: 0x8-0xF に *Gracemont* が対応している。  

 > 		  <caches>
 > 		    <cache level="1" id="0x0" type="1">
 > 		      <cache_size>49152</cache_size>
 > 		      <line_size>64</line_size>
 > 		      <ways>12</ways>
 > 		      <sets>64</sets>
 > 		      <partitions>1</partitions>
 > 		      <self_initializing>1</self_initializing>
 > 		      <fully_associative>0</fully_associative>
 > 		      <write_back_invalidate>0</write_back_invalidate>
 > 		      <cache_inclusiveness>0</cache_inclusiveness>
 > 		      <complex_cache_indexing>0</complex_cache_indexing>
 > 		      <processors>
 > 		        <processor>0x0</processor>
 > 		      </processors>
 > 		    </cache>
 > 		    <cache level="1" id="0x0" type="2">
 > 		      <cache_size>32768</cache_size>
 > 		      <line_size>64</line_size>
 > 		      <ways>8</ways>
 > 		      <sets>64</sets>
 > 		      <partitions>1</partitions>
 > 		      <self_initializing>1</self_initializing>
 > 		      <fully_associative>0</fully_associative>
 > 		      <write_back_invalidate>0</write_back_invalidate>
 > 		      <cache_inclusiveness>0</cache_inclusiveness>
 > 		      <complex_cache_indexing>0</complex_cache_indexing>
 > 		      <processors>
 > 		        <processor>0x0</processor>
 > 		      </processors>
 > 		    </cache>
 >
 > {{< quote >}} [acrn-hypervisor/adl-rvp.xml at 918ec7aaf316b49aba5f58ffed6513f84cc1f96f · projectacrn/acrn-hypervisor](https://github.com/projectacrn/acrn-hypervisor/blob/918ec7aaf316b49aba5f58ffed6513f84cc1f96f/misc/config_tools/data/adl-rvp/adl-rvp.xml) {{< /quote >}}

## Golden Cove, Gramont {#table}

現世代、*Golden Cove* と *Gracemont* からすれば前世代となる *Willow Cove* 、*Tremont* と一緒にキャッシュ構成をまとめたのが以下の表となる。  


| uArch     | Golden Cove   | Willow Cove   | Sunny Cove | Gracemont | Tremont   |
| :--       | :--:          | :--:          | :--:      | :--:      | :--: |
| L1 Data   | 48 KiB<br>/12-way | 48 KiB<br>/12-way | 48 KiB<br>/12-way | 32 KiB<br>/8-way | 32 KiB<br>/8-way 
| L1 Inst   | 32 KiB<br>/8-way  | 32 KiB<br>/12-way | 32 KiB<br>/8-way | >64 KiB<br>/8-way | 32 KiB<br>/8-way
| L2        | 1.25 MiB<br>/10-way | 1.25 MiB<br>/20-way | 512 KiB<br>/8-way | 2 MiB<br>/16-way<br>(per Module) | 1.5-4.5 MiB<br>/12-way<br>(per Cluster/Tile) |
| L3        | 12 MiB<br>/12-way<br>(3 MiB per Core) | 12 MiB<br>/12-way<br>(3 MiB per Core) | 2 MiB<br>/12-way  |12 MiB<br>/12-way<br>(3 MiB per Cluster/Tile) | 4 MiB<br>/16-way

注目されるのは *Golden Cove* のキャッシュ構成だが、L1D/I と L2 のキャッシュサイズは *Willow Cove* と同じだが、L1I と L2 の way数を減らした興味深いものになっている。特に L2キャッシュは 半分の 10-way に減らされている。  
[ACRN Hypervisor](https://github.com/projectacrn/acrn-hypervisor) には *Tiger Lake* の構成ファイルも存在するためそちらも確認したが、やはり *Tiger Lake/Willow Cove* は L2 1.25 MiB/20-way の構成だった。[^tgl-rvp]  

[^tgl-rvp]: [acrn-hypervisor/tgl-rvp.xml at 918ec7aaf316b49aba5f58ffed6513f84cc1f96f · projectacrn/acrn-hypervisor](https://github.com/projectacrn/acrn-hypervisor/blob/918ec7aaf316b49aba5f58ffed6513f84cc1f96f/misc/config_tools/data/tgl-rvp/tgl-rvp.xml#L744)

 > 		    <cache level="2" id="0x0" type="3">
 > 		      <cache_size>1310720</cache_size>
 > 		      <line_size>64</line_size>
 > 		      <ways>10</ways>
 > 		      <sets>2048</sets>
 > 		      <partitions>1</partitions>
 > 		      <self_initializing>1</self_initializing>
 > 		      <fully_associative>0</fully_associative>
 > 		      <write_back_invalidate>0</write_back_invalidate>
 > 		      <cache_inclusiveness>0</cache_inclusiveness>
 > 		      <complex_cache_indexing>0</complex_cache_indexing>
 > 		      <processors>
 > 		        <processor>0x0</processor>
 > 		      </processors>
 > 		    </cache>
 >
 > {{< quote >}} [acrn-hypervisor/adl-rvp.xml at 918ec7aaf316b49aba5f58ffed6513f84cc1f96f · projectacrn/acrn-hypervisor](https://github.com/projectacrn/acrn-hypervisor/blob/918ec7aaf316b49aba5f58ffed6513f84cc1f96f/misc/config_tools/data/adl-rvp/adl-rvp.xml) {{< /quote >}}

CPUキャッシュにおいて、メモリアドレスに対応するキャッシュラインを複数設ける (n-way) ことは、キャッシュラインの競合 (スラッシング) が発生する確率を減らし、性能低下を防ぐ役割がある。way数を減らしたことは、逆にスラッシングが発生しやすくなるように思えるが、個人的な推測を書けば、 *Golden Cove* はサーバー向けプロセッサ *Sapphire Rapids* にも採用することがつい先日明らかにされており、way数を減らしたのはサーバー向けとしてより広いメモリを扱うことを意識した改良なのかもしれない。  
{{< link >}} [Intel Sapphire Rapids は Golden Coveアーキテクチャ | Coelacanth's Dream](/posts/2021/05/20/intel-spr-golden_cove/) {{< /link >}}
むしろ、*Willow Cove* は *Golden Cove* や *Ice Lake* に採用されている *Sunny Cove (L2 512 KiB/8-way)* 等のマイクロアーキテクチャとは違い、サーバー向けプロセッサには採用されないことが決まっていたがためにあれだけ L2キャッシュの way数を増やすことができたのかもしれない。  

また *Gracemont* では L1I キャッシュサイズが *Tremont* の 32 KiB/8-way から大幅に増量され、64 KiB/8-way となった。Core/Big側の *Golden Cove* も 32 KiB/8-way であり、*Gracemont* の L1I キャッシュは異様とも言える大きさだ。  
これについては、*Tremont* で大幅に拡張されたデコーダーをより活用し、性能を向上させる狙いがあるのかもしれない。  

*Gracemont* の L2キャッシュについては構成ファイルから得られる情報をそのまま記載したが、*Tremont* のように、プロセッサによって一定の範囲でサイズが選択可能となっている可能性もある。  
*Tremont* では、マイクロサーバー向けの *Snow Ridge* が 4.5 MiB、他 *Lakefield* 、*Elkhart Lake* 、*Jasper Lake* が 1.5 MiB の構成を採用している。  


{{< ref >}}
 * [Tremont: A Wider Front End and Caches - Intel's new Atom Microarchitecture: The Tremont Core in Lakefield](https://www.anandtech.com/show/15009/intels-new-atom-microarchitecture-the-tremont-core/2)
 * [Cache Architecture: The Effect of Increasing L2 and L3 - Intel’s Tiger Lake 11th Gen Core i7-1185G7 Review and Deep Dive: Baskin’ for the Exotic](https://www.anandtech.com/show/16084/intel-tiger-lake-review-deep-dive-core-11th-gen/4)
{{< /ref >}}
