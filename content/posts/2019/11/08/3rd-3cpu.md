---
title: "3950X & 3000G & 3rd Threadripper "
date: 2019-11-08T01:20:23+09:00
draft: false
tags: [ "Ryzen", "Threadripper", "Athlon" ]
keywords: [ "Ryzen", "Threadripper", "Athlon" ]
categories: [ "Hardware", "AMD", "CPU", "APU" ]
---

既に各メディアが報じてるいるため、詳細は省く。  
ThreadRipperは32C/64Tまで、3950Xと一緒に2019-11-25発売、  
Athlon 3000Gは2019-11-19発売、くらい。  

さて、個人的に気になるのが3rd ThreadRipperのIODとTRX40チップセットの中身だが、  
とりあえず後者はスペックから推測してX570(cIOD 12nm)だと思う。  

| | ---TRX40--- | ---X570--- |
|:---|:---:|:---:|
|PCIe| Gen4 | Gen4 |
|Chipset Uplink| x8 | x4 |
|PCIe Lanes| x8 | x8 |
|Flexible Lanes| x8 | x8 |
|Total PCIe Lanes| x24 | x20 |
|SATA 6Gbps Ports| 4 | 4 |
|USB 2.0 Ports| 4 | 4 |
|USB 3.2 Gen 2 Ports| 8 | 8 |
<br>
このように大体一致する。  
少し飛ばして下の表を見てもらえるとわかるが、cIODには24レーンあるため、X570では使わなかった4レーンをTRX40ではCPUとの接続に追加したのだろう。  
ちらと聞いた話ではこの4レーンを追加するために後方互換性を失ったとか。  
保ったところで、チップセット側でNVMe RAIDをした時に帯域が不足してるのは目に見えているため正しい判断ではあったのだろう。  
問題はユーザーがそれを受け入れるか否かだが。  

一応CPU側からのI/Oもまとめておく。  

| | ---3rd TR--- | ---3rd RY--- |
|:---|:---:|:---:|
|PCIe| Gen4 | Gen4 |
|Chipset Downlink| x8 | x4 |
|PCIe Lanes| x48 | x16 |
|Flexible Lanes| x8 | x4 |
|Total PCIe Lanes| x64 | x24 |
|USB 3.2 Gen 2 Ports| 4 | 4 |  
(ニュースサイトの44レーンという数字に合わせたが、  
[AMD Introduces World’s Most Powerful 16-core Consumer Desktop Processor, the AMD Ryzen™ 9 3950X](https://www.amd.com/en/press-releases/2019-11-07-amd-introduces-world-s-most-powerful-16-core-consumer-desktop-processor)  
一番下にある表のX570の High-Speed Platform Lanes の40レーンというのと、  
[chipsets-am4 - AMD](https://www.amd.com/en/products/chipsets-am4)  
この画像を見るに、  
[第3世代RyzenとNAVIで追加された新機能　AMD CPU/GPUロードマップ - ASCII.jp](https://ascii.jp/elem/000/001/886/1886862/img.html)  
48レーン（内8レーンはチップセットとの接続）かもしれない。  
確証はないが。  
X570 M/BでCPUからx4 NVMeを2つ引き出してるものを見つけられなかったため、  
私の思い違いか仕様変更か単にチップセットから引き出す方が簡単なのか)  
<br>
そして前者のIODだが、  
AMD公式の動画を見た所、大きさからしてIODはRomeと同じsIODと思われる。  
![3rd-tr-package](/image/2019/11/08/3rd-tr-package.webp)  
<https://youtu.be/JFd-UodssUc>  

専用設計にしなかったのは、追加の設計コストがsIODのトランジスタ数 8.34B、416mm2の生産コストを上回ったのだろうか。  
一部の不良のあるsIODを救済するという面もあるかもしれない。  
一番の理由は64C/128Tの製品を出すつもりがあるからかもしれないが。  
64Core製品がThreadripperの枠であることは、ARCTICのクーラーからわかっている。  
[Updated: ARCTIC Freezer 50 Packaging mentions 64-core Threadripper support - Guru3d](https://www.guru3d.com/news-story/arctic-launches-freezer-50-tr-argb-cpu-cooler-for-ryzen-threadripper.html)  
ただ公式サイトを確認した所、その表記は見られなくなっており、32Coresとだけ記述されている。  
[Freezer 50 TR - ARCTIC](https://www2.arctic.ac/freezer50tr/specifications/)  

ちなみにRomeの構造はこのようになっている。  
![Rome](/image/2019/11/08/rome-diagram.webp)
[The Path to Zen2 - SlideShare](https://www.slideshare.net/AMD/the-path-to-zen-2)  

2つのCCDがまとめて接続されてるように見えるが、そうではないらしい。  
[AMD 第2世代EPYCプロセッサの実態を紐解く - 構造・性能・普及状況【Deep Dive】 - マイナビニュース](https://news.mynavi.jp/article/20190819-879251/)  

ベンチマーク解禁がいつとなるかはわからないが、買う気も予算もさらさらない私はCascadelake-Xとの対戦をゆったり待つとする。  
