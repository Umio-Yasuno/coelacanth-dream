---
title: "PS5 のダイ観察"
date: 2021-02-15T23:04:59+09:00
draft: false
tags: [ "DieShot", "Navi14", "RDNA", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

[Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/) 氏により、*PS5 SoC* のダイショットが公開された。  
*Xbox Series X SoC* は HotChips 32 にて既に公開されており、そのため今まで隠されていた *PS5 SoC*  の構造には注目が集まっていた。  

{{< pindex >}}

 * [FPU の規模を小さくした PS5 CPU](#fpu)
 * [Navi14/PS5/XSX WGP比較](#compare-wgp)

{{< /pindex >}}

## FPU の規模を小さくした PS5 CPU {#fpu}

{{< figure src="/image/2021/02/15/ps5-4core-ccx.webp" title="&uarr; PS5 4-Core CCX / &darr; Renoir 4-Core CCX" caption="画像元:<br> &uarr; [DSC02283# | quick and dirty test shot | Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/50943598756/) <br> &darr; [AMD@7nm@Zen2@Renoir@Ryzen_3_4300U@100-000000085_9JB4977P00… | Flickr](https://www.flickr.com/photos/130561288@N04/50016639913/in/album-72157715067913276/)" >}}

*PS5 CPU* の 4-Core CCX と *Renoir APU* の Zen2 4-Core CCX をとりあえず縦幅を合わせ、並べた画像が上。実際の面積等は考慮していない。  

コアの外側端、*Renoir* と比較して *PS5 CPU* の面積が少なくなっている部分は、AMD が ISSCC 2020 にて発表した *Zen 2* コアの概要によると、FPU (Floating Point Unit) にあたり、[^isscc-2020]  
撮影を行った [Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/) 氏が言及したように、*PS5* では FPU の規模を小さくした設計となっている。  
恐らくは FPU のデータ幅を *Zen 2* の 256-bit から 128-bit にしたものと考えられる。FPU の演算パイプ数を減らすと各 FPU の構成やスケージューラーを変える必要が出てしまうし、コンパイラ側の最適化等も変えなければならない。  
FPU を減らしたこと自体については、FP性能が必要であれば GPU で処理すればよく、CPU側の FP性能はあまり必要無いと考えたのかもしれない。前世代の *PS4/Pro* は CU、演算ユニットの規模に対し、フロントエンドの ACE (Asynchronous Compute Engine) を多く搭載し、GPU をより汎用的に使いやすくした設計だった。  
*PS4/Pro* で採用された *Jaguar* コアもまた FPU のデータ幅は 128-bit であるため、互換性を取る上で *Zen 2* コアの FPU は無駄があると考えた可能性もある。  

*Zen 2* コアそのままではなく、面積を減らし、製造コストをより小さくしようとした工夫は、汎用機ではないゲーム機用の SoC として印象的な変更である。  

[^isscc-2020]: [Zen 2: The AMD 7nm Energy-Efficient High-Performance x86-64 Microproc…](https://www.slideshare.net/AMD/zen-2-the-amd-7nm-energyefficient-highperformance-x8664-microprocessor-core) (Page6)

同様に *Zen 2* コアを採用している *Xbox Series X* は 256-bit幅であり、そのままの構成を採っている。  

## Navi14/PS5/XSX WGP比較 {#compare-wgp}

{{< figure src="/image/2021/02/15/navi14-ps5-xsx-2wgp.webp" title="T: Navi14 (RDNA) 2-WGP / M: PS5 2-WGP / D: XSX 2-WGP" caption="画像出典:<br>T: [AMD@7nm@RDNA_1th_gen@Navi14@Radeon_RX_5500_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/49437016132/in/photolist-7qTNge-2ijw99L-2ire3nw-2isbGvV-2irHhdF-2ijzCx7-2irecZs-2irHicE-2irHgTc-2irJq44-2irJp5k-2irEBCv-2irHk7r-2irJoNJ-dzD9po-p5ovvH-aRrcZp-7pMLAX-4Vw39U-6p8mSL-2irECpR-2irECPd-2irJnTx-2irHjX3-2irEzm6-2irJrJd-2irJrCw-2irHhYD/) <br> M: [DSC02280 | quick and dirty test shot | Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/50943598871/) <br> D: [HotChips2020_GPU_Microsoft_Jeff_Andrews_v2.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Microsoft_Jeff_Andrews_v2.pdf)" >}}

*Navi14 (RDNA)* 、*PS5* 、*XSX* の対称的に配置された 2基の WGP を並べた画像が上。  
こちらは、3つともメモリに GDDR6 を採用しており、そして WGP の近くにプロセスの微細化の影響を受けにくい GDDR6 PHY (物理層) があったため、それを基準にサイズを合わせている。  
WGP 2基で比較したのは、配置によっては一部ユニットの構造や専有面積が変わることがあり、その差を減らすため。  
*Navi21 (RDNA 2)* の詳細なダイショットは無いため、あくまでも *Navi14 (RDNA)* と *RDNA 2* ベースとの比較となる。  

まず言えるのは、*PS5 2-WGP* と *XSX 2-WGP* の面積は近く、*Navi14 2-WGP* の約8割となっていることだ。  
*PS5* と *XSX* の WGP は *Navi14* のそれより若干縦に大きくなっているが、全体的にコンパクトに収められている。  
製造プロセスについては、*Navi14* は TSMC 7nm だが、*PS5* は 7nm、そして *XSX* は 7nm Enhanced という言葉を用いており、曖昧な所があるが、*PS5* と *XSX* は *Navi14* の TSMC 7nm より進んだ 7nmプロセスで製造されているとも見られる。物理設計の最適化による功績である可能性もあるが。  


面積だけでなく、中央部、TMU (Texture Mapping Unit) + RA (Ray Accelerator) 4基の構造も近いものとなっており、画像を拡大すると分かりやすい。これが *Navi21 (RDNA 2)* と同様かが一層気になる所だ。  
ただ、やはり同時期に同じ開発企業と協力して設計する以上、似通うものなのだろう。  


| 2-WGP | Pixels | vs Navi14 2-WGP |
| :-- | :--: | :--: |
| Navi14 | 514170px<br>(W: 1305, H: 394) | x1.00 |
| PS5 | 433068px<br>(W: 956, H: 453) | x0.84 |
| XSX | 414072px<br>(W: 972, H: 426) | x0.80

{{< ref >}}
 * [PlayStation 5 Launches This November At $399 For PS5 Digital Edition, And $499 For PS5 With Ultra HD Blu-ray Disc Drive | Sony Corporation of America](https://www.sony.com/content/sony/en/en_us/SCA/company-news/press-releases/sony-computer-entertainment-america-inc/2020/playstation-5-launches-this-november-at-399-for-ps5-digital-edition-and-499-for-ps5-with-ultra-hd-bluray-disc-drive.html)
 * [Xbox Series X](https://www.microsoft.com/en-us/p/xbox-series-x/8wj714n3rbtl?activetab=pivot:techspecstab)
 * [西川善司の3DGE：PS5のスペック予想はいくつ当たったか？ CPUとGPUは予想どおりだが，最も意外なのはPrimitive Shaderの採用](https://www.4gamer.net/games/990/G999027/20200421040/)
 * [HC31_1.1_AMD_ZEN2.pdf](https://old.hotchips.org/hc31/HC31_1.1_AMD_ZEN2.pdf)
 * [HotChips2020_GPU_Microsoft_Jeff_Andrews_v2.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Microsoft_Jeff_Andrews_v2.pdf)
{{< /ref >}}
