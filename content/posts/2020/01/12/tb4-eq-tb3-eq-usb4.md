---
title: "Thunderbolt 4の帯域はThunderbolt 3、USB4と変わらず"
date: 2020-01-12T15:09:32+09:00
draft: false
tags: [ "Bus", ]
keywords: [ "Thunderbolt4", "USB4", "Thunderbolt3" ]
categories: [ "Hardware", "Intel" ]
---

IntelがCES2020にてTigerlakeの新機能の1つとして出したThunderbolt 4（以下、TB4）だが、帯域はThunderbolt 3（以下、TB3）と変わらず、実質的なリブランドになる可能性が高い。  

### USB3の4倍?
IntelはNewsroomでTB4はUSB3の4倍とアナウンスしているが、問題はUSB3のどれかはっきりと言っていないことだ。[^1]  

 > and 4x the throughput of USB 3 with the new integrated Thunderbolt 4.  

[^1]:[2020 CES: Intel Brings Innovation to Life with Intelligent Tech Spanning the Cloud, Network, Edge and PC | Intel Newsroom](https://newsroom.intel.com/news-releases/intel-ces-2020/)

<br>
USB3には現在、ざっくりと帯域で言うと5 Gbit/s、10 Gbit/s、20 Gbit/sの3種類がある。（GB/sはエンコード方式によってUSB 3.2 Gen1とGen2で違い、Gen1は8b/10b、Gen2は128b/130bでGen2の方が高効率。）  

そして、TB3は40 Gbit/sであるから、Intelの言う4倍が20 Gbit/sに掛かるならば、TB4は80 Gbit/sとTB3の2倍となる。  
Thunderboltは信号にPCI Expressを使っており、TB3はPCIeGen3 x4で40 Gbit/sのスループットを実現していた。  
そこから発展させて、x4という構成を保ちながら80 Gbit/sにするにはPCIeGen4でないといけないことから、TigerlakeはPCIeGen4に対応するのではないか、と予想するメディアもいた。  

#### 結果
そこの所気になった[Tom's Hardware](https://www.tomshardware.com)がIntelに確認を取ったところ、4倍というのは**USB 3.1（10 Gbit/s）**に掛かるとのことで、TB4はTB3、それとTB3を仕様に取り込んだ次世代規格であるUSB4と帯域が変わらないことが確定となった。  
[What Is Thunderbolt 4? Tiger Lake Tech Isn't Faster, Thunderbolt 3 With a New Name - Tom's Hardware](https://www.tomshardware.com/news/what-is-thunderbolt-4-tiger-lake-tech-isnt-faster-thunderbolt-3-with-a-new-name)  

したがってTigerlakeがPCIeGen4に対応した可能性も低くなる。  
さらに言えば、モバイル向けにPCIeGen4が来る日も遠くなった。USB4の対応にPCIeGen4である必要はなく、それ以上の速度となる外部バスもしばらく（1,2年）は来ないだろうからだ。  
来てもそれがモバイル向けに求められるかは怪しく、先日発表されたAMD Ryzen 4000シリーズもPCIeGen4に対応しなかった。  

#### どうなんの
さすがにIntelはリブランドだけに留まらず、何らかの付加価値を作り、アピールしてくるはずだが、そこに関してはまだ一切明かされていない。  

<hr>
<code>参考</code>

 * [IntelがThunderbolt 4に言及、詳細不明ながら速度は据え置きか - Computerworldニュース：Computerworld](https://project.nikkeibp.co.jp/idg/atcl/19/00001/00076/?ST=idg-cm-hardware)  
 * [Intel's new 'Thunderbolt 4' spec quickly turns into a confusing messi - PCWorld](https://www.pcworld.com/article/3512937/intels-new-thunderbolt-4-spec-quickly-turns-into-a-mess.html)  
 * [新しいUSBのスタンダード：USB 3.2の説明 - MSI](https://jp.msi.com/blog/new-usb-standard-usb-3-2-gen-1-gen2-explained)  
 * [USB 3.0#USB_3.2 - Wikipedia](https://en.wikipedia.org/wiki/USB_3.0#USB_3.2)  
