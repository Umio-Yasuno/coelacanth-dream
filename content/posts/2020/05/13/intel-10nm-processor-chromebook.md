---
title: "Intel 10nm プロセッサを搭載した Chromebook ボード"
date: 2020-05-13T09:44:09+09:00
draft: false
tags: [ "Chromebook", "Tremont", "Tiger_Lake", "Jasper_Lake" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

以前、Ryzen APUを搭載する Chromebook に関する情報をまとめたが、今回は将来出てくるであろう Intel 10nm世代プロセスで製造されたプロセッサを搭載する Chromebook についてまとめていきたい。  
{{< link >}}[Ryzen搭載Chromebookの情報を整理する | Coelacanth's Dream](/posts/2020/04/10/ryzen-chromebook-arr/){{< /link >}}

AMD は現在 Coreboot における [12nm Picasso](/tags/picasso) 、[14nm Raven2 (Dali/Pollock)](/tags/raven2) のサポートを進め、それら搭載する Chromebook や System76ラップトップ[^1] の開発もまた進んできているが、  
APU の最新世代は [7nm Renoir](/tags/renoir) であり、*Picasso* 、*Raven2 (Dali/Pollock)* はプロセス的にもCPUアーキテクチャ的にも1世代前のプロセッサだ。  

[^1]: [System76 May Offer AMD Ryzen Laptops When They Begin Their Own Manufacturing - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=System76-AMD-Laptop-Possibility)

対して Intel は 10nm世代プロセスで製造される次世代プロセッサ *Jasper Lake* 、*Tiger Lake* の Corebootサポートに積極的に取り組み、それらを搭載する Chromebook の開発も既に進んでいる。  
これは、Intel は元から RVP(Reference Validation Platform)ボードのサポートを行なってきたというのが大きいだろう。  
ここに Intel と AMD のソフトウェア開発に割けられるリソースの違いが見られる気がする。  

## インデックス

 * [Dedede](#dedede)
    * [Jasper Lake](#jsl)
    * [Dedede 仕様](#dedede-spec)
 * [Volteer](#volteer)
    * [Tiger Lake](#tgl)
    * [Volteer 仕様](#volteer-spec)
 * [おまけ、 開発されていた Cannon Lake 搭載 Chromebook ボード](#cnl)

## Dedede {#dedede}
*Jasper Lake* を搭載する Chromebook ボードのコードネーム。  
{{< link >}}[board/dedede: Initial board file for Dedede (Ibcac0a18) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/depthcharge/+/2006531){{< /link >}}

### Jasper Lake {#jsl}
*Jasper Lake* とは CPU に[Tremont](/tags/tremont)アーキテクチャ、GPUに[Gen11](/tags/gen11)アーキテクチャを採用したコアを搭載し、Intel 10nm(+)プロセスで製造されるモバイル向けプロセッサ。[^2]  
双子的な存在に *Elkart Lake* がおり、こちらは組み込み向けプロセッサとなる。  
GPU部は仕様が一致しており、GPUドライバーからは *Jasper Lake* もまとめて *Elkhart Lake* と認識されたりもする。  

[^2]: [Intel Atom Tremont関連のコードネームを整理する | Coelacanth's Dream](/posts/2020/04/20/intel-atom-tremont-codename-arr/#tremont-l)

GPUは *Ice Lake* と同じGen11アーキテクチャではあるが *Ice Lake* GPUは L3cacheバンクあたりのサイズが最大 384KBなのに対し、*Jasper/Elkhart Lake* GPUは[media-driver](https://github.com/intel/media-driver)から最大 320KBと読める。[^9]  
また、一部コーディックのシェーダー(EU)を用いたエンコードや、デノイズ、シャーピング、HDR10トーンマッピング処理のサポートが *Jasper/Elkhart Lake* では外されているなど、*Ice Lake* GPUとは細かい違いが存在する。  

*Jasper Lake* GPUの規模は、Mesa3D へのコミットの中で EU数 24基、L3cacheバンク 4基という仕様が明らかにされている。[^8]  

### Dedede 仕様 {#dedede-spec}
**Dedede** はリファレンスボードであり、実際に製品に搭載される派生ボードには **Waddledoo** [^3] 、**Waddledee** [^4]、**Wheelie** [^5] が存在する。  
**Dedede** から察しが付くように、コードネームはゲーム 星のカービィから取られている。  
Intel が湖、AMD が 星や画家、イタリアの都市、NVIDIA が歴史的な学者からコードネームを取るように
、Chromebook ボードはゲームや漫画等から取られることが多い。  
コードネームはそのアーキテクチャ/プロセッサ/ボードを象徴し、それぞれの関係性をわかりやすくすると同時に、ユーザーにキャラクターとして親しみをもたらしてくれる。  

[^3]: [UPSTREAM: mb/google/dedede: Add waddledoo variant (Icf147d95) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2070287)
[^4]: [UPSTREAM: mb/google/dedede: Add waddledee variant (I83b38918) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2107467)
[^5]: [UPSTREAM: mb/google/dedede: add new variant for wheelie (Ie857526c) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2175841)

**Dedede** とその派生ボードに搭載される *Jasper Lake* のCPUは 2-Core/4-Thread。[^6][^7]  

[^6]: [UPSTREAM: ASoC: SOF: Intel: initial support to JasperLake. (I0aa932c4) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/kernel/+/1993735)
[^7]: <https://github.com/coreboot/coreboot/blob/a02bf7468a5bb22f47be2aaf6186f2c835710fbc/src/mainboard/google/dedede/Kconfig#L60>
[^8]: [intel: Add device info for 1x4x6 Jasper Lake (11fdd5f5) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/11fdd5f52c3db070f33f7ef82d41acf14b1a2670)
[^9]: <https://github.com/intel/media-driver/blob/50e110b86d1fd0d46d0fd9a5be7a707c07026485/media_driver/linux/gen11/ddi/media_sysinfo_g11.cpp#L328>

現行のAtom系アーキテクチャ *Goldmont Plus* を採用する *Gemini Lake* は最大 4-Core/4-Thread であり、**Dedede** に搭載される *Jasper Lake* は物理コア数で見劣りすることとなるが、  
次世代Atom系アーキテクチャ *Tremont* は *Goldmont Plus* から大幅に強化されているため、マルチスレッド性能で負けることはないはずだ。  
GPU性能は、*Gemini Lake* は最大 18EU、L3cacheバンク 2基(計384KB)という仕様[^10]であるため、確実な性能向上が見込める。  

[^10]: <https://github.com/intel/media-driver/blob/50e110b86d1fd0d46d0fd9a5be7a707c07026485/media_driver/linux/gen9/ddi/media_sysinfo_g9.cpp#L478>

ディスプレイ解像度は **Waddledoo** のみ 1920x1080 だが、他の **Waddledee** 、**Wheelie** は 2400x1600 をサポートすると見られる。[^11]

[^11]: [bmpblk: Fix dedede screen resolution in boards.yaml (If44e3c4d) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/bmpblk/+/2193354)

## Volteer {#volteer}
*Tiger Lake* を搭載する Chromebook ボードのコードネーム。  
{{< link >}}[UPSTREAM: util/mainboard/google: add support for Volteer (I01b97dc3) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/1982618){{< /link >}}
### Tiger Lake {#tgl}
*Tiger Lake* とは GPUに [Gen12](/tags/gen12)アーキテクチャを採用し、Intel 10nm+(+)プロセスで製造されるプロセッサ。CPUアーキテクチャは、まだ明言こそされていないが、時期から *Willow Cove* アーキテクチャを採用すると見られている。  

CPU、GPU、プロセスどれも *Ice Lake* より1世代進んだものとなり、それ以外にも Thunderbolt 4、USB4にも新たに対応する。  
ただ Thunderbolt 4 と言っても、転送速度は Thunderbolt 3 から変わらないことが判明している。  
{{< link >}}[Thunderbolt 4の速度はThunderbolt 3、USB4と変わらず | Coelacanth's Dream](/posts/2020/01/12/tb4-eq-tb3-eq-usb4/){{< /link >}}
そのため Thunderbolt 4 と、Thunderbolt 3を仕様に取り込んだ USB4を一括りにしてしまっても問題ないだろう。  

モバイル向けの *Tiger Lake-U* は、2019 Intel Investor Meeting の発表スライドや、**Volteer** 、*tglrvp ボード* の Kconfig から最大 4-Core/8-Thread と見られている。[^13][^14]  

[^13]: <https://s21.q4cdn.com/600692695/files/doc_presentations/2019/05/2019-Intel-Investor-Meeting-Bryant-(May-9-update).pdf>
[^14]: <https://github.com/coreboot/coreboot/blob/a02bf7468a5bb22f47be2aaf6186f2c835710fbc/src/mainboard/intel/tglrvp/Kconfig#L48>

GPUの規模には2種類存在し、GT1 が 32EU、L3cacheバンク 4基(1536KB?)、GT2 が 96EU、L3cacheバンク 8基(3072KB?) となる。  

### Volteer 仕様 {#volteer-spec}
**Dedede** 同様に **Volteer** はリファレンスボードであり、製品に搭載される派生ボードには **Halvor** 、**Makefor** 、**Ripto** 、**Trondo** がある。  
コードネームの元ネタは Spyro？  

CPUは **Volteer** も最大 8-Thread[^18]、GPUは Geekbench 4 に残された実行結果によると TGL GT2[^19]。  
*Tiger Lake* の新機能(?)、Thunderbolt 4/USB4 はしっかり実装されている。[^15]  
*Tiger Lake-U* として特に削られた部分は無く、ハイエンド帯の Chromebook として登場するだろう。  

[^18]: <https://github.com/coreboot/coreboot/blob/a02bf7468a5bb22f47be2aaf6186f2c835710fbc/src/mainboard/google/volteer/Kconfig#L65>
[^19]: <https://browser.geekbench.com/v4/cpu/15183944.gb4>

ディスプレイ解像度は 3840x2160 をサポートする。[^12]

[^15]: [TBT/USB4: Allow enabling TBT & USB4 per port based (I07b33902) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/ec/+/2001221)
[^12]: [bmpblk: add volteer board (I7fcad704) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/bmpblk/+/1895272)

各派生ボードの違いを探したが、はっきりしたものは見つけられなかった。  

この **Volteer** だが、Geekbench 4 の実行結果が残されている。  
{{< link >}}[Google volteer - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15183944){{< /link >}}
2020/01/29 に実行したものだが、4-Coreと表示しているのに L1I/D cache、L2cacheは2基だったりとあやふやなものとなっている。  

## おまけ、 開発されていた Cannon Lake 搭載 Chromebook ボード {#cnl}
新しい世代となる10nmプロセスで転んだことによりキャンセルされてしまった *Cannon Lake* だが、それを搭載する Chromebook ボードの開発が行なわれていた。  
リファレンスボードは **Zoombini** 、その派生ボードには **Meowth** というコードネームが付けられていた。  
{{< link >}}[zoombini: Add support for new board (Ia296ddf6) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/depthcharge/+/669958){{< /link >}}
{{< link >}}[mainboard/google/zoombini/variants/meowth: add new board (Ie6ed7ebb) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/768591){{< /link >}}

*Cannon Lake* を語る上で触れられることが多い GPU部だが、アーキテクチャは Gen10、GT1 から eDRAMを搭載した GT3e まで用意されていた。  
最大規模となる GT3/e は、72EU、L3cacheバンク 12基(3072KB) という仕様であり、EU数は *Ice Lake* の 64EUよりも多かった。  
{{< link >}}<https://github.com/intel/media-driver/blob/bdbce9ce524e1aaa2a5c9770899d4762f443e0b4/media_driver/linux/gen10/ddi/media_sysinfo_g10.cpp>{{< /link >}}
しかし 10nmプロセスの歩留まりが悪く、GPUを有効にした *Cannon Lake* プロセッサが出荷されることはなかった。  
Intel も[Intel-graohics-compiler](https://github.com/intel/intel-graphics-compiler)のあるコミットで、

 > dead CNL platform

というコメントを付けている。(引用元: <cite>[Remove the dead CNL platform. · intel/intel-graphics-compiler@6e9f0af](https://github.com/intel/intel-graphics-compiler/commit/6e9f0af19184cdc3f94c32c23692eaccd3b0ef20)</cite>)  
上でコードネームは親しみをもたらす、と書いたが、その分キャンセルといった言葉ではなく、*dead* を使われると心を傷むものがある。  

4-Slice という構成から、フロントエンド部が大きく、複雑なものとなってしまったのかもしれない。  
Gen11アーキテクチャから 1-Slice とし、内部の SubSlice を増やす方向性にシフトした。  

搭載する *Cannon Lake* のCPUコア数は最大 4CPUという仕様であったらしい。  
{{< link >}}[mainboard/google/zoombini/variants/meowth: enable all cpu cores (I9bbfe496) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/838502){{< /link >}}
<del>
書き方から物理 4コアと読めるが、*Cannon Lake* で唯一出荷された **Core i3-8121U** は 2-Core/4-Thread となっている。[^16]  
一方で、Intel が開発するリファレンスボード、cannonlake\_rvp は最大 8-Threadとしている。[^17]</del>  

<del>個人的には、10nmプロセスの歩留まりが非常に悪く、*Ice Lake* 同様にダイの半分近くを占めていたであろうGPUを無効化にしても数が確保できず、  
仕方なく 4-Core中 2-Coreのみを有効にすることを仕様として何とか **Core i3-8121U** を出荷したのではないかと考えている。</del>  

{{< ins >}}

Kconfig における MAX_CPUS は SMT、HTT 等による仮想コア、スレッドを含めた値であるため、4コアではなく、2コア 4スレッドを想定していた可能性のが高いと思われます。  


{{< /ins >}}

[^16]: [Intel® Core™ i3-8121U Processor (4M Cache, up to 3.20 GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/136863/intel-core-i3-8121u-processor-4m-cache-up-to-3-20-ghz.html)
[^17]: [mainboard/intel/cannonlake_rvp: set Max CPUs and Mainboard Family · coreboot/coreboot@5f34b37](https://github.com/coreboot/coreboot/commit/5f34b37d80379eaf49e425719a6dffe58b9652fd#diff-b9d009c546f44ca8d481b1ae5aecf6cf)

結局、搭載するプロセッサがキャンセルされてしまってはどうしようもないため、**Zoombini** 、**Meowth** は開発、サポートから外された。  
{{< link >}}[UPSTREAM: zoombini: remove support for deprecated zoombini board (I015306f1) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/1389176){{< /link >}}
