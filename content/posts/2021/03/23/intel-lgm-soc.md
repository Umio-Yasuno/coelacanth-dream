---
title: "Intel Lightning Mountain を追う"
date: 2021-03-23T00:00:00+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

2019/08頃に、Intel の新しいネットワークプロセッサ SoC *Lightning Mountain* をサポートするパッチが Linux Kernel に投稿され始めた。[^lgm-first-patch]  
*Lightning Mountain* は、Intel 14nmプロセスにおける第一世代 Atomアーキテクチャ *Airmont* を採用する SoC とされる。そのため、Linux Kernel上では新登場する SoC であっても、その中身は特別新しくはない。  
しかし *Lightning Mountain* の特別興味を惹く点は、最初の 2019/08頃のパッチ以降もポツポツと新たなパッチが投稿されてはいるが、積極的にサポートをアップストリームに統合させようとする様子は見られず、  
さらには既存のパッチについても十分にメンテナンスが行われているとは言い難く、SUSE のソフトウェアエンジニアである [Pavel Machek](https://archive.fosdem.org/2007/schedule/speakers/pavel+machek.html) 氏より、*Lightning Mountain* の LEDデバイスドライバーのビルドが失敗することが 2021/03/09 に報告された。[^lgm-led-fail]  
そして、[Adam Borowski](https://pl.linkedin.com/in/adam-borowski-bba80363) 氏が聞いたところでは、*Lightning Mountain* を含む Intel のホームゲートウェイ向けプロセッサの部門は [MaxLinear](https://www.maxlinear.com/)社に売却されたらしく、元のパッチを投稿したエンジニアも移ったはずだが、その MaxLinearの連絡先を知らないとのこと。  
MaxLinear社による Intel ホームゲートウェイ部門の買収は 2020/04/06 に行われている。[^maxlinear-acquire]  
これら一連の流れについては [Phoronix](https://www.phoronix.com/) の記事が詳しい。  

 * [Intel's Lightning Mountain Appears Punted Off Or Canned As Part Of MaxLinear Acquisition - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Lightning-Mountain-Code-2021)

その後、MaxLinear社は *Lightning Mountain* のコードをメンテナンスしていく意思を見せたとのことだが、新しいようでそうではなく、サポートもそう積極的に行われていない *Lightning Mountain* に自分は興味を惹かれ、一度記事にまとめるに至った。  

[^maxlinear-acquire]: [MaxLinear to acquire Intel’s Home Gateway Platform Division - MaxLinear](https://www.maxlinear.com/Company/press-releases/2020/MaxLinear-to-acquire-Intel%E2%80%99s-Home-Gateway-Platform)
[^lgm-led-fail]: [LKML: Pavel Machek: Intel, please maintain your drivers was Re: [PATCH] leds: lgm: fix gpiolib dependency](https://lkml.org/lkml/2021/3/9/1006)
[^lgm-first-patch]: [[PATCH v2 0/3] serial: lantiq: Update driver to support new SoC - Rahul Tanwar](https://lore.kernel.org/linux-serial/cover.1565257887.git.rahul.tanwar@linux.intel.com/) / [[PATCH v2 1/3] serial: lantiq: Use proper DT compatible string - Rahul Tanwar](https://lore.kernel.org/linux-serial/57e2b69e9fbd93328a477b4c7dd2dcc78784ecb1.1565257887.git.rahul.tanwar@linux.intel.com/)

<!--
 * [Search · lgm OR "Lightning Mountain" OR "LightningMountain"](https://github.com/torvalds/linux/search?o=asc&q=lgm+OR+%22Lightning+Mountain%22+OR+%22LightningMountain%22&s=committer-date&type=commits)
-->

{{< pindex >}}
 * [2017年に登場してた Lightning Mountain](#lgm-2017)
    * [SC9853I と SC9861G](#sc9853i-9816g)
    * [今はカーラジオに採用されて販売中](#car-radio)
{{< /pindex >}}

## 2017年に登場してた Lightning Mountain {#lgm-2017}

ホームゲートウェイ部門の売却と関係してか、Intel のサイトで *Lightning Mountain* を検索しても情報は得られなかった。  
そのため、投稿されたパッチから追うことにする。  

{{< ins datetime="2021/03/27" >}}

Intelのサイトがアップデートされ、1件だけだが資料が出てくるようになった。  
しかしパブリックには設定されておらず、アクセスは制限される。  

 * <https://www.intel.com/content/www/us/en/search.html?ws=recent#q=%22Lightning%20Mountain%22&t=All>

{{< /ins >}}

 >        #define INTEL_FAM6_ATOM_AIRMONT		0x4C /* Cherry Trail, Braswell */
 >        #define INTEL_FAM6_ATOM_AIRMONT_MID	0x5A /* Moorefield */
 >        #define INTEL_FAM6_ATOM_AIRMONT_NP	0x75 /* Lightning Mountain */
 >
 > {{< quote >}} [linux/intel-family.h at e68061375f792af245fefbc13e3e078fa92d3539 · torvalds/linux](https://github.com/torvalds/linux/blob/e68061375f792af245fefbc13e3e078fa92d3539/arch/x86/include/asm/intel-family.h#L115) {{< /quote >}}

*Lightning Mountain* の x86_Model は [linux/intel-family.h](https://github.com/torvalds/linux/blob/e68061375f792af245fefbc13e3e078fa92d3539/arch/x86/include/asm/intel-family.h#L115) に記述されており、CPU の識別情報は `Family: 0x6(6), Model: 0x75(117)` となる。この部分のパッチは 2019/09/05 に投稿されていた。[^lgm-model]  
バリアントとしては `AIRMONT_NP` であり、NP は *Network Processor* の意と思われる。[^lgm-np]  
これが *Lightning Mountain* を追う上で重要な手掛かりとなる。  

[^lgm-model]: [[PATCH 3/4] x86/cpu: Add new Airmont variant to Intel family - Tony Luck](https://lore.kernel.org/lkml/20190905193020.14707-4-tony.luck@intel.com/)
[^lgm-np]: [clk: intel: Add CGU clock driver for a new SoC · torvalds/linux@d058fd9](https://github.com/torvalds/linux/commit/d058fd9e8984cd9f18564f7fec38e07ce671c8b8)

特定のプロセッサ情報を探す際に有用な方法として、`site:browser.geekbench.com "Family <familyN> Model <modelN>"` を検索エンジンに入力して検索するというものがある。  
これによってそのプロセッサの Geekbench実行結果を効率良く探すことができる。  
Geekbenchの検索機能でもいいが、それだと Family, Model の値がベンチマークスコアにも引っ掛かりノイズが増える。また、Geekbench は複数のバージョンに分かれ、CPUベンチマークと GPU向けのコンピュートベンチマークとがそれぞれ存在するため、外部の検索エンジンを利用した方がまとめて探せて楽だろう。  
`family: 0x<familyN> model: 0x<modelN>` で検索し、Linux Kernel のブートログ (`dmesg`) を探す方法もある。  
前者の Geekbench は Family, Model を 10進数で、Linux Kernel は 16進数で表現しているため、そこは注意が必要となる。  

そして前者の方法で検索したところ、以下の Geekbench実行結果が見付かった。  

 * [LEAGOO T5c - Geekbench Browser - September 21st 2017, 8:38pm](https://browser.geekbench.com/v4/cpu/4109751)

### SC9853I と SC9861G {#sc9853i-9816g}

ここではプロセッサ名は **Spreadtrum IA** とだけあるが、その後にアップロードされた他の実行結果では **Spreadtrum SC9853I** と表示されている。[^lgm-gb4] また、**SC9853I** は GPU に Mali T820 MP2 を搭載している。  
**Spreadtrum SC9853I** を搭載する **Leagoo T5c** スマートフォンは 2017/12頃にリリースされた *らしい* のだが[^t5c]、公式サイトにも情報が無く、それ以上はっきりしたことは分からない。  
2018年の 2月頃には購入者に届き始め、日本語で書かれたレビュー記事も検索すればいくつかヒットする。  

Spreadtrum は中国に本拠地を置く、モバイル向けチップセットを開発するファブレス企業。2018/06/13 に RDA と新ブランド UNISOC に統合されている。[^unisoc]  

**Leagoo T5c** が **SC9853I (Lightning Mountain)** を搭載した初のスマートフォンとのことで、他にも **SC9853I** を採用した製品がいくつか見付かった。  
Panasonic からインド市場向けに **ELUGA Ray 800** [^ray-800]、タイの True Move H からは **True SMART 4G P1 Prime** [^4g-p1-prime]、中国の KONKA からは **KONKA 711S** [^711s]が発売されている。  
ただこれらが **Leagoo T5c** と違うのは、Intel 8コア x86-64プロセッサということがアピールされていないどころか隠されている点で、まともに製品ページが今も存在する Panasonic **ELUGA Ray 800** でもプロセッサ名は記載せず、オクタコアプロセッサとあるだけだ。  


[^lgm-gb4]: [LEAGOO T5c - Geekbench Browser - May 27th 2018, 9:05am](https://browser.geekbench.com/v4/cpu/8417421)
[^t5c]: [Leagoo T5c Smartphone Full Specification And Features](https://www.gizmochina.com/product/leagoo-t5c-2/)
[^ray-800]: [Panasonic Eluga Ray 800 Smartphone - Overview, Reviews, Features & Specs](https://mobile.panasonic.com/in/smartphones/eluga-ray-800) / [PANASONIC Eluga_Ray_800 - Geekbench Browser](https://browser.geekbench.com/v5/cpu/1445043)
[^4g-p1-prime]: [true SMART 4G P1 Prime - Geekbench Browser](https://browser.geekbench.com/v4/cpu/16009588) / [タイでTrue SMART 4G P1 Prime発売、5.45インチ・デュアルカメラ搭載のスマートフォン | phablet.jp (ファブレット.jp)](https://phablet.jp/?p=36202)
[^711s]: [KONKA 711S-JS - Geekbench Browser](https://browser.geekbench.com/v4/cpu/15141602) / [ROM KONKA 711S | [Official] add the 01/20/2018 on Needrom](https://www.needrom.com/download/konka-711s/)

Intel と Spreadtrum の提携は 2017/02/27 にニュースリリースが出されており、Mobile World Congress (MWC) 2017 にて、Intel 14nmプロセスで製造され *Atom Airmontアーキテクチャ* コアを 8コア搭載、そして LTEモデムを統合した **SC9861G-IA** が発表されている。[^mwc]  
製品ターゲットには、グローバルなミッド、プレミアムレベルのスマートフォンを挙げている。  
**SC9861G** は Spreadtrum側のプレスリリースで 2GHz動作することがアピールされており、先に書いた **SC9853I** は最大 1.87GHz 動作である。  
しかし、**SC9853I** は少なくはあるが採用製品が世に出ているのに対し、**SC9861G** は採用製品の情報が無い。  
一応 Geekbench に **SC9816G** らしき実行結果が 2つあり、片方はプレスリリース通り 2GHz (2.03GHz) で動作しているが、プロセッサ名に正式な SKU名だろう **SC9816G** が無い。マザーボード側にそれらしきものがあるにはあるが、末尾が G ではなく E になっている。  
また、Stepping: 8 と **SC9853I** (Stepping: 10) よりも若干前のプロセッサである。  

 * [SPRD sp9861e_1h10 - Geekbench Browser - August 17th 2017, 11:34am](https://browser.geekbench.com/v4/cpu/3724869)

そのため、何らかの理由で **SC9816G** をキャンセル、代わりに最大動作クロックを 1.87GHzに設定した **SC9853I** を立て、ターゲットもコストパフォーマンスを重視したエントリー付近に変えたのではないかと考えられる。**Leagoo T5c** に続く **SC9853I** 採用製品でプロセッサが強調されていないことと関係しているかもしれない。  
だが、**SC9816G** 発表時点では GPU に *PowerVR GT7200* を採用するとのことだったが、**SC9853I** は *Mali-T820 MP2* を採用している。  
そのため、クロックの変更だけではなく大きな仕様変更が途中行われた可能性も考えられる。  

[^mwc]: [Spreadtrum launches 14nm 8-core 64-bit mid- and high-end LTE SoC platform - Spreadtrum Communications, Inc.](http://web.archive.org/web/20170301092811/www.spreadtrum.com/en/show_news.html?id=fe766282-e9cc-4ffd-b211-e3d19675d3f7) / [Intel Demonstrates 5G Power, Versatility with AT&T; Introduces New Mobile SoC with Spreadtrum](https://newsroom.intel.com/news/intel-demonstrates-5g-power-versatility-with-att-introduces-mobile-soc-with-spreadtrum/)
[^unisoc]: [SpreadtrumとRDAの合併が完了しUNISOC誕生 – OSAKANA TAROのメモ帳](https://blog.osakana.net/archives/8745) / [Unigroup Spreadtrum & RDA Launches Its Integrated New Brand - UNISOC](https://www.prnewswire.com/news-releases/unigroup-spreadtrum--rda-launches-its-integrated-new-brand---unisoc-300665467.html)

## 今はカーラジオに採用されて販売中 {#car-radio}

調べていて気付いたのは、スマートフォン以外に **SC9853I (Lightning Mountain)** を搭載したマシンによる実行結果が、比較的最近にアップロードされていることだ。つい一週間前 (2021/03/20) に実行されたものもある。  

 * [SPRD sp9853i_1h10_vmm - Geekbench Browser - March 20th 2021, 7:36pm](https://browser.geekbench.com/v5/cpu/7036360)

上のモデル名 *SPRD sp9853i_1h10_vmm* で検索を掛けたところ GPS Power というフォーラムで、「Android搭載カーラジオで Tomtom (というアプリ) を実行することは出来ないか」といった内容の書き込みが見付かった。そのカーラジオに使われているボードが *SPRD sp9853i_1h10_vmm* となる。[^gpupower]  

そこからさらに進めると、AliExpress に並ぶ **SC9853I** を搭載した ARKRIGHT製のカーラジオに行き着く。  


 * [2 din 9 Inch Universal Car Radio GPS Navigation Android 8.1 SC9853 Octa Core Car Radio Stereo DSP 4G Sim Card Multimedia Player|Car Multimedia Player| - AliExpress](https://www.aliexpress.com/item/4000420840058.html)

カーラジオは *Lightning Mountain SoC* らしく 4Gモジュールが組み込まれていることを売りの一つにしている。  
AliExpress で **SC9853I** を検索すると、カーラジオを 13種程出てくる。それらの主な違いは、ディスプレイサイズとインターフェイス、メモリサイズ、ストレージサイズとなる。  


また、他の *Airmontアーキテクチャ* を採用する Atomプロセッサが LPDDR3メモリまでの対応だったのに対し、*Lightning Mountain SoC* は LPDDR4メモリにも対応している。  
Atom系プロセッサでは、*Airmont* から 1つ世代が進んだ *Goldmontアーキテクチャ* を採用する *Apollo Lake* から対応している。*Apollo Lake* の発売開始時期は 2016Q3 とあるので[^apl]、時期を考えれば自然かもしれないが、アーキテクチャとの差異を考えると興味深い点ではある。  

ただ、自分が AliExpress に慣れていないというのもあるが、一連のカーラジオのリリース時期、いつから販売しているのかを知ることは出来なかった。  


[^gpupower]: [Tomtom apk - broken in car radio with android](https://www.gpspower.net/tomtom-android/362748-tomtom-apk-broken-car-radio-android.html)
[^apl]: [製品の開発コード名 Apollo Lake](https://ark.intel.com/content/www/jp/ja/ark/products/codename/80644/apollo-lake.html)

<br>
ここまででいくつかの *Lightning Mountain SoC* が過去にスマートフォン向けに 2017年登場、採用され、今はカーラジオが販売されていることまでは分かったが、切っ掛けである最初のパッチが何故 2019/08頃に投稿されたのかは不明なままだ。  
Android は Linux Kernel を採用しているため、搭載製品に使われているソースコードを入手できれば何かしら分かるのではと期待したが、サポートページを探しても {{< comple >}} または製品、サポートページ自体が {{< /comple >}} 見付からなかった。  
*Lightning Mountain SoC* 自体よりも、その扱いが一番の謎と言えるかもしれない。  

中国市場は独特の流れを持つらしく、いつかに *Xbox One SoC* を転用した *AMD Cato APU* 搭載製品が中国市場で発売されたことがあった。  
{{< link >}} [A9-9820 は Xbox One SoC と同一ダイ | Coelacanth's Dream](/posts/2020/10/14/a9-9820-silicon/) {{< /link >}}
*Lightning Mountain* もまた、{{< comple >}} 不遇に思える点はあるが {{< /comple >}} そうした魅力を持つプロセッサの 1つだろう。  

{{< ref >}}
 * [Intel's Lightning Mountain Appears Punted Off Or Canned As Part Of MaxLinear Acquisition - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Lightning-Mountain-Code-2021)
 * [Spreadtrum launches 14nm 8-core 64-bit mid- and high-end LTE SoC platform - Spreadtrum Communications, Inc.](http://web.archive.org/web/20170301092811/www.spreadtrum.com/en/show_news.html?id=fe766282-e9cc-4ffd-b211-e3d19675d3f7)
 * [Intel Demonstrates 5G Power, Versatility with AT&T; Introduces New Mobile SoC with Spreadtrum](https://newsroom.intel.com/news/intel-demonstrates-5g-power-versatility-with-att-introduces-mobile-soc-with-spreadtrum/)
 * [Intel Airmont採用の中華SoC Spreadtrum SC9853i採用のLeagoo T5cを使ってみた – OSAKANA TAROのメモ帳](https://blog.osakana.net/archives/8561)
 * [MaxLinear to acquire Intel’s Home Gateway Platform Division - MaxLinear](https://www.maxlinear.com/Company/press-releases/2020/MaxLinear-to-acquire-Intel%E2%80%99s-Home-Gateway-Platform)
 * [電脳あれこれ : 【レビュー】LEAGOO T5c](http://dennouarekore.blog.jp/archives/23088978.html)
 * [【Leagoo T5c】実機レビュー この端末を買ってはいけない理由 | ガジェット23(gad23.com)](https://gad23.com/leagoo-t5s-donotbuy/)
 * [タイでTrue SMART 4G P1 Prime発売、5.45インチ・デュアルカメラ搭載のスマートフォン | phablet.jp (ファブレット.jp)](https://phablet.jp/?p=36202)
 * [パナソニック、インド市場向け 4000mAh バッテリー搭載 5.5インチスマートフォン「ELUGA Ray 800」発表 | GPad](https://gpad.tv/phone/panasonic-eluga-ray-800/)
{{< /ref >}}
