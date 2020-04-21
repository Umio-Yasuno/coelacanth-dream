---
title: "Intel Atom Tremont関連のコードネームを整理する"
date: 2020-04-20T14:17:24+09:00
draft: false
tags: [ "Tremont", "Gen11" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

Intelの次世代低消費電力 x86アーキテクチャ、*Tremont* ベースのコアを搭載するプロセッサは複数存在し、それぞれにコードネームが与えられている。  
それ故に複雑なところがあるため、今回はLinux Kernel、Mesa3D、Coreboot等のOSSから情報を集め、Intelが公開している情報を元にしてそれぞれの特徴を整理してみた。  

## インデックス

 * [Tremontアーキテクチャ](#tremont-arch)
 * [Jacobsville (Tremont-D)](#tremont-d)
 * [Elkhart Lake (Tremont)](#tremont)
 * [Jasper Lake (Tremont-L)](#tremont-l)
  * [Elkhart Lake と Jasper Lake の共通点と違い](#elkhart-jasper-diff)
 * [Lakefield](#lakefield)

## Tremont アーキテクチャ {#tremont-arch}
Intelの次世代低消費電力 x86アーキテクチャ。  
2x 3-way x86デコーダに、実行ポートを10ポートも持つ、これまでのAtomアーキテクチャとはかけ離れた厚い構成となっている。  

アーキテクチャの詳細は、ここで自分が無理に解説するより、テクニカルライター 大原雄介氏の記事を紹介するのが一番だと考えられるため、その様にすることとした。  

参考: [ASCII.jp：AtomベースのSmall CoreがTremontと判明　インテル CPUロードマップ (2/4)](https://ascii.jp/elem/000/001/981/1981403/2/)

## Jacobsville (Tremont-D) {#tremont-d}
*Tremont* コアを採用したマイクロサーバ向けプロセッサ。[^2][^3]  
マイクロサーバということから、10nmで製造され最大24コアの無線基地局向けSoC **Atom P5900** のベースとなるプロセッサを指すと思われる。  
[^2]: [x86/cpu: Add Atom Tremont (Jacobsville) · torvalds/linux@00ae831](https://github.com/torvalds/linux/commit/00ae831dfe4474ef6029558f5eb3ef0332d80043)
[^3]: [x86/intel: Aggregate microserver naming · torvalds/linux@5ebb34e](https://github.com/torvalds/linux/commit/5ebb34edbefa8ea6a7e109179d5fc7b3529dbeba#diff-9f27b735203b802ba0162ce1099eea33)

*Snow Ridge* という名前もあるが、そちらはI/Oといったアンコア部も含めたSoCもしくはプラットフォームを指すコードネームである、はず、たぶん。  
{{< link >}}[Products formerly Snow Ridge](https://ark.intel.com/content/www/us/en/ark/products/codename/87586/snow-ridge.html?wapkw=snow%20ridge){{< /link >}}
コードネームのややこしい点は、時にそれがCPU、GPU、SoC、プラットフォームのどれを指すのかはっきりしないところだ。  

*Jacobsville* は 4 Tremontコアと L2cache 4.5MBをまとめた *Core Tile* を6基持つ構成となっている。  

| Tremont-D | P5962B[^4] |
| :--- | :---: |
| Core Title | 6 |
| Core/Thread (per Tile) | 24/24 (4) |
| L2cache (per Tile) | 27 MB (4.5 MB) |
| Memory Channel | 2 |
| Memory Speed | 2933 MHz |
| PCIe Gen3 | 16 Lane |

[^4]: [Intel Atom® Processor P5962B (27M Cache, 2.20 GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/202682/intel-atom-processor-p5962b-27m-cache-2-20-ghz.html)

## Elkhart Lake (Tremont) {#tremont}
まだ仕様がはっきりとしていないが、現状判明している搭載製品から組み込み向けとされるプロセッサ。[^5]  
またその製品から、メモリはDDR4 1chである可能性がある。  

[^5]: <ftp://data.aaeon.com.tw/DOWNLOAD/Brochure/2020_Network_Appliances_brochure_AAEON.pdf><br>&emsp;&emsp;<ftp://data.aaeon.com.tw/DOWNLOAD/Brochure/2020_Network_Appliances_brochure_AAEON.pdf#page=7><br>&emsp;&emsp;<ftp://data.aaeon.com.tw/DOWNLOAD/Brochure/2020_Network_Appliances_brochure_AAEON.pdf#page=14>

それ以外に判明している仕様はGPUくらいで、GPUアーキテクチャは *Ice Lake* と同じ[Gen11](/tags/gen11)、EU(Execution Unit)数は32基、L3cacheは4バンク、1280KB。[^7]  
*Ice Lake* と同じ世代ではあっても機能面で微妙に違いが存在し、[intel/media-driver](https://github.com/intel/media-driver)によると、一部コーデックにてシェーダー(EU)を用いたエンコードがサポートされない。[^8]  
他にも、デノイズやシャーピング、HDR10トーンマッピングといった映像処理のサポートも削られている。  

[^7]: [media-driver/media_sysinfo_g11.cpp at intel-media-20.1.1 · intel/media-driver](https://github.com/intel/media-driver/blob/intel-media-20.1.1/media_driver/linux/gen11/ddi/media_sysinfo_g11.cpp#L322)
[^8]: [update readme for TGL/EHL/JSL · intel/media-driver@d0cbf53](https://github.com/intel/media-driver/commit/d0cbf53cd4dc23f1ef99b4b2e9bfab74172d9c9d)

PCHのコードネームは *Mule Creek Canyon PCH* と、ちょっと凝ったものになっている。[^15]

[^15]: [drm/i915/ehl: Introduce Mule Creek Canyon PCH · torvalds/linux@c6f7acb](https://github.com/torvalds/linux/commit/c6f7acb80abf5f73be4ee08541e3393a0146b15e)

## Jasper Lake (Tremont-L) {#tremont-l}
*Tremont* コアを採用したモバイル向けプロセッサ。[^1]

[^1]: [x86/cpu: Add Jasper Lake to Intel family · torvalds/linux@b2d32af](https://github.com/torvalds/linux/commit/b2d32af0bff402b4c1fce28311759dd1f6af058a)

CPUコア数に関しては若干情報があるのだが、正直怪しいところがある。  
まず[Coreboot](https://github.com/coreboot/coreboot)にある *Jasper Lake* のリファレンスボード、**jasperlake_rvp** の `Kconfig` は `MAX_CPUS` の値が `8` となっている。[^9]このままだと *Jasper Lake* の最大 8-Thread、と思われるが、  
*Jasper Lake* を搭載するChromebookのリファレンスボード、コードネーム **dedede** では `MAX_CPUS` を `4` としている。[^10]  
どちらを信用するかで言えば、**jasperlake_rvp** には **icelake_rvp** を元にして名前を置き換えた、という経緯があるため、[^11]**dedede** に分がある。*Ice Lake (Client)* は最大 8-Threadであり、**jasperlake_rvp** はその値をそのまま持ってきた結果とも考えられる。  
そういうことで、*Jasper Lake* は最大 4-Threadではないかと推測するが、  
それはあくまでも **dedede** で想定しているスレッド数であり、より多いコア/スレッド数を持つ *Jasper Lake* が出てくる可能性もある。  

[^9]: <https://github.com/coreboot/coreboot/blob/630aa4b3db1b7fa459380ec52328d632b53b22de/src/mainboard/intel/jasperlake_rvp/Kconfig#L34>
[^10]: <https://github.com/coreboot/coreboot/blob/aa56c11b1911fa49e53a145926b00670f9939f27/src/mainboard/google/dedede/Kconfig#L59>
[^11]: [mb/intel/jasperlake_rvp: Add initial mainboard code · coreboot/coreboot@630aa4b](https://github.com/coreboot/coreboot/commit/630aa4b3db1b7fa459380ec52328d632b53b22de)

*Jasper Lake* はGPUに関連するソフトウェア、[intel/media-driver](https://github.com/intel/media-driver) や [Mesa3D](https://gitlab.freedesktop.org/mesa/mesa) 等では *Elkhart Lake* として扱われており、両者の間にGPUの仕様違いは規模以外にないものと思われる。[^12][^13]  

[^12]: [[Encode] Add some device IDs for JSL · intel/media-driver@4b5a279](https://github.com/intel/media-driver/commit/4b5a279dae45f36e7bc42bb4ac662591567b5c2e#diff-56a1f17349b8bf63003aa4674344637b)
[^13]: [intel: Add device info for 1x4x6 Jasper Lake (11fdd5f5) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/11fdd5f52c3db070f33f7ef82d41acf14b1a2670)

メモリの仕様は、{{< comple >}}またも[Coreboot](https://github.com/coreboot/coreboot)のコードからだが{{< /comple >}}LP/DDR4 2ch(128-bit)だと読める。[^6]  

[^6]: <https://github.com/coreboot/coreboot/blob/512b77abb582e6c2566d3873b273dd32731e7bae/src/soc/intel/jasperlake/include/soc/meminit.h#L33>

PCHのコードネームは *Jasper Lake PCH* と、こちらはシンプル。[^14]  

モバイル向けとのことから、 *Jasper Lake* はここであげるプロセッサの中では今後最も見ることになるのはないかと思う。次点で *Lakefield* か。  

### Elkhart Lake と Jasper Lake の共通点と違い {#elkhart-jasper-diff}
*Elkhart Lake* と *Jasper Lake* は非常に似ている。  
GPUの機能は同じで、PCHもコードネームこそ違うが、中身としては映像出力1ポートのマッピングが微妙に異なるだけだ。[^14]ついでに言うと、*Cannon Lake PCH* 以降もIPを再使用しているため、*Ice Lake* 、*Elkhart Lake* 、*Jasper Lake* 、*Tiger Lake* のPCHに大きな違いはない。[^16]  
強いてそれらを分けるとするならば、*Mule Creek Canyon PCH* は *Ice Lake PCH* 寄り、[^15]  
*Jasper Lake PCH* は *Tiger Lake PCH* 寄りの仕様と言える。[^14]  
  

[^14]: [drm/i915: Introduce Jasper Lake PCH · torvalds/linux@943682e](https://github.com/torvalds/linux/commit/943682e3bd19385511171d730499120ab7245566)
[^16]: [platform/x86: intel_pmc_core: Add Atom based Jasper Lake (JSL) platfo… · torvalds/linux@16292be](https://github.com/torvalds/linux/commit/16292bed9c56a20715d942fd5d9e025f01fa65fe)

他にはメモリのチャネル数の違いが2つを分ける *可能性* があるが、それだけではないはずだ。  
*Tremontアーキテクチャ* には特定用途向けの *Single Cluster Mode* が存在し、テクニカルライター 大原雄介氏は以下のように推測している。  

 >  　ちなみに、気になるのは“Single Cluster Mode”なるモードの搭載で、言葉通りに読めば、ハイパースレッディングを無効にして、最大で6命令/サイクルのデコードが可能なモードもあるように見える。  
 >  　“on product targets”というのは、現在のAtomは単にローエンドのモバイル製品(や一部NUCなどの製品)のみならず、ネットワーク機器や組み込み機器などにも使われているからだ。  
 >  　こうした用途の中にはマルチスレッド性能が不要なのでシングルスレッド性能を上げてほしい、というものも存在するので、こうした特定用途向けにSingle Cluster Modeが利用可能、という話であろう。  

 > 引用元: <cite>[ASCII.jp：AtomベースのSmall CoreがTremontと判明　インテル CPUロードマップ (2/4)](https://ascii.jp/elem/000/001/981/1981403/2/)</cite>

恐らく、2つのプロセッサを分ける最大の要素は、この *Sigle Cluster Mode* が有効にされているか否かであり、  
組み込み向けの *Elkhaet Lake* では有効にされ、モバイル向けの *Jasper Lake* では無効にされ、代わりに *Hyper-Threading* が有効にされているのではないだろうか。

しかし、それと上述した *Jasper Lake* は最大4スレッドという考えを合わせると、*Jasper Lake* は 2-Core/4-Thread となるが、Atom系にしても小さいように思える。  
現世代Atomプロセッサ、*Gemini Lake* は 4-Core で *Hyper-Threading* はサポートしないため、アーキテクチャが *Tremont* である *Jasper Lake* では 2-Core/4-Thread でも性能向上が見込めるのか、  
それとも、4-Threadより上の *Jasper Lake* が予定されているのか、  
または、*Single Cluster Mode* に関する自分の推測が外れているのか。  
こればかりは新たな情報が出てこないければわからない。  

## Lakefiled {#lakefield}
*Lakefield* は高性能な *Sunny Cove* ベースのコアと省電力に優れる *Tremont* ベースのコアを両方併せ持つ非対称ハイブリッドプロセッサであり、  
I/O系を集約し、22nm(P1222)で製造される Base die、CPU、GPU、メモリコントローラを集約した、10nmで製造される Compute dieをFoveros技術で3D積層する、Intelの最新技術が詰め込まれた画期的なSoCでもある。
DRAMも積層されるが、それはPOP(Package-on-package)技術による積層となる。  

参考: <cite><https://www.hotchips.org/hc31/HC31_2.10_LKF_HC_2019_Final_v7.pdf></cite>

ただまだ情報は少なく、CPU関連では先日の Intel 拡張命令リファレンスのアップデートでハイブリッドプロセッサの情報を示すEAXレジスタが追加されたくらいだ。[^17]
*Sunny Cove* と *Tremont* で対応する命令の範囲は異なるが、コンパイラやスケジューラはどのように見ることになるのだろうか。

[^17]: [Intel、拡張命令リファレンスをアップデート (Sapphire Rapids /Alder Lake /ハイブリッドプロセッサ) | Coelacanth's Dream](/posts/2020/04/01/intel-isa-extensiton-update-sapphirerapids-alderlake/#hybrid-processor)

GPUはもう少し明らかにされており、GPUアーキテクチャは *Ice Lake /Elkhart Lake /Jasper Lake* と同世代の Gen11で、EU数は最大64基、  
メディア関連も Gen11だが、*Ice Lake* 同等の機能を持つのか、*Elkhart /Jasper Lake* と同等になるのかは不明。EU 64基と、*Ice Lake* 同等であることに期待が持てるが果たして。  
Display Coreは Gen11.5と進んでおり、映像出力数は最大4つ、解像度は最大5k60か4k120となっている。  

