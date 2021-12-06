---
title: "Zen2のメモリ帯域"
date: 2019-11-10T00:08:05+09:00
draft: false
tags: [ "Ryzen", "Zen_2" ]
keywords: [ "Ryzen", "Zen2" ]
categories: [ "Hardware", "AMD", "CPU" ]
---

今更な気もするけどZen2のメモリ帯域についてのまとめ。  

### Ryzen Matisse
まず、Zen/+ではCCXとデータファブリックを32B/Cycleで、  
メモリコントローラーとデータファブリックも32B/Cで接続していたが、  
![zen-diagram](/image/2019/11/10/pinnacle-diagram.webp)
[AMD Ryzen Processor Software Optimisation](https://gpuopen.com/gdc-presentations/2019/gdc-2019-s2-amd-ryzen-processor-software-optimization.pdf)  

Zen2ではCPUダイ（CCD）とI/Oダイ（cIOD）の２つに分離、  
そしてCCD（2CCX）とデータファブリック（cIOD）をRead 32B/C、Write 16B/Cで、  
メモリコントローラーとデータファブリックを32B/Cで接続している。  
[AMD 第2世代EPYCプロセッサの実態を紐解く - 構造・性能・普及状況【Deep Dive】 - マイナビニュース](https://news.mynavi.jp/photo/article/20190819-879251/images/photo03l.jpg)  
このことにより、第三世代Ryzenでは1CCDの製品（例: 3600、3800X等）だとメモリ帯域ベンチマークにおいてWriteが約半分になっていた。  

[AMD Ryzen 3900X and 3700X (Zen2) Review (Page 3) - TweakTown](https://www.tweaktown.com/reviews/9051/amd-ryzen-3900x-3700x-zen2-review/index3.html)  
TweakTownがAMDに質問した所、省電力目的でそうしたとのこと。  
ほとんどのベンチマークにおいて3700Xが2700Xに勝っているため、AMDのアプローチは間違ってはいないだろう。  
一般用途でそこまでメモリ帯域に支配される処理があるとも思いにくい。  

第三世代Ryzen発売当時、資料では32B/Cとしか書かれておらず、また、AMDの回答を載せたのもTweakTownだけだった（たぶん）ことから、  
一部でAIDA64やUEFI等のソフトウェアによる不具合で半分になってるんじゃないかとか言われていた。  

### EPYC Rome
メモリ帯域に制限がかかるのは第二世代EPYC、Romeの一部製品でも見られる。  

> Performance optimized to 4 channels with 2667 MHz speed DIMMs for the following models: AMD EPYC™ 7282, AMD EPYC™ 7272, AMD EPYC™ 7252, and AMD EPYC™ 7232P processors.

とあるように、これら以外の製品の Per Socket Mem BW が 204.80 GB/s（8ch 3200MHz）なのに対し、  
これらは 85.30 GB/s（4ch 2667MHz相当）と記載されており、  
上記のことから4CCDの構成を取っていると考えられそうだが、  
ServerTheHomeのレビュー記事内のlstopo出力結果画像と、  
[AMD EPYC 7232P Review Hard to Buy but Solid Part - ServerTheHome](https://www.servethehome.com/amd-epyc-7232p-review-hard-to-buy-but-solid-part/)  
[AMD EPYC 7232P In GB NVMe Server Topology - ServerTheHome](https://www.servethehome.com/amd-epyc-7232p-review-hard-to-buy-but-solid-part/amd-epyc-7232p-in-gb-nvme-server-topology/)  
これらのL3キャッシュサイズが64MBまでのことから、  
2CCDの構成にし、メモリ帯域をReadで計算しているのだろう。  

ちなみにL3キャッシュサイズが128/192MBの製品は、4/6CCDの構成となっているはずだ。  
メモリ帯域はReadによるものなので、最低4CCDあれば8ch分は満たせる。  
96MB 3CCDの構成がないのは、sIOD内のデータファブリックが2つあり、それぞれのCCD数は同数（=対称）でないといけない、といった制限があるのだろうか。  

### Threadripper CastlePeak
第三世代Threadripperは今の所4CCD構成のみなため、Write帯域にも制限は掛からない。  
4CCDでも16Cは可能なはずだが、モデルナンバー的に出すつもりはないのだろうか。  

### おまけ
公式では直ったと思ってたら、GIGABYTEの[CPU Support](https://www.gigabyte.com/Motherboard/TRX40-AORUS-MASTER-rev-10/support#support-cpu)では間違ったままだった。  
![spec-wrong-gigabyte](/image/2019/11/11/spec-wrong-gigabyte.webp)  
