---
title: "VegaMの正体"
date: 2019-11-06T13:47:00+09:00
draft: false
tags: [ "Radeon", "GCN", "VegaM", "GFX8", "gfx804", "Kabylake-G", "Vega12", "gfx904" ]
keywords: [ "Radeon", "VegaM" ]
categories: [ "Hardware", "GPU", "AMD" ]
---

先日受注終了が発表されたKabylake-GのGPU部分、VegaMについてのまとめ。  
[Intel CPUとAMD GPUを統合した「Kaby Lake G」、2020年1月に受注打ち切りへ-マイナビニュース](https://news.mynavi.jp/article/20191008-906612/)  
[IntelとAMDの異色コラボが実現した「Kaby Lake-G」、生産終了へ-PC Watch](https://pc.watch.impress.co.jp/docs/news/1211584.html)  

### Kabylake-G
発表されたのは2017-11のことで、あのIntelとAMDが組んだということで話題として結構盛り上がった。  
[歴史的融合。IntelがAMD GPU内蔵Coreプロセッサを発表-PC Watch](https://pc.watch.impress.co.jp/docs/news/1090107.html)  
そしてその2社のチップを採用しているAppleがMacBookPro等に使うのではないか、という推測が少なからずあった。  
実際に製品として出たのは2018-5。  
NUC 「Hades Canyon」は小型でありながらゲーム、コンテンツ作成、VRも可能と謳われた。  
GTX 1060 Max-Qより少し上の性能があったことから、それは間違ってはいないだろう。  
だがそれを小型筐体に収めたのは排熱や電源周りで若干の無理があった気もする。  
[Kaby Lake+Radeonで本格的にゲームを楽しめる高性能NUC「Hades Canyon」-PC Watch](https://pc.watch.impress.co.jp/docs/column/hothot/1126359.html)  

### VegaMの中身
最初にはっきり言うとVegaではない。  
[Radeon Graphics/Compute Hardware](https://www.x.org/wiki/RadeonFeature/#index6h2)  
コアもマルチメディア機能もPolaris1xと同世代だ。  
Polaris10(RX 480)と並べた詳細な構成は以下、  

||---VegaM---|---Polaris10---
|:--|:--:|:--:
|CMOS|14nm|14nm|
|GPU|GFX8|GFX8|
|ShaderEngine|4|4|
|Max CUs|24|36|
|Max SPs|1536|2304|
|Max TMUs|96|144|
|Max ROPs|64|32|
|L2$|1MB?|2MB|
|Memory Type|HBM2|GDDR5|
|Memory Interface|1024-bit|256-bit|
|Memory Bandwidth|204.8GB/s|256GB/s|
|DCE|v11.2|v11.2|
|UVD|v6.3|v6.3|
|VCE|v3.4|v3.4|  

参考: [TechPowerUP-GPU Database-AMD Polaris 22](https://www.techpowerup.com/gpu-specs/amd-polaris-22.g821)  
(TechPowerUPからもVegaではなくPolarisとはっきり認識されてるらしい。)
CU数に対してROPがかなり多いが、ダイサイズを抑えつつグラフィック性能においてメモリ帯域を活かすには必要だったのだろう。  
L2cacheはTechPowerUPの記載に従ったが、どのようにして求めているのか不明であり、  
自分でやろうにも実物がない。  
ただROP数とHBM2の粒度を考えると2MBはあってもいいような気がする。  

### VegaMの悲劇?
Appleが採用すると思われてたが、そっちはそっちでVega12が出てきてしまった。  
中身も正真正銘Vega(GFX9)である。  

| |Vega12|
|:--|:--:|
|CMOS|14nm|
|GPU|GFX9|
|ShaderEngine|4|
|Max CUs|20|
|Max SPs|1280|
|Max TMUs|80|
|Max ROPs|32|
|L2$|2MB|
|Memory Type|HBM2|
|Memory Interface|1024-bit|
|DCE|v12.0|
|UVD|v7.0|
|VCE|v4.0|  

ROPは32となったが、VegaということでFP16のPacked実行にも対応しているはずだ。  
コンピュート性能も欲しいAppleとしてはこちらの方が良かったのだろう。  
この頃からeGPUをサポートし始めたと記憶しており、それなら無理に性能の高いGPUを載せる必要はあまりない。  

それと、このVega12を使った製品には、  

* Radeon Pro Vega 16
* Radeon Pro Vega 20

があり、これまたChipNameにVega20(7nm)がありややこしかった。  
Vegaの名前のややこしさに関してはモバイル向けを含めるとキリがないため、これ以上はやめておく。  

### 推測
Polarisに近いチップがVegaと同時期に出たあたり、チップ以外の部分で開発が遅れたように思う。  
実際パッケージングが大変だったらしい。  
[Hot Chips 30 - AMDのGPUを搭載したIntelプロセサ-マイナビニュース](https://news.mynavi.jp/article/20180925-695503/)  

EMIBとHBM2との接続は、後にFPGAにも使っているため、Intelにとって有益な実験製品だったことは間違いないだろう。  
[Intel、業界初のHBM2搭載FPGA「Stratix 10 MX FPGA」-PC Watch](https://pc.watch.impress.co.jp/docs/news/1097692.html)  
AMDとしても話を持ちかけられたのは財政的に厳しかった頃だろうから、カスタムチップ事業で金が入ることは悪くなかったはずだ。  
IntelとAMDとして見るとwin-winだが、そうなるとVegaMの悲壮さが増すように思うのは私だけだろうか。  

### おまけ
<del>VegaMのMが何の略かと言うと、
![vegam-name](/image/2019/11/06/vegam-name.webp)  
と Mobile らしい。  
モバイル向けAPUの製品名にもVega x Mobileが使われているためややこしい。  
そう思ったから後の発表ではMobileという文字を使わず、VegaMだけで通したのもしれない。  
それにしてもVegaは名前がややこしくなる運命にでもあるのだろうか。 </del>  
Vega MobileはVega12のことで、VegaMはVegaMということらしい。  
[年内にVegaの延長となる12LPのGPUをリリース　AMD GPUロードマップ - ASCII.jp](https://ascii.jp/elem/000/001/620/1620179/)  
やっぱややこしいって。  