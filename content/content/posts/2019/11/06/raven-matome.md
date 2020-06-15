---
title: "Raven Matome"
date: 2019-11-06T00:17:40+09:00
draft: false
tags: ["Radeon", "GCN", "Raven", "GFX9", "gfx902", "gfx909" ]
keywords: [ "Radeon", "Raven", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

ZenとVegaによる優秀な性能とコスパで広くに人気なAPU、Ravenについてのまとめ。  

### スペック
まず一番最初に出たRavenのスペックをざっとまとめたのが以下。  

||Raven|
|:--|:--:|
|CMOS|14nm|
|CPU|Zen(+)|
|Max CPU Core/Thread|4/8|
|L3$ CPU|4MB|
|GPU|Vega(GFX9)|
|GPU Clock| 1250MHz
|ShaderEngine|1|
|Max CUs|11|
|Max TMUs|44|
|Max ROPs|8|
|L2$ GPU|1MB|
|Memory Type|DDR4|
|Support Memory Speed|2933MHz|
|VCN ver|1.0|
|DCN ver|1.0/1.01|
|DeviceID|15DD|
|GPU ID|gfx902|
|DieSize|209.78mm2|  
  
使われた製品名は  

* Ryzen 5 2400G/2200G
* Athlon 200GE/220GE/240GE
* Ryzen 3 2200U/2300U
* Ryzen 5 2500U/2600H
* Ryzen 7 2700U/2800H
* Ryzen Embedded V1202B/V1605B/V1756B/V1807B

14nmなのにZen(+)という書き方をしたのは、RavenにはZen+からの機能や改良点が盛り込まれており、  
キャッシュレイテンシ削減によるIPC向上も確認されているからだ。  
[AMD Ryzen 5 2400G review - Performance - CineBench (and IPC)-Guru3D](https://www.guru3d.com/articles_pages/amd_ryzen_5_2400g_review,8.html)  
CU数はそれまでのAPUよりも多くなり、クロックも向上、CPU性能もZenになったことでかなり引き上げられ、丁度いいバランスとなっている。  
マルチメディアではVCN1.0にてRadeon初のVP9 HWデコードに対応した。  
DCN1.0と1.01の違いだが、正直わからない。  
ほんと初期のRavenは1.0だったらしいのだが、恐らく製品段階では1.01になったと思われる。  

性能以外のことでは、Zenではダイとヒートスプレッダ間の接合にハンダが使われ好評だったのに対し、コスト削減のためかRavenではグリスが使われた。  
そのため、小型PCに詰め込む人やOCは殻割りをおこなったが、ヒートスプレッダの接着部分すぐ近くにチップコンデンサが配置されていたり、PGAなためピンを曲げないようにしなければいけないとかで難易度は高かったようだ。  

個人的な話だと、ある時AmazonにてPro 2400Gのバルクが現れ、その情報が広まると家庭内サーバ目的でECCメモリが使いたい人や趣味に走る人が購入していき、  
すぐに売り切れたことが記憶に残っている。  

### Raven2/Picasso

||---Raven2---|---Picasso---|
|:--|:--:|:--:|
|CMOS|14nm|12nm|
|CPU|Zen(+)|Zen+|
|Max CPU Core/Thread|2/4|4/8|
|L3$ CPU|4MB|4MB|
|GPU|Vega(GFX9)|Vega(GFX9)|
|GPU Clock|1200MHz|1400MHz|
|ShaderEngine|1|1|
|Max CUs|3|11|
|Max ROPs|4|8|
|Max TMU|12|44|
|L2$ GPU|0.5MB|1MB|
|Memory Type|DDR4|DDR4|
|Support Memory Speed|2400MHz|2933MHz|
|VCN ver|1.0|1.0|
|DCN ver|1.01|1.01|
|DeviceID|15DD/15D8|15D8|
|GPU ID|gfx909|gfx902|
|DieSize|154.68mm2?|209.78mm2|  

(2019-11-07追記//)よくよく考えたらRaven2が15DDだというソースを見たことがなく、探しても完全にそうと言えるものはなかった。  
Picassoに関しては15D8で確定している。  
[drm/amdkfd: Add picasso pci id](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=e7ad88553aa1d48e950ca9a4934d246c1bee4be4)  
どうもRaven2はRevisionIDで区別してるらしい。  
こういった記述があれば、  
<https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L85>  
こういったコメントもある、  
[[RFC] drm: Add AMD GFX9+ format modifiers.](https://lists.freedesktop.org/archives/dri-devel/2019-October/240407.html)

> We use family_id + external_rev to distinguish between incompatible GPUs.  PCI ID is not enough, as RAVEN and RAVEN2 have the same PCI device id,

そのためRaven2のDeviceIDには15DD/15D8の2つがある、ということにしておく。  
(//追記終了)
  
使われた製品は、  

* Raven2
    * Ryzen 5 3200U
    * Athlon 300U
    * Ryzen Embedded R1505G
    * Ryzen Embedded R1606G
* Picasso
    * Ryzen 5 3400G/3200G
    * Ryzen 3 3300U
    * Ryzen 5 3500U/3550H/3580U
    * Ryzen 7 3700U/3750H/3780U

この2つを簡潔に説明すると、  
Raven2は14nmのままRavenをより小型に設計、  
PicassoはRavenをまんま12nmにした、  
という感じだ。  

それとRaven2のダイサイズは画像を元に求めたものなので、あまり当てにならない。  

この2つは同時期(2018-09頃)にパッチが投稿され、私としてはどう違うのかよくわからず少し混乱したのを覚えている。  

### Renoir/Dali
この2つはまだ製品が出ておらず、Renoirはパッチが色々投稿され、あれこれ噂が出てきているが、Daliの方は名前が出て以降さっぱり。  
とりあえずわかっていることは、  

* Renoir
    * LP/DDR4/Xをサポート(Max Memory Speed 4266MHz)
    * VCN2.0(Navi1xと同一) [drm/amdgpu: add VCN2.0 to Renoir IP blocks](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=279ba48e1f7677008e6e04d3eeb37b81b7496ed0)
    * DCN2.1でDSC搭載
    * DeviceID: 1636 [drm/amdgpu: add renoir pci id](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=61bdb39c913fcd998faef7664a5f2f0170000335)
    * GPU ID: gfx909
* Dali
    * DCN1.01(Raven/2、Picassoと同一)
    * Raven2ベース [drm/amd/display: add Asic ID for Dali](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=c4cacce7850042ad6b65d518d47497c0ae70c8b6)

くらいだ。  

DaliがRaven2ベースと言うが、RenoirもGPU IDはraven2と同一であり、どう差別化するのかうまく想像できない。 [ac_llvm_util.c](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/llvm/ac_llvm_util.c#L145)  
RenoirはCPUがZen2であると噂されており、それならわかりやすいが、Daliの方はどうするのだろうか。  
海外メディアの推測では、より低消費電力用途向けではないかということだったが、Raven2からさらに削るつもりかプロセスを14/12nmより進んだものにするのか。  

(2019-11-07追記//)  Athlon 3000Gの名前を聞いて思いついたのだが、DaliはRaven2をまんま12nmにしたものかもしれない。[Athlon 3000G? - 北森瓦版](https://northwood.blog.fc2.com/blog-entry-10024.html)  
それなら特にパッチが投稿されないのも噂されないのも納得が行くような気がする。  
[drm/amd/display: add Asic ID for Dali](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=c4cacce7850042ad6b65d518d47497c0ae70c8b6)  
これを見るにDaliはPicassoの新リビジョン(E3、E4)のように読める。  
まだ確定した訳ではないが、最初からちゃんと読んでおくべきだった。  
それと今回は追記日時を書いたが、このサイトはGithub Pageを利用しているため、  
ここを見れば更新日時と内容がわかるようになっている。  
<https://github.com/Umio-Yasuno/umio-yasuno.github.io>  
(//追記終了)