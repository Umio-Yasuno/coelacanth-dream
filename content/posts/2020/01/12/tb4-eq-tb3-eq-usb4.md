---
title: "Thunderbolt 4の速度はThunderbolt 3、USB4と変わらず"
date: 2020-01-12T15:09:32+09:00
draft: true
tags: [ "Bus", ]
keywords: [ "Thunderbolt4", "USB4", "Thunderbolt3" ]
categories: [ "Hardware", "Intel" ]
---

IntelがCES2020にてTigerlakeの新機能の1つとして出したThunderbolt 4（以下、TB4）だが、帯域はThunderbolt 3（以下、TB3）と変わらず、実質的なリブランドになる可能性が高い。  

## USB3の4倍?
IntelはNewsroomでTB4はUSB3の4倍とアナウンスしているが、問題はUSB3のどれかはっきりと言っていないことだ。[^1]  

 > and 4x the throughput of USB 3 with the new integrated Thunderbolt 4.  

[^1]:[2020 CES: Intel Brings Innovation to Life with Intelligent Tech Spanning the Cloud, Network, Edge and PC | Intel Newsroom](https://newsroom.intel.com/news-releases/intel-ces-2020/)


USB3には現在、ざっくりと帯域で言うと5 Gbit/s、10 Gbit/s、20 Gbit/sの3種類がある。（GB/sはエンコード方式によってUSB 3.2 Gen1とGen2で違い、Gen1は8b/10b、Gen2は128b/130bでGen2の方が高効率。）  

そして、TB3は40 Gbit/sであるから、Intelの言う4倍が20 Gbit/sに掛かるならば、TB4は80 Gbit/sとTB3の2倍となる。  
Thunderboltは信号にPCI Expressを使っており、TB3はPCIeGen3 x4で40 Gbit/sのスループットを実現していた。  
そこから発展させて、x4という構成を保ちながら80 Gbit/sにするにはPCIeGen4でないといけないことから、TigerlakeはPCIeGen4に対応するのではないか、と予想するメディアもいた。  

### 結果
そこの所気になった[Tom's Hardware](https://www.tomshardware.com)がIntelに確認を取ったところ、4倍というのは**USB 3.1（10 Gbit/s）**に掛かるとのことで、TB4はTB3、それとTB3を仕様に取り込んだ次世代規格であるUSB4と帯域が変わらないことが確定となった。  
[What Is Thunderbolt 4? Tiger Lake Tech Isn't Faster, Thunderbolt 3 With a New Name - Tom's Hardware](https://www.tomshardware.com/news/what-is-thunderbolt-4-tiger-lake-tech-isnt-faster-thunderbolt-3-with-a-new-name)  

### 推測
したがってTigerlakeがPCIeGen4に対応した可能性も低くなる。  
少なくともモバイル向けでPCIeGen4の必要性は薄い。先日発表されたAMD Ryzen 4000シリーズもPCIeGen4に対応しなかった。  

TigerlakeとPCH間接続がPCIeGen4 x4になる、という話が流れたこともあったが、  
[Intel "Tiger Lake" Supports PCIe Gen 4 and Features Xe Graphics, Phantom Canyon NUC Detailed - TechPowerUp](https://www.techpowerup.com/258196/intel-tiger-lake-supports-pcie-gen-4-and-features-xe-graphics-phantom-canyon-nuc-detailed)  
TB4ではなくTB3となっていることから、その時からある程度変更が入っていることは間違いないだろう。  

Tigerlakeの新機能に

 > NEXT GEN I/O TECHNOLOGY

があったりと可能性がない訳ではないが、PCIeGen4であるとは言っていないし、あるならあるで先日のCES2020の場でアピールしそうな気もする。  
[2019 Intel Investor Meeting - 2019_Intel_Investor_Meeting_Bryant.pdf - Intel](http://intelstudios.edgesuite.net/im/2019/pdf/2019_Intel_Investor_Meeting_Bryant.pdf#page=10)  
PCIeGen4にCPU側が対応しても、デバイス側としては自社製品が**まだ**ないためアピールとしては事欠けるというのが実情だったりするかもしれないが。  
[Intel® Stratix® 10 DX FPGAs - Intel® FPGAs](https://www.intel.com/content/www/us/en/products/programmable/sip/stratix-10-dx.html)は対応しているが一般向けではなく、せめてSSD等が望まれる。  
一応、Optane SSDのGen4対応は予定されており、Linuxドライバー開発用のサンプルがあるくらいには進んでいるらしい。  
[Intel has PCIe 4.0 Optane SSDs Ready, But Nothing for Customers to Plug Them Into- Tom's Hardware](https://www.tomshardware.com/news/intel-has-pcie-40-optane-ssds-ready-but-nothing-to-plug-them-in-to)  
最も、これもサーバー向けのOptane DCといった製品になると思われ、一般向けではない。  

## 感想
IntelからのTigerlakeに関しての情報はちらつかせるだけのものが多く、かえって謎が増えてるような気がしてならない。  
さすがにTB3のリブランドだけに留まらず、Intelは何らかの付加価値を作ってくるはずだが、そこに関してはまだ一切明かされていない。  

{{< ref >}}
 * [What Is Thunderbolt 4? Tiger Lake Tech Isn't Faster, Thunderbolt 3 With a New Name - Tom's Hardware](https://www.tomshardware.com/news/what-is-thunderbolt-4-tiger-lake-tech-isnt-faster-thunderbolt-3-with-a-new-name)  
 * [IntelがThunderbolt 4に言及、詳細不明ながら速度は据え置きか - Computerworldニュース：Computerworld](https://project.nikkeibp.co.jp/idg/atcl/19/00001/00076/?ST=idg-cm-hardware)  
 * [Intel's new 'Thunderbolt 4' spec quickly turns into a confusing messi - PCWorld](https://www.pcworld.com/article/3512937/intels-new-thunderbolt-4-spec-quickly-turns-into-a-mess.html)  
 * [Intel's 'new' Thunderbolt 4 is just a re-branding of Thunderbolt 3 - TweakTown](https://www.tweaktown.com/news/69845/intels-new-thunderbolt-4-re-branding-3/index.html)  
 * [Thunderbolt 4 vs. USB 4 - what's the difference? • Indeedly](https://indeedly.io/thunderbolt-4-is-usb-4-maxed-out/)  
 * [新しいUSBのスタンダード：USB 3.2の説明 - MSI](https://jp.msi.com/blog/new-usb-standard-usb-3-2-gen-1-gen2-explained)  
 * [USB 3.0#USB_3.2 - Wikipedia](https://en.wikipedia.org/wiki/USB_3.0#USB_3.2)  
{{< /ref >}}
