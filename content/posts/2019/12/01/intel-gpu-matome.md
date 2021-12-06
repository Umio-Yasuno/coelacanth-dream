---
title: "Intel GPU Matome (Gen10/11/12)"
date: 2019-12-01T07:56:37+09:00
draft: false
tags: [ "Gen10", "Gen11", "Gen12" ]
keywords: [ "Intel", "Gen10", "Gen11", "Gen12" ]
categories: [ "Hardware", "GPU", "Intel" ]
---

Tiger LakeだのDG1だのGen12だので、最近Intel GPUが盛り上がっているため、  
公開されているコードをハードウェアや製品に結び付けられる程度には読み方を覚え、まとめてみた。  

公開されている情報量は多いが、それなりに整理がされており、付随するコメントも親切でやりやすくはあった。  

Sandybridgeとかからまとめるのはさすがに量が多くなりすぎてしまうため、Skylake (Gen9) からとした。  
今も現役な14nm世代であり、特に複雑なのが丁度そのあたりなため、まとめる意義があるはずだ。  

スペース削減のため、表ではSkylake以降のコードネームにおいて末尾の Lake を「L」1文字に省略している。  
Kaby Lakeと、Skylake以降のコードネームでは Lake が分離しているため、わざわざ記述する必要もないと思う。  
それと略称の最後がLなら、まず Lake だと言っていい。  
ただGemini Lakeだけは例外で、GLKと略されている。Atom系だから？  

| Intel GPU | a.k.a. | Memo |  | Gen | Device ID |
| :--- | :---: | :---: | :---: | :---: | ---: |
| Skylake | SKL | | | Gen9 | 0x1902 - 0x193D |
| Kaby L | KBL | | | Gen9 | 0x5906 - 0x593D |
| | | Amber L (AML) | | Gen9 | 0x591C, 0x87C0 |
| Coffee L | CFL | | | Gen9 | 0x3E90 - 0x3EA8 |
| | | Amber L (AML) | | Gen9 | 0x87CA |
| | | Whiskey L (WHL) |  | Gen9 | 0x3EA0 - 0x3EA4 |
| | | Comet L (CML) |  | Gen9 | 0x9B21 - 0x9BF6 |
| Broxton | BXT | Atom | Canceled | Gen9 | 0x0A84, 0x1A84, 0x1A85 |
| | | Apollo L (APL) | | Gen9 | 0x5A84, 0x5A85 |
| Gemini L | GLK | Atom | | Gen9 | 0x3184, 0x3185 |
| WillowView | | = Willow Trail? | Canceled | |
| GlenView | GLV | Atom? | ??? | Gen9 | 0x3E04 |
| Cannon L | CNL | | | Gen10 | 0x5A42 - 0x5A5C |
| Ice L (LP) | ICL (LP) | | | Gen11 | 0x8A50 - 0x8A71 | 
| LakeField | LKF | | | Gen11 | 0x9840 - 0x9850 |
| Jasper L | JSL | | | Gen11 | 0x4500 |
| Elkhart L | EHL | = Jasper | | Gen11 | 0x4500 - 0x4569 |
| Tiger L (LP) | TGL (LP) | | | Gen12 | 0x9A40 - 0x9A7F |

参考: [igfxfmid.h - intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/master/inc/common/igfxfmid.h)  
（追記 2019/12/02 01:23）GlenViewのGenをGen9に修正。 指摘を受けて見直したらそこにはGen9。  
<https://github.com/intel/intel-graphics-compiler/blob/862d1d982b4a54417e801b55cb63019b7c2322e4/IGC/common/SystemThread.h#L85>  

真面目に読み込んでみた感想としては、知らないコードネームがちらほらあって驚いた。  
Broxton、WillowView、GlenViewとか。  
GlenViewは、その名前とGen9であることくらいしか判明していない。  
BroxtonとWillowViewはキャンセルされており、影も薄くなるよなという感じ。  

それとAmber、Whiskey、Cometは少しややこしいが、PCHでもっとややこしくなる。  


### スペックの知り方

Intelのmedia-driverレポジトリ内のファイルを読むのが手っ取り早いし正確だ。  

	github.com/intel/media-driver/tree/master/media_driver/linux/genX/ddi/media_sysinfo_gX.cpp  

（Xには世代の数字が入る。）  

<https://github.com/intel/media-driver/tree/master/media_driver/linux>  

以下、Gen10/11/12のびみょうなまとめ。  

#### Gen10
Cannon Lakeだけの世代となり、そのCannon Lakeも一応世にではしたけれどGPUが無効にされていたため実性能がわかっていない幻のGen10のファイルを読むと、  
GT3、GT3eのスペックも記述されており、規模だけで言えばIce LakeのGen11より大きい72EU、さらにはeDRAM付きもありえたかもしれないというのが面白い。  

