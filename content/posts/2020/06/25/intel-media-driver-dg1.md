---
title: "Intel、media-driver に DG1 へ向けたパッチを投稿　―― GPGPUに傾いた DG1 のキャッシュ設定"
date: 2020-06-25T10:11:44+09:00
draft: false
tags: [ "DG1", "Gen12" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
---

Intel は、動画のデコード/エンコード/ポストプロセッシングを Intel GPU で実行するための [media-driver](https://github.com/intel/media-driver) に *DG1* を部分的にサポートするパッチを投稿した。  
{{< link >}}[[Upstream] DG1 open source stage 1 · intel/media-driver@de48276](https://github.com/intel/media-driver/commit/de482769db4f94a99d672c82dd250c9ea484b52a){{< /link >}}

部分的なサポートということもあり、*DG1* サポートを有効にするビルドオプションはまだ無く、*DG1* がどの程度動画のデコード/エンコード/ポストプロセッシングをサポートするかは明らかにされていない。  

{{< ins >}}

*DG1* に向けたパッチ第2弾が投稿された。  
{{< link >}}[[Upstream] DG1 open source stage 2 · intel/media-driver@bcfaf0b](https://github.com/intel/media-driver/commit/bcfaf0b43e02d14b373995523c1475d07442c1f9?short_path=acd8702){{< /link >}}

動画のデコード/エンコード/ポストプロセッシングの概要が追加されたが、特別強化されている様ではなく、*Tiger Lake* と同等の機能となるようだ。  
{{< link >}} <https://github.com/intel/media-driver/commit/bcfaf0b43e02d14b373995523c1475d07442c1f9?short_path=acd8702#diff-acd87025799e24be8c7e5078baa32897> {{< /link >}}

{{< /ins >}}

しかし、追加されたファイル/コードにより、[前回](/posts/2020/06/20/intel-dg1-support-opencl-levelzero/)判明した *DG1* が持つ大きなキャッシュが内部でどのように割り振られてるかがわかった。  

 >       #define DG1_L3_CONFIG_NUM                      3
 >       static const L3ConfigRegisterValues DG1_L3_PLANE[] = {
 >          //L3Alloc     TCCNTL                 //  Rest   DC  RO   Z    Color  UTC    CB    Sum (in KB)
 >          { 0x00000200, 0x0,          0, 0 },  //  2048   0   0    0    0       0      0    2048
 >          { 0x40000000, 0x3E000010,   0, 0 },  //  1024   0   0    0    0       992   32    2048
 >          { 0x0080F800, 0x00000010,   0, 0 },  //  0    1024  992  0    0       0     32    2048
 >       };
 >
 > 引用元: <cite>[media-driver/media_interfaces_g12_dg1.h at de482769db4f94a99d672c82dd250c9ea484b52a · intel/media-driver](https://github.com/intel/media-driver/blob/de482769db4f94a99d672c82dd250c9ea484b52a/media_driver/agnostic/gen12_dg1/media_interfaces/media_interfaces_g12_dg1.h)</cite>

L3cacheバンクあたりの容量が確かに 2048KB と、前回判明した内容が自分の見間違い等ではなかったようで安心した。  

キャッシュ設定は 3種類と、統合GPUである *Ice Lake LP* の 9種類[^1]、*Tiger Lake* の 7種類[^2]に比べると少ない。  

[^1]: [media-driver/cm_rt_g11.h at a634e836f31469db58f4d082d8c0c9f89116acbb · intel/media-driver](https://github.com/intel/media-driver/blob/a634e836f31469db58f4d082d8c0c9f89116acbb/cmrtlib/agnostic/share/cm_rt_g11.h)
[^2]: [media-driver/cm_rt_g12_tgl.h at a634e836f31469db58f4d082d8c0c9f89116acbb · intel/media-driver](https://github.com/intel/media-driver/blob/a634e836f31469db58f4d082d8c0c9f89116acbb/cmrtlib/agnostic/share/cm_rt_g12_tgl.h)

`Rest /DC /RO /Z /Color /UTC /CB` がそれぞれ何を意味するかは、Intel が公開しているドキュメントを確認するに、  
`DC (Data Cluster)` はデータキャッシュとしてレジスタ割り付けやグローバルメモリアクセスに使われ、  
`RO (Read-Only)` は命令、状態(State)、定数、テキスチャを格納するキャッシュであり、`Rest` はそれら 2つを組み合わせたもの、  
`Z /Color` はグラフィクス処理に使われる深度 /色情報を格納するキャッシュであり、`UTC (Unified Tile Cache)` はそれらを組み合わせたもの、  
`CB` は `Command Buffer` の略であり、名の通りの働きをすると思われる。[^3]  
<!--  未確認、NVIDIA GPU の L1cacheみたく、`Rest` と `UTC` は組み合わせたキャッシュそれぞれの容量が可変であったりするのだろうか。   -->

[^3]: [Volume 7: Memory Cache - intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf)

その上で *DG1* のキャッシュ設定を見ると、  
1つ目と 3つ目は、データ/リードオンリーキャッシュに極端に割り振り、グラフィクス系キャッシュは省いた GPGPU向けの設定、  
2つ目は `Rest` と `UTC` に大体半分ずつ(1024KB) 割り振ったグラフィクス向けの設定と推察される。  
L3cacheバンクの設定に、シェーダーの入出力等に使われる `URB (Unified Return Buffer)` が無いが、L3cacheの外に備えられているのかもしれない。  
そうなると *DG1* は、各Sub-Slice内のローカルメモリと、L3cache 16MB以上にオンチップメモリを持つことになる。  

Intel *DG1* からはキャッシュ容量の大きさだけでなく、その設定からも GPGPU向けであるように感じられる。  
正確な要素は含んでいないが、GPUコアは *Tiger Lake GT2* と同規模であっても、大きいキャッシュメモリと VRAM(GDDR5かGDDR6) のPHY{{< comple >}}メモリバス幅は 96-bit だと言われている{{< /comple >}}を備えることを考えると、  
*DG1* のダイサイズは *Tiger Lake* とあまり変わらないのではないかと思う。最初から dGPUとしては実験的な感じで設計したのだろうか？  

キャッシュ設定やコストから、PCIeカードをソフトウェア開発者向けに絞って出荷するというのは納得できるが、  
オンボードで *Tiger Lake* と組み合わせたノートPCの構成を提供するというのは、意図が読みづらい。  
それもまたプレミアムな製品のみとなるのか、それとも性能向上の秘策があるのか、あるいはただシェアを伸ばしたいのか。  
一応、*DG1* の Device(PCI) ID は 3種確認できるため[^4]、製品も複数展開される可能性は存在する。  

[^4]: [media-driver/media_sysinfo_g12.cpp at de482769db4f94a99d672c82dd250c9ea484b52a · intel/media-driver](https://github.com/intel/media-driver/blob/de482769db4f94a99d672c82dd250c9ea484b52a/media_driver/linux/gen12/ddi/media_sysinfo_g12.cpp#L251)

{{< ref >}}

 * [GeForce vs ATI Radeon - アーキテクチャ解説で紐解くGPU戦争"夏の陣" (後編) (3) ROPユニットはどう進化したか | マイナビニュース](https://news.mynavi.jp/article/20080728-s_gpu02/3)

{{< /ref >}}
