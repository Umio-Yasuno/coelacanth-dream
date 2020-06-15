---
title: "Radeon RX 5500 XT発表！ & 国内は18日から発売開始"
date: 2019-12-13T05:18:40+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

スペックは以前の発表通り、11WGP（22CU）、Game Clock 1717MHz、Boost Clock 1845MHz。  
[AMD Unveils the AMD Radeon™ RX 5500 XT Graphics Card: Incredible 1080p Performance, Breath-Taking Visuals and Powerful Software Features](https://www.amd.com/en/press-releases/2019-12-12-amd-unveils-the-amd-radeon-rx-5500-xt-graphics-card-incredible-1080p)  

発表自体は2019-10-07にされていたが、"RX 5500 series"と少しぼやけた言い方だったためか、  
この二ヶ月、RX 5500 XTは12WGP（24CU）じゃないかと一部で噂されていた。  
無印とXT付きくらいしかラインナップがないのだから、ふつーにRX 5500 XTと最初から発表した方が良かったように思う。  
スペックに関しては特に変更がなかったため、AMDの目的は不明。  

無印はOEM向けということで、以前心配していた差別化の問題は、そもそも自作PC、ゲーミング市場にはXTしかでないから問題ない、というものだった。  

### 性能は概ねRX580よりちょっと上、消費電力はRX570以下
[AMD Radeon RX 5500 XT Linux Performance - Phoronix](https://www.phoronix.com/scan.php?page=article&item=amd-rx5500xt-linux)  
[MSI Radeon RX 5500 XT Gaming X 8 GB Review - TechPowerUp](https://www.techpowerup.com/review/msi-radeon-rx-5500-xt-gaming-x-8-gb/)  
[アンダー3万円ミドルの新星「Radeon RX 5500 XT 8GB版」の実力やいかに？ - ASCII.jp](https://ascii.jp/elem/000/001/993/1993096/)  
[Radeon RX 5500 XTとGeForce GTX 1650 Superを試す、2万円級の新定番を比較レビュー  - マイナビニュース](https://news.mynavi.jp/article/20191212-938753/)  

RX 580と比較すると、CUが4割ほど少なく、メモリ帯域もわずかに及ばないのにRX 5500 XTが性能で勝っているのは、  
7nmによるクロック向上とRDNAアーキテクチャでのキャッシュ増量、IPC増加が効いているのだろう。  

| AMDGPU | RX 5500 XT | RX 570 | RX 580 |
| :--- | :---: | :---: | :---: |
| CUs | 22 | 32 | 36 |
| SPs | 1408 | 2048 | 2304 |
| TMUs | 88 | 128 | 144 |
| ROPs | 32 | 32 | 32 |
||
| Base Clock (MHz) | ? | 1168 | 1257 |
| Game Clock (MHz) | 1717 | | |
| Boost Clock (MHz) | 1845 | 1244 | 1340 |
||
| Max Memory Size (GB) | 8 | 8 | 8 |
| Memory Interface (bit) | 128 | 256 | 256 |
| Memory Speed (Gbps) | 14 | 7 | 8 |
| Memory Bandwidth (GB/s) | 224 | 224 | 256 |
||
| Peak Texture Fill-Rate (GT/s) | 162.36 | 159.23 | 192.96 |
| Peak Pixel Fill-Rate (GP/s) | 59.00 | 39.81 | 42.88 |
| FP16 (TFLOPS) | 10.4 | 5.1 | 6.2 |
| FP32 (TFLOPS) | 5.2 | 5.1 | 6.2 |
| Typical Board Power (W) | 130 | 150 | 185 |
 
ただCUに伴ってTMUも少なくなっているため、そこがボトルネックになることがあるかもしれない。  

マルチメディア周りもPolarisから進んでいるため、使い勝手は性能以上にいいはずだ。  
それ以外にもGFX9（Vega）からの新機能（DSBR、Packed、NGG）も改良され、取り入れられている。<span style="color:darkslategray">是非試してみたいが、うん、まあ……。金がない。</span>  
Vegaではあまり日の目を見なかったそれらが、この世代で活かされるなればGPUとしての価値はさらに高まるだろう。  

肝心の価格はと言うと、  
アメリカでの参考価格は4GB 169ドル、8GB 199ドル、  
国内の実売価格は税込みで、4GBが2万円台半ば、8GBが3万弱といったところ。  
[Radeon RX 5500 XT搭載グラフィックスカードがASRockとMSIから登場。メモリ容量8GB版と4GB版をラインナップ](https://www.4gamer.net/games/337/G033715/20191212130/)  
思っていたよりも落ち着いていた。  

Boost ClockはRX 5700/XTの時、各社リファレンスよりも上げていたが、  
今回は律儀に?1845MHzを守っている。  
ピーク時の消費電力がTBPの130Wギリギリに近いからだろうか。  
[AMD Radeon RX 5500 XT Linux Performance Page10 - Phoronix](https://www.phoronix.com/scan.php?page=article&item=amd-rx5500xt-linux&num=10)  
<span style="color:darkslategray">邪推すれば、リネーム時に差をつけやすくするためとも考えられる。130Wというのを律儀に守るのであれば補助電源は1x 6-pinでいいはずだ。8-pinにしたのは、リネームで高いクロック高い消費電力としても、基板をほぼ使い回すためかもしれない。フル有効の12WGP（24CU）も加えれば、リネームでもきっちりと性能に差が出るだろう。  
まあ邪推。</span>

過去Navi14まとめ  
[Navi14 Matome](/posts/2019/11/04/navi14-matome/)  
[Navi14 Matome Part2](/posts/2019/11/13/navi14-matome-part2/)  
[Navi14 Matome Part3](/posts/2019/11/14/navi14-matome-part3/)  
[Navi14のちょっとしたまとめ](/posts/2019/12/06/navi14-a-little-matome/)  