それとスペックがMesaの方と微妙に差異がある。  
[i965_pci_ids.h - Mesa](https://gitlab.freedesktop.org/mesa/mesa/blob/master/include/pci_ids/i965_pci_ids.h#L213)  
[gen_device_info.c - Mesa](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/intel/dev/gen_device_info.c#L855)  
MesaではGen10 GT2が総サブスライス数:5、サブスライスあたりのEU数:8とのことから合計40EUと読めるが、  
media-driverでは総サブスライス数:4、サブスライスあたりのEU数:8と、合計32EUとなっている。  

それとMesaにはGT0.5/1/1.5があるのに対し、media-driverではGT1/1.5だけであり、  
スペックで見れば、 40EUの構成を消して下から押し上げて数字を当てはめたという感じだ。  
なんとかGPUの仕様を変えて出荷しようと迷走した結果なのだろうか。  

#### Gen11
Ice Lakeで真新しい情報は特にない、はず。  
だがGen11を搭載するもう１つのElkhart Lakeでは、32EU、L3キャッシュ 1280KBとされていた。  
このElkhart Lake、名前こそ結構コードで出ているものの、詳細はまだベールに包まれている。  
Atom系だとは言われているが。  
Jasper Lakeという名前もあるが、PCH以外Elkhart Lakeとほぼ同一と思われる。  

LakeFieldは、media-driverに詳細はまだないが、intel-grahics-compilerでの名前から64EUだとわかる。  
<https://github.com/intel/intel-graphics-compiler/blob/master/inc/common/igfxfmid.h#L569>

また、LakeFieldのAtom、GPU側がElkhart Lakeと読むことができ、  
[gen_perf.c - Mesa](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/intel/perf/gen_perf.c#L790)  
Gen11だという発表とも一致する。  
[Intelが3D積層のヘテロジニアスマルチコアCPU「Lakefield」の技術を発表 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1204371.html)  

アーキテクチャ面では、Gen11のスライス数が1までで、サブスライスを増やすといった構成になっているが、  
これがどう作用しているのかは（私が）よくわからない。  

L3キャッシュはバンクあたり384KBと、それまでの256KBから増量された。  

ディスプレイ周りではDSC（Display Stream Compression）に対応してる、はずだが特にアピールされていない。  
[Display Stream Compression enabling on eDP/DP - Patchwork Intel GFX](https://patchwork.freedesktop.org/series/47514/)  
現状Ice Lakeはモバイル向けしか出ていないのに、8Kディスプレイへの接続を推しても……ということなのだろうか。  
Gen10でもDSCらしき記述があるがこれも、GPUが無効化されているのにそんな話しても……といった感じか。  
[i915_pci.c#n736](https://cgit.freedesktop.org/drm-intel/tree/drivers/gpu/drm/i915/i915_pci.c#n736)  

#### Gen12
Gen12はまだmedia-driverにはないが、Mesaには記述されている。  
<https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/intel/dev/gen_device_info.c#L1035>  
スペックではGT1が32EU、GT2が96EUとなり、スライス数、<del>バンクあたりのL3キャッシュはGen11を継承して1、384KB。  
合計L3キャッシュ容量もIce Lakeと同じ3072 KB。</del>  

{{< ins datetime="2020-01-17" >}}
IntelのGithubレポジトリの1つ、compute-runtime よりL3キャッシュは8バンク、3840 KBという構成だとわかった。  
[hw_info_tgllp.inl - Intel / compute-runtime](https://github.com/intel/compute-runtime/blob/master/runtime/gen12lp/hw_info_tgllp.inl#L120)  

バンク数はIcelakeと同じであることから、バンクあたりの容量をGen11の384 KBから480 KBに増やしたと推測される。  

{{< /ins >}}

 Tiger Lake LPという名前でありながら、intel-graphics-compilerにはID名に DESK_65W があったり。  
<https://github.com/intel/intel-graphics-compiler/blob/master/inc/common/igfxfmid.h#L557>  
DESK_WS_65Wというのもあるが、  
intel-graphics-compilerにはキャンセルされたものや、予定していたであろうCannon LakeのIDが大量に残されていたりするため、  
記憶の隅に置いておく程度が丁度いいだろう。  

subslicesが、dual_subslicesとなっているがこれまたどういった狙いがあるかは不明。  
何となくRDNAのWGP（DualCU）が思い浮かぶが、AMDからIntelに移籍したRaja Koduri氏と関係しているのだろうか……  

#### DG1
世代的にはGen12_5とされているが、Gen12（Tiger Lake LP）とどこまで違うかは不明。  
キャッシュ、メモリ構成、規模等で分けているのだろうか？  
[[Intel-gfx] [PATCH 0/1] dg1: enable dsb back](https://lists.freedesktop.org/archives/intel-gfx/2019-November/219298.html)  

dGPU関連ではMulti-GPUのサポートが追加されている。  
[[Intel-gfx] [CI] drm/i915/pmu: Support multiple GPUs](https://lists.freedesktop.org/archives/intel-gfx/2019-October/216404.html)  
iGPU側は特に指定されていないため、Gen9でも恩恵を受けられるかもしれない。  

オープンな範囲で公開されている情報と私の理解力では、まだDG1はこの程度の情報しか見つけられなかった。  
かといって噂レベルの情報を取り入れても混乱するため、ここまでとする。

次はIntel PCHをまとめるつもりだが、やる気次第。  
