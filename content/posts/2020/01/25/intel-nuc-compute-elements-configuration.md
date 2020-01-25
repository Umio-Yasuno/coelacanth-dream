---
title: "Intel NUC Compute Elementsの軽い解説とまとめ"
date: 2020-01-25T03:13:41+09:00
draft: false
tags: [ "Intel" ]
keywords: [ "", ]
categories: [ "Hardware", "CPU" ]
noindex: false
---

CES2020にて正式発表されたIntel NUC Compute Elementの構成が個人的に気になったため、整理ついでに一度まとめてみることにした。  
主要なソースはIntel公式の資料。  
[NUC9QN_TechProdSpec.pdf - Intel.com](https://www.intel.com/content/dam/support/us/en/documents/intel-nuc/nuc-kits/NUC9QN_TechProdSpec.pdf)

### 概要
Ghost Canyon をおおまかに言って2つのボードと電源、それとケースで構成される。  

1つは**Compute Element**で、こちらにCPU、PCHがあり、2x SODIMM memory、2x M.2 SSD(NVMe/SATA)を搭載できる。  
また、オーディオヘッダー、USBヘッダー、ファンヘッダー、パワースイッチ等のIOヘッダーもこちらにある。EPS12V 8pin 電源コネクタとパワーボタンもあることから Compute Element単体でも起動できそうだが、実際の所はどうなのだろう。  
ExtreamとProの2種類があり、違いは搭載されるCPUとそれによるECCのサポート、vProの有無となる。  
PCHは共通してCM246チップセット。  

もう1つは**BBWC1B**ベースボード。  
主にGPU等の拡張カードを差すためのボードで、Compute Elementが必要な機能を統合している分シンプルな構成をしており、Compute Element用のPCIe 3.0 x16スロット、PCIe 3.0 x4接続のM.2スロット(2242/2280/22110)、PCIe 3.0 x16スロット、PCIe 3.0 x4スロット、  
そして電源入力コネクターとPMBUS(Power Management Bus) 5pin ヘッダーが実装されている。  
M.2スロットはCompute Elementとは違い、NVMeのみ対応となる。  
後述する理由からPLXのような特別なチップを搭載している訳でもないらしい。  

電源は**FSP500-30AS**という製品名で、Flex-ATX形状、最大定格出力は500W、変換効率は80+Platinumとのこと。  

ケースは、前面にパワーボタン、UHS-II SDXCリーダー、2x USB 3.1 Gen2 Type-Aポート、3.5mm オーディオコネクターがあり、  
上面には（恐らく）92mmファンを2基搭載。  

これらをまとめて**Intel NUC 9 Extream/Pro Kit**を構成する。  

### 詳細
#### CPU
EXtreamはCore i5-9300H /i7-9750H /i9-9980HK、ProはCore i7-9850H /Xeon E2286Mのどれかが搭載されている。  
ソケットはBGAタイプであり、換装は不可能。  

| NUC 9 Extream | i5-9300H | i7-9750H | i9-9980HK |
| :--- | :---: | :---: | :---: |
| Core / Thread | 4/8 | 6/12 | 8/16 |
| Base Clock (GHz) | 2.4 | 2.6 | 2.4 |
| Max Turbo Clock (GHz) | 4.1 | 4.5 | 5.0 |

| NUC 9 Pro | i7-9850H | E2286M |
| :--- | :---: | :---: |
| Core / Thread | 6/12 | 8/16 |
| Base Clock (GHz) | 2.6 | 2.4 |
| Max Turbo Clock (GHz) | 4.6 | 5.0 |

#### PCH
上述したように、Extream/Pro共通でCM246。  
[Mobile Intel® CM246 Chipset Product Specifications - ark.intel.com](https://ark.intel.com/content/www/us/en/ark/products/135100/mobile-intel-cm246-chipset.html)  
ブロックダイアグラムは以下

{{< figure src="/image/2020/01/25/nuc9-kit-block-diagram.webp" attr="Source: [NUC9QN_TechProdSpec.pdf - Intel.com](https://www.intel.com/content/dam/support/us/en/documents/intel-nuc/nuc-kits/NUC9QN_TechProdSpec.pdf)" >}}

ark.intel.com[^1]にもあるように、Compute Element上のM.2スロットは2つともPCHからのもの。  
それとは別にSATA 6Gbpsが存在し、Compute Elementのボード上にもデータと電源のピンがまとめられたFPCスタイルのコネクターが実装されているが、ケース側にSATAドライブを収めるスペースがない。  
Compute Elementを使った別製品を計画しているのだろうか？  

[^1]:[Products formerly Ghost Canyon - ark.intel.com](https://ark.intel.com/content/www/us/en/ark/products/codename/189255/ghost-canyon.html)

#### BCWC1B
M.2スロット(NVME, PCIe3.0 x4)、PCIe3.0 x16スロット、PCIe3.0 x4スロットがあるが、Compute ElementからはPCIe3.0 x16までしか出ておらず、  
そのためPCIe3.0 x16スロットのみ使用している時はx16の帯域で動作するが、他のBCWC1B上のM.2スロット、PCIe3.0 x4スロットのどちらか1つでもx16スロットと同時に使用している場合はレーンが分配され、  
x16スロットに接続されたデバイスは、x8の帯域で動作することとなる。  
このことからPLXといったPCIeスイッチチップは搭載していないはずだ。  
基板上にもそれらしきものは見当たらない。  

電源が10pinの見慣れないコネクタだが、調べた所ATX 10pinで、OEMの小型PCで使われることがある……らしい。  
ピンアサインは各社で微妙に違うため実質独自規格なのだろうか？  
電源には詳しくない……  

最近になってIntelが提唱するATX電源コネクタの新規格、**ATX12VO**も10pinだが、やはり微妙に違い、またPCIeスロットに3.3Vを供給する必要があるため、別物なはずだ。  
[ATX12VO: Future Power Supplies will not have 24-pins ATX connector anymore, but 10-pins - guru3d.com](https://www.guru3d.com/news-story/atx12vo-future-power-supplies-will-not-have-24-pins-atx-connector-anymorebut-10-pins.html)  

#### FSP500-30AS
Compute Element用のEPS12V 8pin、直角タイプの8pinと直線タイプの6+2pinのPCIe補助電源、PMBUS 5pinのコネクターが引き出されている。   
容量としてはPCIe x16スロットと補助電源を合わせて375W(PCIe 75W + 2x8pin 300W)までのGPUが動作可能なはずだが、  
サポートは225W(PCIe 75W + 8pin 150W)までとなっている。  
<span class="hide">そりゃそうか……</span>

#### エアフロー
左側面から吸気し、上面の2x 92mmファンによって排気する仕組み。  
Compute Elementのブロアーファンも同じ方向であり、電源もケース内に吐き出す形と思われるため、まとめて排気できる。  
ただGPUの冷却はまだ良いとして、Compute Elementはまともに冷却できるか疑問が残る。  
GPUをx16スロットに差すと隙間がなくなってしまい、ブロアーファンが吸気するための余裕がだいぶなくなる。  
Compute Elementの実装面を逆にして、GPUと別方向から吸気するといった方法は取れなかったのだろうか。もしかしたらPCIe x16の配線による制限等があったのかもしれない。  
それぞれのファンの回転数は不明。  
Ghostを名に冠しているのだから静かだといいが。  

#### 感想
売れるのかな。  
価格を見ると、i5モデルで$945.02 - $947.49、i7モデルで$1104.99 - $1107.45、i9モデルで$1535.66 - $1538.13。  
dGPU無しでこれだ。<span class="hide">普通に自作すれば同額でdGPU有りの構成が組めるだろうし、選択肢も広い</span>  
それに省スペースで高性能が可能と言っても、それがどこまで実現できるかはまた別の話が気がする。  
障害になるのはやはり熱だ。  
性能が高ければ当然相応の熱が出る。そして小型にすると排熱が難しくなる。  
VegaMを搭載したSkull Canyonも同じ問題を抱えていたのではないかと思う。  
スペースはないから小型で、価格が高くてもいいから出来る限り性能が高いPCが欲しい、という人がどれだけいるだろうか。  
業務向けならばまだ需要はあるはずだとは思う。  

パーツの自由度は一般的なPCより劣るものの、これまでのNUCに比べたら格段に広いため、Intelが今後どういった新Compute Elementを出すか次第かもしれない。  
