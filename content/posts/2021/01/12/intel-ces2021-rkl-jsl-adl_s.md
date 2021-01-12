---
title: "Intel、Rocket Lake、Tiger Lake-H、Jasper Lake など新世代プロセッサを正式発表"
date: 2021-01-12T16:30:17+09:00
draft: true
tags: [ "Rocket_Lake", "Jasper_Lake", "Alder_Lake" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

今月 11日よりオンラインで開催されている CES2021 に合わせ、Intel は 3つの News Conference を開催した。  
その中の「Do More with the Power of Computing」にて、新世代のプロセッサとなる *Rocket Lake-S* のさらなる発表、*Jasper Lake* と *Tiger Lake-H* の正式発表、そして *Alder Lake-S* の動作デモが披露された。  

 * [CES 2021: Intel Announces Four New Processor Families | Intel Newsroom](https://newsroom.intel.com/news-releases/ces-2021-intel-announces-four-new-processor-families/)

{{< pindex >}}

 * [Rocket Lake](#rkl)
 * [Jasper Lake](#jsl)
 * [Alder Lake](#adl)
    * [Alder Lake-S は PCIe Gen5 16-Lane を備えるか](#adl_s-gen5-x16)

{{< /pindex >}}

## Rocket Lake {#rkl}

*Rocket Lake* の概要は既に発表されていたが、IPC向上について具体的な数字が出され、そしてダイの画像が公開された。  
{{< link >}} [Intel Rocket Lake-S の概要を発表　―― Cypress Cove の詳細はまだ、AV1 HWエンコードには非対応 | Coelacanth's Dream](/posts/2020/10/30/intel-rkl/) {{< /link >}}

*Rocket Lake-S* はその前世代の *Comet Lake (Skylake アーキテクチャ)* と比較して、19% の IPC向上を果たしたとしている。  
*Ice Lake* では 18% の向上とされていたが、1%の違いはさらなる最適化かそれとも計算方法の違いか。[^icl-ipc]  
ちなみに、この前世代から 19% の IPC向上 というのは、AMD *Zen 3 アーキテクチャ* と同じ向上幅だったりする。[^zen3-ipc]  

[^icl-ipc]: [【後藤弘茂のWeekly海外ニュース】Intelが10nmプロセスの「Ice Lake」を正式発表 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1187002.html)
[^zen3-ipc]: [【笠原一輝のユビキタス情報局】Zen 3とRocket LakeでさらにヒートアップするAMD vs Intel - PC Watch](https://pc.watch.impress.co.jp/docs/column/ubiq/1281903.html)

今回 *Rocket Lake-S* の画像が公開されたことで、ずっと前より噂されていた MCM構造ではなく、CPU と GPU を統合したモノリシック構造であることが判明した。  
CPU には 10nm で製造される *Ice Lake* を、GPU には 10nm SuperFin で製造される *Tiger Lake* と同じ *{{< xe >}} アーキテクチャ* をそれぞれバックポートした異色の構成となる。  

ダイの画像は、以下リンク先の PDF 19ページから確認できる。  

 * [PowerPoint Presentation - 2021_CES_PressConference_FINAL_1.11.2021.pdf](https://newsroom.intel.com/wp-content/uploads/sites/11/2021/01/2021_CES_PressConference_FINAL_1.11.2021.pdf)

## Jasper Lake {#jsl}

教育市場向けにコードネーム *Jasper Lake* をベースとする **Pentium Silver / Celeron** SKU 計 6個が正式発表された。  
*Jasper Lake* は CPUマイクロアーキテクチャに *Tremont* を採用し、Intel 10nm で製造される。  
ここでの 10nm は *Tiger Lake* 等の SuperFin ではない、*Ice Lake* と同じ 10nm となる。  

 * [Products formerly Jasper Lake](https://ark.intel.com/content/www/us/en/ark/products/codename/128823/jasper-lake.html)

| Jasper Lake | Core | CPU<br>base/boost clock | GPU EU | GPU<br>base/boost clock | TDP |
| :-- | :--: | :--: | :--: | :--: | :--: |
| N6005 | 4 | 2.0/3.3 GHz | 32 EU | 400/900 MHz | 10 W |
| N6000 | 4 | 1.1/3.3 GHz | 32 EU | 350/850 MHz | 6 W |
| N5105 | 4 | 2.0/2.9 GHz | 24 EU | 450/800 MHz | 10 W |
| N5100 | 4 | 1.1/2.8 GHz | 24 EU | 350/800 MHz | 6 W |
| N4505 | 2 | 2.0/2.9 GHz | 16 EU | 450/750 MHz | 10 W |
| N4500 | 2 | 1.1/2.8 GHz | 16 EU | 350/750 MHz | 6 W |


*Jasper Lake* のメモリインターフェイスは DDR4/LPDDR4x 128-bit に対応する。ただ LPDDR4x であってもメモリ速度は 2933MHz までの対応となる。  
他 I/O には、USB 2.0 8ポート、USB 3.2 5Gbps 4ポート、USB 3.2 2ポート、SATA 6.0Gb/s 最大 2ポート、PCIe Gen3 8-Lane を備える。  
ただし、SATA 2ポートと USB 3.2 5Gbps は PCIe Gen3 の一部と排他関係にある。  

少し意外だったのは、*Jasper Lake* は PCH を搭載する点で、「Intel® Pentium® Silver and Intel® Celeron® Processors」 によると、PCH は 14nmプロセスで製造され、CPU と同じパッケージ上に搭載される。USB や PCIe 等は PCH から接続するようだ。  
I/O も 1つのダイに収めず、CPU と PCH に分離したのは 10nmプロセスの製造能力と関係しているのだろうか？  

{{< figure src="/image/2021/01/12/jsl-diagram.webp" title="Jasper Lake Diagram" caption="画像出典: [Intel® Pentium® Silver N6005 Processor (1.5M Cache, up to 3.30 GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/212327/intel-pentium-silver-n6005-processor-1-5m-cache-up-to-3-30-ghz.html)" >}}

CPUダイ自体は *Elkhart Lake* と同一で、PCH とパッケージを変えている可能性も考えられるが、*Elkhart Lake* と *Jasper Lake* で GNA (Gaussian mixture model and Neural network Acceleration unit) の世代が異なり、  
*Elkhart Lake* は *Ice Lake* や *Gemini Lake* と同じ Gen1 だが、*Jasper Lake* は *Tiger Lake* と同じ Gen2 となる。  

Intel サイトのスペックシートには、1.5M Cache と Cache 4 MB の両方が記載されているが、これは L2キャッシュ 1.5MB、L3キャッシュ (LLC) 4MB の意で、*Tremont アーキテクチャ* では 4コアとそれらで共有する L2キャッシュでクラスタ (タイルとも呼ばれる) を構成する。  
L2キャッシュは CPUコアのみが使用可能だが、L3キャッシュは GPU からも使用できるという違いがある。  
{{< link >}} [気になる Jasper Lake のキャッシュ構成　―― 【追記】 Snow Ridge と Elkhart Lake について訂正 | Coelacanth's Dream](/posts/2020/10/24/intel-jsl-cache/) {{< /link >}}
同じく *Tremont アーキテクチャ* を採用する、サーバー向けの *Snow Ridge* 、組み込み向けの *Elkhart Lake* は搭載製品に応じて LLC の有効を選択可能となっていたが、一般モバイル向けとなる *Jasper Lake* は性能向上の目的もあり、デフォルトで有効化されるということなのだろう。  

前世代の Atomプロセッサ、*Gemini Lake* との性能比較も公開されている。全ての項目で性能向上を果たしており、中でもグラフィクス性能が目覚ましく、3DMark Fire Strike で 78% の向上を見せている。  
ただ TDP を示す PL1 は同じ 6W なのに対し、**Pentium Silver N6000** の PL2 は 20W と、比較対象の *Gemini Lake* より 5W 高いのは少し気になる所かもしれない。  

 * [Intel® Pentium® Silver Mobile Processors - 1 - ID:615781 | Performance Index](https://edc.intel.com/content/www/us/en/products/performance/benchmarks/intel-pentium-silver-processors/)

## Alder Lake-S {#adl_s}

*Alder Lake* が 2021年後半に登場することを再び語ると同時に、Windows OS の動作デモを披露し、開発が順調であることがアピールされた。  
何となく、以前 10nmキャンセルの話に Intel が速攻で反応してから、次世代プロセッサの開発状況に対し若干オープンになったように思う。そういった気質なのかもしれないが、Raja Koduri 氏は Xe GPU のパッケージを Twitter で公開したりしている。  

そして、*Alder Lake* が 10nm Enhanced SuperFin で製造されることが明言された。  
10nm Enhanced SuperFin ではさらなるトランジスタの高速化、MIMキャパシタの改良が為されるとしている。  

### Alder Lake-S は PCIe Gen5 16-Lane を備えるか {#adl_s-gen5-x16}

今回の発表とは少しずれるが *Alder Lake-S* に関係した話として、PCIeレーンが *Alder Lake-P* のそれより増やされるというものがある。  

先日、Coreboot のあるパッチに対するコメントで明かされた、*Alder Lake-P* は PCIe Gen5 8-Lane と PCIe Gen4 4-Lane x2 を備えるということを取り上げたが、  
{{< link >}} [Alder Lake は PCIe Gen5 に対応、Tiger Lake-H は Gen4 20-Lane を備える | Coelacanth's Dream](/posts/2021/01/01/adl_p-gen5-tgl_h-gen4-x20/) {{< /link >}}
関係したパッチのコメントにて、*Alder Lake-S* ではさらに `PEG11` (PCI Express Graphics Interface) が追加されることが述べられた。  

 > ADL-S has one extra PEG11, will add ADL-S later in year
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/49136/comment/ef70c9d3_664b5002/> {{< /quote >}}

PEG と PCIe RP (Root Port) の関係は、以下の引用部から、PCIe Gen4 4-Lane x2 には PEG60 と PEG62 が割り振られ、PCIe Gen5 8-Lane は PEG10 となっている。  
割り振られた数字から追加される PEG11 は PCIe Gen5 に対応し、*Alder Lake-S* はデスクトップ向けとして GPU との接続を目的に PCIe Gen5 16-Lane を備えることが考えられる。  

 > 00:01.0 (pcie5) has 1 x8 port (pcie5 or lower)  
 > 00:06.0 (pci4_0) has 1 x4 port (pcie4 or lower)  
 > 00:06.2 (pci4_1) has 1 x4 port (pcie4 or lower)  
 > {{< quote >}} [soc/intel/alderlake: Refactor PCIE port config (I0b390e43) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/48340) {{< /quote >}}

 > RP1: PEG60 : 0:6:0 : CPU SSD1  
 > RP2: PEG10 : 0:1:0 : x8 CPU Slot  
 > RP3: PEG62 : 0:6:2 : CPU SSD2  
 >
 > {{< quote >}} [soc/intel/alderlake: Refactor SoC code to maintain CPU and PCH PCIE RPs (I92123450) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/49136) {{< /quote >}}

  

*Rocket Lake-S* では利用可能な CPU側の PCIe が 20-Lane に増やされ、GPU (16-Lane) と NVMe SSD (4-Lane) とを直接接続できるようになったが、  
*Alder Lake-S* ではさらに増えて 24-Lane となり、NVMe SSD をさらに搭載可能となる。  

ただ 10nm Enhanced SuperFin でこの PCIeレーン数となると、ダイサイズへの影響がそれなりにあるだろうことが気掛かりだ。  

| | Tiger Lake | Tiger Lake-H | Alder Lake-P | Alder Lake-M | Alder Lake-S |
| :-- | :--: | :--: | :--: | :--: | :--: |
| CPU side PCIe | Gen4<br>4-Lane x2 | Gen4<br> 16-Lane + 4-Lane | Gen5 8-Lane<br>Gen4 4-Lane x2 | ? | Gen5 8-Lane x2<br>Gen4 4-Lane x2 |
| PCH | TGP_LP | TGP_H | ADP_P | TGP_LP? | ADP_S |

