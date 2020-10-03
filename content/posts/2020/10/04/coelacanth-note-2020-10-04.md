---
title: "【シーラカンスノート】 Alder Lake-P RVP, ADL-S Gen12…… 【2020/10/04】"
date: 2020-10-04T01:10:10+09:00
draft: false
tags: [ "Alder_Lake", "Lakefield" ]
# keywords: [ "", ]
categories: [ "Note", ]
noindex: false
# summary: ""
toc: false
---

{{< pindex >}}

 * [Alder Lake-P RVP ITE](#adlp-rvo)
 * [Alder Lake-S Gen12](#adls-gen12)
 * [ハイブリッドプロセッサのトポロジを検出、出力するパッチ](#hybrid-topology)
 * [DG1/SG1](#dg1-sg1)

{{< /pindex >}}

## Alder Lake-P RVP ITE {#adlp-rvo}
Chromium OS のプロジェクトの 1つである EC(Embedded Controller) に Alder Lake-P RVP ITE をサポートが追加された。  
{{< link >}} [adlprvp: add Alderlake RVP support (I9d85e0cb) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2435177) {{< /link >}}

Tiger Lake RVP のファイルと軽く見比べたところでは、USB PD をサポートするポートが 2基から 4基に増やされているようだ。  
また、`#define CONFIG_CHIPSET_TIGERLAKE` という記述があった。  
*Alder Lake-P* はモバイル向けのモデルではないかと思われるが、それと組み合わせられる PCH は *Tiger Lake LP PCH* が続投するのか、それとも仮のコードか。  

## Alder Lake-S Gen12 {#adls-gen12}

[The Intel® Media SDK](https://github.com/Intel-Media-SDK/MediaSDK) にデスクトップ向けモデルであろう *Alder Lake-S* プラットフォームをサポートするプルリクエストが投稿された。  
{{< link >}} [[ADL-S] Enable ADL-S platform support by aidan2020sh · Pull Request #2381 · Intel-Media-SDK/MediaSDK](https://github.com/Intel-Media-SDK/MediaSDK/pull/2381/files) {{< /link >}}

[以前](/posts/2020/09/14/intel-adls-gen12-gpu/) にも書いたが、やはり *Alder Lake-S* の GPU部は *Rocket Lake, Tiger Lake, DG1/SG1* 等と同じ Gen12LPアーキテクチャとなるようだ。  

## ハイブリッドプロセッサのトポロジを検出、出力するパッチ {#hybrid-topology}

ハイブリッドプロセッサが持つ、アーキテクチャが異なるコアのトポロジを検出し、sysfsインターフェイスに出力するパッチが投稿された。  
{{< link >}} [LKML: Ricardo Neri: [PATCH 0/4] drivers core: Introduce CPU type sysfs interface](https://lkml.org/lkml/2020/10/2/1208){{< /link >}}

Intel のハイブリッドプロセッサでは既に *Lakefield* がいるが、Linux でのサポートはまだ途中段階と言え、Geekbench で *Lakefield* の結果を見ても OS は Windows ばかりである。  
ユーザーモードの GPUドライバー、Mesa3D にも *Lakefield* をサポートするようなパッチは未だ投稿されていない。  
Coreboot 等にも *Lakefield* のリファレンスボード関連のファイルが追加されていないあたり、特定のメーカー、特定の OS(Windows) までにしか提供できない段階にあるのかもしれない。  

先日、Lenovo より *Lakefield* を搭載する **ThinkPad X1 Fold** の発売が発表されたが、価格は税別で &yen;363,000 という、インターフェイス部が特殊とはいえ性能比で考えるとぶっ飛んだ価格ではある。[^lenovo-lkf]  
また、Microsoft の同様に *Lakefield* 搭載する **Surface Neo** も 2021年以降に延期されているのを見ていると、*Lakefield* は実験的なハイブリッドプロセッサであり、*Alder Lake* から本格的な段階に入るのではないかと思える。[^ms-lkf]  
*Lakefield* はハイブリッドプロセッサであること以外に、3Dパッケージング技術によるパッケージの小ささや消費電力の低さ等の革新的な面があるため、それを活かした製品となると複雑になってしまうのかもしれないが。  

一応、*Lakefield* を搭載した標準的なラップトップを唯一 Samsung が出しているため、 *Lakefield* を試したい人はそちらを買うのだろう。[^samsung-lkf]  

[^lenovo-lkf]: [レノボ、折りたたみ有機EL搭載パソコン「ThinkPad X1 Fold」を10月13日に発売 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1278746.html)
[^ms-lkf]: [Microsoft、2画面のSurface Neo発売を2021年以降に延期 - PC Watch](https://pc.watch.impress.co.jp/docs/news/1250856.html)
[^samsung-lkf]: [Galaxy Book S Wifi – 13” Gigabit Wifi Laptop | Samsung US](https://www.samsung.com/us/computing/windows-laptops/galaxy-book-s/wi-fi/)

## DG1/SG1 {#dg1-sg1}

[intel/metrics-discovery](https://github.com/intel/metrics-discovery) では [Mesa3D](https://gitlab.freedesktop.org/mesa/mesa) にホストされているレポジトリよりも内容が若干先行しているらしく、Intel のサーバー向け dGPU *SG1* のデバイスID等が既に追加されている。  

 >      CHIPSET(0x4905, dg1, "DG1 GT2", "Intel(R) Graphics")
 >      CHIPSET(0x4906, dg1, "DG1 GT2", "Intel(R) Graphics")
 >      CHIPSET(0x4907, sg1, "SG1 GT2", "Intel(R) Graphics")
 >      CHIPSET(0x4908, dg1, "DG1 GT2", "Intel(R) Graphics")
 >
 > {{< quote >}} [metrics-discovery/iris_pci_ids.h at 3771d7da972db4904e8aa48d5ebc2059d432a689 · intel/metrics-discovery](https://github.com/intel/metrics-discovery/blob/3771d7da972db4904e8aa48d5ebc2059d432a689/external/mesa/include/pci_ids/iris_pci_ids.h) {{< /quote >}}

それ以外に、当初は `0x4905` のみだった *DG1* のデバイスIDも 2つ追加されている。  
ネット上では [APISAK (@TUM_APISAK](https://twitter.com/TUM_APISAK) 氏によって *DG1* と同規模でありながら別の名前を持つ GPU のベンチマーク結果が発見されている。  
{{< link >}} [Details for Result ID Intel(R) Iris(R) Xe MAX Graphics (768SP 96C 1.55GHz, 1MB L2, 3GB) (OpenCL) : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_run.php?q=c2ffcdf4d2b3d2efdae9daefdce9cfbd80b096f396ab9bbdcef3cb&l=en) {{< /link >}}

*DG1* はモバイル向けの dGPU としても提供されるようだが、その時は *Tiger Lake* GPU のブランド名に合わせた名前となるのだろうか。  

それと、[intel/metrics-discovery](https://github.com/intel/metrics-discovery) には *Gen12LP(Gen12.1)* よりも規模が大きくなるとされる *Gen12.5* のマクロも追加されていた。  
{{< link >}} [metrics-discovery/gen_device_info.c at f69d861459c19a9e7bddef5a05df124e182a5af4 · intel/metrics-discovery](https://github.com/intel/metrics-discovery/blob/f69d861459c19a9e7bddef5a05df124e182a5af4/external/mesa/src/intel/dev/gen_device_info.c#L1036) {{< /link >}}
Slice数が 1 で固定されている `GEN12_GT_FEATURES` を用いていないことに加え、*Gen12LP* よりも高性能をターゲットする以上、Slice数を増やしてくると思われる。  


