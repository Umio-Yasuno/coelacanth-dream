---
title: "AMD Zen APUを搭載したChromebookの話――Ryzen 7 3700C、Ryzen 5 3500C――"
date: 2020-02-09T12:31:30+09:00
draft: false
tags: [ "Picasso", "Raven2", "Dali", "Pollock" ]
keywords: [ "Chromebook", "Picasso", "Dali", "Pollock", "Ryzen", "Athlon" ]
categories: [ "Hardware", "APU" ]
noindex: false
---

ChromiumOSへのパッチから以下の新たなZen Mobile SKUが見つかった。  

 * Ryzen 7 3700C (Picasso)
 * Ryzen 5 3500C (Picasso)
 * Ryzen 3 3250C (Dali [Raven2])
 * Athlon Silver 3050C (Dali [Raven2])
 * Athlon Gold 3150C (Dali [Raven2])

[ 2040455: Rework map_oprom_vendev to add revision check and mapping —  Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/3/src/soc/amd/picasso/northbridge.c#b326)  

基本、既存のMobile向けSKUの末尾をUからCに置き換えたSKU名となるが、  
パッチ内のRevision IDを照らし合わせるとRyzen 7 3700Cのみ、Ryzen 7 3700UではなくRyzen 7 3750Hと共有のものとなっている。  
[AMDGPUのDID、RID、Productのびみょうまとめ Part2 | Coelacanth's Dream](/posts/2019/12/30/did-rid-product-matome-p2/#picasso-gfx902)  

それとDaliの一部Revision ID、さらにはPollockもSKU名こそ不明だが一部が判明し、個人的には結構嬉しい発見だった。  

### Chromebookとして
これまでChromebook向けにAMD APUはA4-9120CとA6-9220Cしか出ておらず、  
プロセスは28nm、CPUは2-Core/2-Thread、GPUは3CU、メモリは1866MHz 1ch。  
その中では上であるA6-9220CでもCPU Clock Base 1.8GHz /Boost 2.7GHz、GPU 720MHzである。  
[7th Generation AMD A4-9120C APU with Radeon™ R4 Graphics | AMD](https://www.amd.com/en/products/apu/7th-gen-a4-9120c-apu#product-specs)  
[7th Generation AMD A6-9220C APU with Radeon™ R5 Graphics | AMD](https://www.amd.com/en/products/apu/7th-gen-a6-9220c-apu)  

またAMD公式サイトにはSupported TechnologiesにVP9 Decodeがあるが、ハードウェア的には対応していないはずであり、また対応していることを示すコードが見つからなかったため、（見落としている可能性もある）  
CPUによるソフトウェアデコードが可能、という意味かも知れない。  
[RadeonFeature](https://www.x.org/wiki/RadeonFeature/#radeonuvdunifiedvideodecoderhardware)  

そういう訳で、前世代でコスト低め、TDP 6Wという点を含めても性能はかなり低めで、使い勝手もそれ相応だったと思われる。  
そしてA4-9120Cだけなため、モデル数という点でも競合に当然圧されていた。  

<br>
上記のSKUを搭載したChromebookが世に出れば、性能、モデル数の問題は解消されることとなる。  

そしてその中のPicasso Ryzen 5/7は、高性能なChromebookブランド、Pixelbookとなると考えられ、  
現状、最新のPixelbook Goでも8th Gen Intel Core m3 /i5 /i7 Y-Series (Amber Lake Y) であるから、CPU性能では同等、GPU性能では圧倒できるだろう。（同時にIntel Ice /Jasper /Elkhart /Tiger lakeあたりの10nm SoCを採用した製品が出る可能性もあるが）  


### 推測
また、Pollockもしくはパッチ内にあったSKU以外に、TDPをより小さくしたSKUが出るんじゃないかと **個人的に** 予想している。  
IntelとAMDでTDPの意味合いが違うため無視するとしても、Ryzen MobileはデフォルトTDP 15W、下限 12W、上限 25WとA4-9120C/A6-9220Cの6Wと比べると大きな差がある。  
ソケット変更があるため、熱設計に関しては深く考える必要はないのかもしれないが、最新世代でも前世代製品並の消費電力、熱設計を求める声はあるだろう。  

そして、AMDはまだ公式発表こそしていないが、より省電力なRaven2ベースの組み込み向けSKUを用意している。  

 > Alternativ können auch Prozessoren der Serie R1000 Low-Power mit den Varianten R1305G Dual Core (vier Threads), 1,5 GHz (beziehunsweise 2,8 GHz Boost) mit einer TDP von 8 W (6 bis 8 W) beziehungsweise R1102G Dual Core (zwei Threads), 1,2 GHz (beziehungsweise 2,6 GHz Boost) mit einer TDP von 6 W bestückt werden. Die integrierte Vega-Grafikprozessoreinheit mit drei bis zu 1200 MHz schnellen Compute Units steht für hohe Grafikleistung. 

 > (引用元: [Maßgeschneiderte HMI-Systemkomponenten in 90 Tagen](https://www.elektronikpraxis.vogel.de/massgeschneiderte-hmi-systemkomponenten-in-90-tagen-a-901430/))

 * R1305G: CPU 2-Core/4-Thread Base 1.5GHz /Boost 2.8GHz, GPU 3CU 1GHz, TDP 8-10W[^1]
 * R1102G: CPU 2-Core/4-Thread Base 1.2GHz /Boost 2.6Ghz, GPU 不明, TDP 6W

[^1]: [D3713-V/R mITX | Kontron](https://www.kontron.com/products/boards-and-standard-form-factors/motherboards/mini-itx/d3713-v-r-mitx.html)

R1102GはTDP 6Wと、一致し、後継には丁度いいはずだ。  

結論として、R1102Gの全く同じか近い構成のSKUをChromebook向けにも出してくるのではないかと予想している。  

そういった理屈抜きにしても最新CPUを低消費電力で動かしてみたいという気持ちがある。  
ChromebookならLinux入れるのも、ドライバー周りのハードルも低いだろう。  
<span class="hide">買うかどうかはかなり怪しいが。自分、Revison IDが判明しただけで喜べるような人間だし。</span>
