---
title: "Renoirの情報アップデート&推測 Part3"
date: 2019-12-21T00:27:47+09:00
draft: false
tags: ["Radeon", "GCN", "Raven", "GFX9", "gfx909", "Renoir", "Dali", "Zen_2" ]
keywords: [ "Radeon", "Renoir", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

### Renoir SKU
[AMD Renoir APUs | GPU details for all SKUs with TDP | All GPU configurations for mobile & desktop - Reddit : Amd](https://www.reddit.com/r/Amd/comments/ebxsvh/amd_renoir_apus_gpu_details_for_all_skus_with_tdp/)  
Redditユーザーの[_rogame](https://www.reddit.com/user/_rogame/)氏がAMD Bootcamp Driver内の情報と過去のZen APU情報を結びつけ、SKUの詳細を考察したというもの。  
見た限り、筋は通っているように思える。  

ただ、RenoirのRevisionIDに関しては多くがGPUOpen-Toolsのcommon-src-DeviceInfoにて開示されているが、[DeviceInfoUtils.cpp](https://github.com/GPUOpen-Tools/common-src-DeviceInfo/blob/master/DeviceInfoUtils.cpp#L528)  
ShaderEngine数、CU数といった性能に関した記述は全く出ていない（Ravenから更新されていない）ため、上記の_rogame氏の考察を裏付けることはできない。  

それでもRevisionIDの傾向に沿っているため、大きく外れているという訳でもない。  

	（Xには0からFまでのどれかが入る）
			0X = Pro/WS/Mx
			4X = Apple
			8X = Embedded
			9X = Embedded (low?)
			CX = Consumer (Gaming)
			DX = Pro
			EX = Reserve? (Custom?)
			FX = Test?

AMDは他のGPUでも基本この傾向にあり、[DeviceInfoUtils.cpp](https://github.com/GPUOpen-Tools/common-src-DeviceInfo/blob/master/DeviceInfoUtils.cpp)と組み合わせるとAMDがどういったSKUを用意しているかがぼんやりと見えてくる。  

個人的には、_rogame氏の考察は概ね合っていると思うし、90%信用していいと思う。  
後述の自身の考察では合っているものとして扱う。  

### Ryzen 7 4700U
[Renoir APU | Ryzen 7 4700U 8C/8T 2GHz/4.2GHz | Pcmark10 & more - Reddit : Amd](https://www.reddit.com/r/Amd/comments/ed9qyr/renoir_apu_ryzen_7_4700u_8c8t_2ghz42ghz_pcmark10/)  
こちらも_rogame氏によるもの。  
Ryzen 7 4700Uで3DMARKを実行した結果のスクリーンショットで、_rogame氏はGPUのCU数は10/11CUだと見ている。  
そしてRyzen 7 4700Uは8C/8Tの構成であり、RenoirはRaven/Picassoの2倍のコア/スレッドが可能、ということがほぼ確定となった。  
SMTが無効化されていることから、有効にされた8C/16TがRyzen 9の枠に納まるのかもしれない。  

以前、RenoirのRyzen 9はあくまで噂としたが、それはコア数をある程度デスクトップと合わせるんじゃないか、なんて考えていたからだ。  
今にしては、AMDはモバイル向けの4C/8TにもRyzen 7を、Intelも6C/12Tにi9のブランドを飾っている。  
だから完全に思い違いで、知識不足だった。  

### 考察とか

#### CPU構成
まず間違いなくL3キャッシュ容量は減らすと思われる。  

Zen/+アーキテクチャでは、CCXごとにL3モジュール（1MB）が8基存在し、それによってL3キャッシュ 8MBとしていた。  
[Summit Ridgeは冷却性能でクロックが変動　AMD CPUロードマップ  - ASCII.jp](https://ascii.jp/elem/000/001/406/1406039/)  
<https://ascii.jp/elem/000/001/406/1406052/img.html>  
だがAPUであるRavenでは、そのL3モジュール（1MB）を4基に減らし、L3キャッシュ4MBとした。  
[AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G - Flickr](https://www.flickr.com/photos/130561288@N04/39716562275)（[Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/)によるダイショット）  
これは、SRAMは容量に対して必要とする面積が大きく、それによるリーク電流で消費電力が増えてしまうため、モバイルが主なターゲットとなるAPUでも不利となるからだと思われる。  
コスト面を考えてもダイサイズは小さい方がいい。  
参考: [プロセッサのキャッシュに不揮発性メモリを使う - PC Watch](https://pc.watch.impress.co.jp/docs/column/semicon/544296.html)  
Renoirでも同様の理由でL3モジュール減らすはずで、やるとしたらZen2 CCD（8C/16T）の32基の半分、16基となるだろうか。  

Zen2 CCDのダイショットを見る限り、モジュールあたりの容量は増やさずに、モジュールの数を2倍に増やす形でL3キャッシュ 32MBを実現しているため、  
[AMD@7nm(12nmIOD)@Zen2@Rome@EPYC_7702_ES - Flickr](https://www.flickr.com/photos/130561288@N04/49045449908/)（またまた[Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/)によるダイショット）
やろうと思えばRenoirでCCXあたりL3モジュール 4基、総8基にできそうだが、  
コアの縦の長さに合わせて配置しているため、そこまでいじろうとすると手間がかかるだろうからやらないと考えている。  

#### GPU構成
そこまで大きな変更はないはず。  
Raven（gfx902）であった、scissor機能やDCCのバグを修正をした、Raven2と同じ**gfx909**となるくらいで。  
L2キャッシュ 1MB、8 ROPという構成もそのままだろう。  

#### CPU+GPU
_rogame氏のRenoir SKUでは最大13CUとなっており、それならダイにも13CUとなりそうだが、まずRenoirにはCCXが2基ある。  
大体同じ比率として  
[AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G - Flickr](https://www.flickr.com/photos/130561288@N04/39716562275)  
を元に考えると、CCXを90度回転させて2基配置するのが効率良いのではないかと思う。  
![Renoir kuso-colla](/image/2019/12/21/renoir_guess01.webp)  

クソコラで申し訳ないが、自分が言いたいのはそうしたCCXの配置によって伸びた分が、CUを11から増やすのにも丁度いい、ということだ。  

ただ効率が良さそう、と言っても多少ダイサイズは増加する。  
これまた雑な計算だと縦に30%程増加する。  

209.78[mm<sup>2</sup>] * 1.3 = 272.714[mm<sup>2</sup>]  
GF 14/12nmからTSMC 7nmとして、当然単純に半分にならない部分もあるため、  
どんぶり勘定となるがRaven2と同程度か少し大きいくらいのダイサイズになるのではないかと思う。  
Zen2 CCDは必要だった、IFOPの物理的な接続部分がいらないため、2CCXのダイサイズはだいぶ小さくできる。  
I/O部分は、ソケットがFP5からFP6へと変更するが、上画像の空白部分を埋める程度でそう大きくは変わらないと思っている。  

Intel 10nmのIcelakeが ~131mm<sup>2</sup>程であり、それと比べると大きくはなるだろう。  
[デスクトップ向けIce Lakeの出荷は絶望的　インテル CPUロードマップ  - ASCII.jp](https://ascii.jp/elem/000/001/873/1873186/index-4.html)  

#### 電力密度とか
Renoirの不安要素に電力密度がある。  
何しろRaven2程のダイサイズにZen2 8コアとVega13CUとかが詰め込まれているのだ。  
そして_rogame氏のSKUではグレード高ければコア数もCU数も多いことになっている。  
ゲームのようなCPU/GPU両方を使うソフトだと、お互いがTDP枠を喰いあってサーマルスロットリングが発生するのは容易に想像できる。  
そうでなくともCPU/GPU共にピーク性能は抑えられそうな気がするが、以前あったGPU 1.8GHzというのが引っかかる。  
ブースト機能が悪さをしなければいいのだが。  

電力密度は大きくなり、発熱する部分の割合が増える。  
Raven/Picassoでも発熱によるサーマルスロットリングが問題となっていたが、Renoirではさらに悪化すると思われる。  
16CUが可能そうなのに、SKUでは13CUとなっているのは電力密度の問題を少しでも軽減させる目的があるかもしれない。  

電力密度の問題を軽減する意味も含めて、CPU偏重、GPU偏重、バランスといった別々の傾向を持ったSKUも可能だと思われるが、そういった話が全く無いのはSKUの複雑化を避けたのだろうか。  

### 構造推測
![Renoir Diagram](/image/2019/12/21/renoir_guess02.webp)  

推測図の更新。  

以下、表まとめ:  

| Renoir | CPU / Thread |
| :-- | :--: |
| Ryzen 9 | 8/16 ? |
| Ryzen 7 | 8/8 ? |
| Ryzen 5 | 6/12 ? |
| Ryzen 3 | 4/8 ? |

<br>

| <span style="color:crimson">RV | <span style="color:coral">Raven | <span style="color:coral">Raven2 | <span style="color:coral">Picasso | <span style="color:#f4a460">Renoir | <span style="color:#f4a460">Dali |
| :--- | :---: | :---: | :---: | :---: | :---: |
| CMOS CPU | 14nm | 14nm | 12nm | 7nm? | 12nm? |
| CPU | Zen(+) | Zen(+) | Zen+ | Zen2 | Zen+? |
| Max CPU Core/Thread | 4/8 | 2/4 | 4/8 | 8/16 ? | 2/4 ? |
| L3$ CPU | 4MB | 4MB | 4MB | 16MB? | 4MB? |
||
| CMOS GPU | 14nm | 14nm | 12nm | 7nm? | 12nm? |
| GPU | Vega (GFX9) | Vega (GFX9) | Vega (GFX9) | Vega (GFX9) | Vega (GFX9) |
| GPU Clock | 1300 MHz | 1200 MHz | 1400 MHz | 1800 MHz? | | 
| Shader Engine | 1 | 1 | 1 | 1 ? | 1 ? |
| Max CUs | 11 | 3 | 11 | 13 ? | 3 ? |
| Max TMUs | 44 | 12 | 44 | 52 ? | 12 ? |
| Max ROPs | 8 | 4 | 8 | 8 ? | 4 ? |
| L2$ GPU | 1MB | 0.5MB | 1MB | 1MB? | 0.5MB? |
| | <span style="color:coral">Raven | <span style="color:coral">Raven2 | <span style="color:coral">Picasso | <span style="color:#f4a460">Renoir | <span style="color:#f4a460">Dali |
| Memory Type | DDR4 | DDR4 | DDR4 | LP/DDR4/X | DDR4 ? |
| Support Memory Speed | 2933 MHz | 2400 MHz | 2933 MHz | 4266 MHz? | 2933 MHz? |
| VCN ver | 1.0 | 1.0 | 1.0 | 2.0 | 1.0? |
| DCN ver | 1.0 | 1.0 | 1.0 | 2.1 | 1.0 |
| DeviceID | 15DD | 15D8 /15DD | 15D8 | 1636 | 15D8 /15D9? |
| GPU ID | gfx902 | gfx909 | gfx902 | gfx909 | gfx909 |
| Die Size | 209.78 mm<sup>2</sup> | 154.68 mm<sup>2</sup>?? | 209.78 mm<sup>2</sup> | ~180mm<sup>2</sup>? | |

<hr>
<code>Renoir関連まとめ</code>

 * [Renoirの構造推測 ](/posts/2019/11/09/renoir-guess/)  
 * [Renoirの構造推測 Part2 ](/posts/2019/12/02/renoir-guess-p2/)  
