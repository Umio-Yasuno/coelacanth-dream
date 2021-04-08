---
title: "Cannon Lake を追う"
date: 2020-09-29T18:02:08+09:00
draft: false
tags: [ "Gen10", "Cannon_Lake" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

「*Cannon Lake* はキャンセルされた」というのが共通認識としてあるが、具体的にいつ頃キャンセルされたかはすぐには浮かばない。自分がそうだ。  
調べてみたが明確な答えは無いし、Intel 内部での決定がいつ下されたかは知りようがない。  
思うに、せめてものと 2018/08 に Core i3-8121U(Cannon Lake) を搭載した NUC が発表され[^cnl-nuc]、そして 2019/01 の CES で *Ice Lake* が発表され[^ces-icl]、世間の興味がそちらに移った頃、そのあたりが個人的な感覚で 「*Cannon Lake* がキャンセルされた」と言うにふさわしい時期なのではだろうか。  

[^cnl-nuc]: [Intel Introduces New NUC Kits and NUC Mini PCs to the Intel NUC Family | Intel Newsroom](https://newsroom.intel.com/news/intel-introduces-new-nuc-kits-nuc-mini-pcs-intel-nuc-family/)
[^ces-icl]: [2019 CES: Intel Showcases New Technology for Next Era of Computing | Intel Newsroom](https://newsroom.intel.com/news/2019-ces-intel-showcases-new-technology-next-era-computing/)

そして、オープンソースドライバーの Mesa3D には *Cannon Lake* のみを対象としたコードを取り除くマージリクエストが投稿され[^remove-cnl-only-code]、[intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler) では、*Cannon Lake* プラットフォームはもう死んだものとしている。[^dead-cnl-platform]  

[^remove-cnl-only-code]: [intel: Remove Gen10 specific code (!6899) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6899)
[^dead-cnl-platform]: [Remove the dead CNL platform. · intel/intel-graphics-compiler@6e9f0af](https://github.com/intel/intel-graphics-compiler/commit/6e9f0af19184cdc3f94c32c23692eaccd3b0ef20#diff-a814fa78344f64350e2584d78c11ae0eR145)

そうして *Cannon Lake* が埋められ、最新の CPU/GPU の情報を追う中で忘れる前に、一度 *Cannon Lake* がどんなプロセッサであったかをまとめて記しておきたい。  

## Cannon Lake {#cnl}
### CPU {#cpu}
CPU のキャッシュ構成は *Skylake* 同様に、コアあたり L1D/Iキャッシュは 32KB、L2キャッシュは 256KB、L3キャッシュは 2MB。  
*Skylakeアーキテクチャ* から対応命令が追加されており、AVX512系、SHA-NI に対応している。  
だが CPUアーキテクチャの詳細に関してはほとんど明かされておらず、AVX512 をどのように実行しているかは不明。512-bit ベクタユニットを持つ 1つのポートで実行するのか、命令を分解し、256-bit ベクタユニットを持つ 2つのポートで実行するのか。  
*Ice Lake (Client)* は前者の方式でサポートし、*Skylake (Server)* と *Ice Lake (Server)* は両方の方式を併せ持つ。  
[AnandTech](https://www.anandtech.com) の Ian Cutress 氏が検証を行なった所では、シングルスレッドで y-Cruncher を実行した時の性能が *Kaby Lake* から大きく向上しているため、*Ice Lake (Client)* 同様の実装になっている可能性が考えられる。[^anandtech-cnl]  
しかし、それは 2.2GHz に揃えた検証の結果であり、AVX使用時のクロックでは *Kaby Lake* が 1GHz 近く高いこともあり、デフォルトの TDP 15W で実行した際の性能はそう変わらないものとなっている。  

[^anandtech-cnl]: [Stock CPU Performance: System Tests - Intel's 10nm Cannon Lake and Core i3-8121U Deep Dive Review](https://www.anandtech.com/show/13405/intel-10nm-cannon-lake-and-core-i3-8121u-deep-dive-review/9)

このあたり、初期の 10nmプロセスとそれを採用する *Cannon Lake* がキャンセルされた理由として、動作クロックの低さが大きかったように思える。  
*Ice Lake* で採用された 10nm(+) では歩留まりと動作クロックの改善が施され、CPUアーキテクチャも IPC向上に力を入れたものとなった。  

### GPU {#gpu}
GPU は *Skylake* や *Gemini Lake* 等より世代が進んだ *Gen10アーキテクチャ* となるが、採用は *Cannon Lake* のみとなった。  

各バリアントでの GPU 規模は一部ソースコードに記述されているが、微妙に仕様が異なっている。  
恐らく、Mesa3D は *Cannon Lake-U/Y (2+2)* で想定されていたバリアントが記述され、media-driver ではそれよりも上位の SKU、例えば *Cannon Lake-H(仮)* 等も想定したものなのだろう。  

| Gen10 GPU<br>(Mesa3D) | GT0.5 | GT1 | GT1.5 | GT2 |
| :-- | :--: | :--: | :--: | :--: |
| Slice | 1 | 1 | 2 | 2 |
| Sub-Slice | 2 | 3 | 4(2+2) | 5(2+3) |
| L3cache Bank | 2 | 3 | 6 | 6 |
| Total EU | 16 | 24 | 32 | 40 |

出典: [src/intel/dev/gen_device_info.c · 559b26b7ee093c2cbe446bf8023876642deadcee · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/559b26b7ee093c2cbe446bf8023876642deadcee/src/intel/dev/gen_device_info.c)

| Gen10 GPU<br>(media-driver) | GT1 | GT1.5 | GT2 | GT3 | GT3e |
| :-- | :--: | :--: | :--: | :--: | :--: |
| Slice | 1 | 1 | 2 | 4 | 4 |
| Sub-Slice | 2 | 3 | 4 | 9 | 9 |
| L3cache Bank | 2 | 3 | 6 | 12 | 12 |
| L3cache Size | 512KB | 768KB | 1536KB | 3072KB | 3072KB |
| Total EU | 16 | 24 | 32 | 72 | 72 |

出典: [media-driver/media_sysinfo_g10.cpp at bdbce9ce524e1aaa2a5c9770899d4762f443e0b4 · intel/media-driver](https://github.com/intel/media-driver/blob/bdbce9ce524e1aaa2a5c9770899d4762f443e0b4/media_driver/linux/gen10/ddi/media_sysinfo_g10.cpp)

*Cannon Lake GT3/e* の 9Sub-Slice、72EU、eDRAM対応という点は *Skylake GT4/e* と同じだが、*Skylake GT4/e* が 3Slice、L3キャッシュサイズ 2304KB、バンクあたり 192KB だったのに対し、  
*Cannon Lake GT3/e* では 4Slice、L3キャッシュサイズ 3072KB、バンクあたり 256KB という構成になっている。  

*Gen10アーキテクチャ* の詳細もまた明かされていないが、その構成から、*Gen9アーキテクチャ* よりも Slice数、バンクあたりの L3キャッシュサイズを増やし、グラフィクス性能の向上を図ったものと思われる。  

ディスプレイ部に関しては、AMD GPU では *RDNA /GFX10* 世代から対応した DSC(Display Stream Compression) に *Gen10アーキテクチャ* から対応していたことが窺える記述がある。[^gen10-dsc]  

[^gen10-dsc]: [linux/i915_pci.c at 8186749621ed6b8fc42644c399e8c755a2b6f630 · torvalds/linux](https://github.com/torvalds/linux/blob/8186749621ed6b8fc42644c399e8c755a2b6f630/drivers/gpu/drm/i915/i915_pci.c#L786)

しかし、結局 *Cannon Lake* の GPU が有効化されることはなく、唯一製品に搭載、出荷されている **Core i3-8121U** では無効化されている。  
そのため、*Cannon Lake* が死んだというよりも、その GPU部、*Gen10アーキテクチャ* が埋められようとしていると言った方が正確だろう。  


### GNA {#gna}
*Ice Lake* に搭載された推論アクセラレーター、GNA(Gaussian mixture model and Neural network Acceleration unit) だが、*Cannon Lake* にも搭載されており、GNA の世代は *Ice Lake* とは変わらず、Gen1 となっている。[^cnl-gna]  
さらに言えば、GNA は *Cannon Lake* から搭載されたという訳でもなく、Atom系の *Gemini Lake* から搭載されているらしい。こちらも Gen1 となる。  

[^cnl-gna]: [acrn-kernel/gna_pci.c at 30000faef6e05a0e2430ff2a7f61539f86970d70 · projectacrn/acrn-kernel](https://github.com/projectacrn/acrn-kernel/blob/30000faef6e05a0e2430ff2a7f61539f86970d70/drivers/misc/gna/gna_pci.c#L39)

そして、調査(ソースコード漁り)の過程で知ったのだが、Atom系である *Tremontコア* を採用する [Elkhart Lake](/tags/elkhart_lake) と [Jasper Lake](/tags/jasper_lake) にも GNA が搭載されているものの、両者で世代が異なり、  
*Elkhart Lake* は *Ice Lake* 等と同じ GNA Gen1 を搭載するのに対し、*Jasper Lake* は *Tiger Lake* と同じ GNA Gen2 を搭載する。  
CPUアーキテクチャや GPU部等、共通点のある両者のプロセッサ間で、GNA の世代を異にした理由は不明。  
GNA Gen2 で何が変わったかは GNA で扱える最大レイヤー数が増えたということくらいしか分からない。[^gna-gen2]  

[^gna-gen2]: <https://github.com/projectacrn/acrn-kernel/blob/30000faef6e05a0e2430ff2a7f61539f86970d70/drivers/misc/gna/gna_pci.c#L35>

GNA は GPU とは違い、**Core i3-8121U** でも有効化されている。[^gna-enable-i3-8121u]  

[^gna-enable-i3-8121u]: [GNA Plugin - OpenVINO™ Toolkit](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_GNA.html)

## 終わりに

*Cannon Lake* は *Skylake* 系プロセッサから正統に拡張、強化が施され、メモリインターフェイスでは *Ice Lake* よりも先に LPDDR4 をサポートしていた。  
Intel 10nmプロセス開発難航の影響を受け、歩留まり、動作クロックに恵まれなかったが、アーキテクチャは CPU/GPU 共に進んでいた。  
*Cannon Lake* は確かに *Skylake* と *Ice Lake* とを繋ぐプロセッサであったと言えるだろう。  

### 余談

以前、*Cannon Lake-H* の名がベンチマーク結果から現れ、少し話題となったが、プラットフォーム名とコア/スレッド数しか分からず、調べても特に見つからなかったため、ここでは取り扱わなかった。  

また、Coreboot に *Coffee Lake RVP* ボード関連のファイルが追加された時、*Cannon Lake RVP* 関連のファイル名と変数名をリネームし、他はコピーといった感じであった。[^cfl-rvp]そのため、実際には *Cannon Lake-H* ではなく初期の *Coffee Lake-H* が動作していた可能性も否定できない。  
半ば *Cannon Lake-H* が都市伝説とも思える中で惜しまれるのは、やはり情報の少なさである。  

[^cfl-rvp]: [mb/intel/coffeelake_rvp: Add new mainboard coffeelake RVP (If9a5a53e) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/25122)

Intel は *Cannon Lake* に関する資料をまだ公開しておらず、ark.intel.com の *Core i3-8121U* ページの Technical Documentation には何も置かれていない。  
{{< link >}} [Intel® Core™ i3-8121U Processor (4M Cache, up to 3.20 GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/136863/intel-core-i3-8121u-processor-4m-cache-up-to-3-20-ghz.html?wapkw=cannon%20lake) {{< /link >}}
いつか資料が公開され、Intel の方から再び *Cannon Lake* を語ってくれる日を期待している。  
