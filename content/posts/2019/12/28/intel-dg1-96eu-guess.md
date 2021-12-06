---
title: "Intel DG1は96EU構成? & 推測"
date: 2019-12-28T10:01:20+09:00
draft: true
tags: [ "Gen11", "Gen12", "DG1" ]
# keywords: [ "Intel", "Gen11", "Gen12", "DG1" ]
categories: [ "Hardware", "GPU", "Intel" ]
---

EECに登録された情報から、Intel期待<span class="hide">?</span>のdGPU、DG1のEU数が96だという話が出ている。  
[Набор разработчика DG1 External FRD1 96EU Accessory Kit (Alpha) (DGD12KEF3A) - portal.eaeunion.org](https://portal.eaeunion.org/sites/odata/_layouts/15/Portal.EEC.Registry.UI/DisplayForm.aspx?ItemId=66098&ListId=d84d16d7-2cc9-4cff-a13b-530f96889dbc)  
	<code>（情報元: <https://twitter.com/KOMACHI_ENSAKA/status/1209833522086547458>）</code>  

ただ名前からして、まだベータ版手前のアルファであり、実際に製品として出てくるDG1も同じ96EUかどうかはわからないが、  
今から設計を一部やり直してEUを増やすなんてしたら2020年中に出てくるか怪しくなるため、製品も96EUとなる可能性が高い。  

海外メディアや一部国内ブログでは、96EUというのはTiger Lake GT2と同じだ、と言われているが、それよりどちらも同じGen12LP（12.1）と言った方が正確だと思う。  
それと、Shading UnitsがこれまでのEUあたり8 Units（SIMD8）から16 Units（SIMD16）になるのでは、と言われたりもしてるが、今の所それを裏付ける記述を見つけることはできなかった。  

### 推測
#### 性能
Intel GPUのスペックを、よく使うRadeonのフォーマットに合わせた表が以下。  

| Intel GPU | Gen11 GT2 | Gen12LP (GT2) |
| :-- | :--: | :--: |
| GPU Clock | 1.1 GHz | 1.1 GHz~? |
| Slices | 1 | 1 |
| Sub-Slices | 8 | 6 |
| EUs | 64 | 96 |
| Shading Units | 512 | 768 |
| TMUs | 32 | 24 ?? |
| ROPs | 16 | 16 ? |
| GPU L3$ | 3072KB | 3072KB? |
||
| Memory Type | LPDDR4/X | LPDDR4X /LPDDR5 |
| Memory Speed | 3733 MT/s | ~6400 MT/s? |
| Memory Bandwidth | 59.7 GB/s | 102.4 GB/s |
||
| Peak Texture Fill-Rate (GT/s) | 35.2 | 26.4~? |
| Peak Pixel Fill-Rate (GP/s) | 17.6 | 17.6~? |
| FP16 (TFLOPS) | 2.2 | 3.4~? |
| FP32 (TFLOPS) | 1.1 | 1.7~? |

IntelのGen11公式資料を元にした。  
[Architecture Overview for Intel® Processor Graphics Gen11 - Intel](https://software.intel.com/en-us/download/architecture-overview-for-intel-processor-graphics-gen11)  

まずDG1の構成としては、このGen12LPから大きく変わらないのではないかと思う。  
メモリも、GDDR6を使うのではないかと噂されたりするが、LPDDR5でも128-bit（4x32-bit） GDDR5 7Gbps（112 GB/s）近くの帯域はあるため、下手に慣れてないGDDR6のメモリーコントローラーを組み込むより、実績のある（と言っても出るのはTigerlakeと近い時期になるだろうけど）LPDDR5のままのが確実ではないだろうか。  
もしGDDR6を使い、帯域をある程度合わせると、64-bit（2x32-bit） GDDR6 14Gbps（112 GB/s）になる。  
チップ数では128-bit LPDDR5、64-bit GDDR6のどちらも2枚必要とし、コスト的には、既に量産されているGDDR6の方が有利に思えるが、2020年に出てくる次世代ゲーム機（Sony PlayStation 5、Microsoft Xbox Series X）によってGDDR6の需要が大きく高まると見られている。  
[グラフィックスDRAMの価格、2020年に急上昇する可能性 - TrendForce予測 - マイナビニュース](https://news.mynavi.jp/article/20191227-947180/)  
そのためどっちが有利か、をうまく推し量れない。  

ダイサイズは、Gen11 GT2を統合したIce Lakeが131mm<sup>2</sup>程と見られ、ここからCPUとImagingユニットを抜き、その他マルチメディア部やThunderbolt 3、メモリコントローラーを残し、GPUを96EUに増やしても100mm<sup>2</sup>弱に収まるのではないかと思う。  
[デスクトップ向けIce Lakeの出荷は絶望的　インテル CPUロードマップ - ASCII.jp](https://ascii.jp/elem/000/001/873/1873186/index-4.html)  
メモリインターフェイスは、PHYがダイの外周を広げて中をスカスカにさせないためにも、多分2PHY（2x LPDDR5/2x GDDR6）、攻めて4PHY（4x LPDDR5/4x GDDR6）あたりだと想像している。  

実グラフィック性能としてはメモリ帯域やROPからして、既存の製品だとGT1030やRX 550に近い性能になると思われる。  
Tigerlakeと比較した性能では、当然DG1の方が上になるはずだ。  
統合GPUだと、どうしてもCPUとのメモリ帯域やTDPの食い合いが避けられない。dGPUならリングバスからも解放されるだろう。  

#### 製品として
IntelのdGPU市場再参入に持ち込む武器がDG1、というのはなんとも心許ないように思う。  

マルチGPUのサポートが追加されており、複数のdGPUだけでなくiGPUとdGPUとの組み合わせにも期待でき、（かつてのATI Hybrid Graphicsみたく）  
[[Intel-gfx] [CI] drm/i915/pmu: Support multiple GPUs](https://lists.freedesktop.org/archives/intel-gfx/2019-October/216404.html)  
Intel Platformとして売り出すことが考えられるが、それでどこまで他社GPUと差を詰められるか。  
規模としては同じなため、Gen12 GT2単体とGT2+DG1との比較で、良くて1.3〜1.5倍ではないだろうか。GTX1050、RX 560に届かないくらい。  

それとも、モバイル向けにGPUが統合されていないCPUを出す予定があり、それと組み合わせることを想定しているか。  
AMDのRenoirは最大8-Coreになるとされているが、Tigerlake-U/Yは今の所4-Coreという情報しか出ていない。  
[2019 Intel Investor Meeting - 2019_Intel_Investor_Meeting_Bryant.pdf](http://intelstudios.edgesuite.net/im/2019/pdf/2019_Intel_Investor_Meeting_Bryant.pdf#page=11)  

 > 4) 5W AML 2+2 vs 9W TGL 4+2 projections

現行のモバイル向けCPUがComet Lake（6C/12T）、Ice Lake（4C/8T）の2種類があるように、Tiger Lakeの世代でも4Cより上は14++nmでSkylakeアーキ<span class="hide">（別?）</span>で、前世代になるComet Lakeとの差別化のため、GPUを取り除き、その分CPUコアを増やすということも考えられる。  

GPUというよりはマルチメディア用途で打ち出す可能性もある。  
Gen12ではさらにエンコーダー/デコーダーを強化すると見られ、上記スライドでも8k60pエンコード対応を示しており、[media-driver](https://github.com/intel/media-driver)にはHEVC 12-bit、16Kにも対応することを匂わせる記述がある。  
ただ、だいぶ前から2020年にIntelのdGPUが出る！と言いながらマルチメディア向けです、なんてのはないだろうから、微妙な説。  
なまじIntel自身が優秀なマルチメディア機能を持ったGPUをCPUに統合して売り広めただけに、そこまで需要があるとは思えない。  

また、DG1はGen12を試し、ソフトウェアベンダ―による最適化を促す立ち位置にあり、そう間を置かずにもっと高性能になるであろうDG2が出る可能性だってあるかもしれない。  


とにもかくにも公式発表、実物が待ち遠しい。  
CES2020で具体的な姿が見られるといいのだが。  

### おまけ
Intel、AMDの現行、次世代の統合GPUを比較した表。  

| Integral GPU | Intel Gen11 GT2 | Intel Gen12LP | AMD Picasso (3700U) | AMD Renoir (13CU) |
| :-- | :--: | :--: | :--: | :--: |
| GPU Clock | 1.1 GHz | 1.1 GHz~? | 1.4 GHz | 1.8 GHz? |
| Shading Units | 512 | 768 | 640 | 832 ? |
| TMUs | 32 | 24 ?? | 40 | 52 ? |
| ROPs | 16 | 16 ? | 8 | 8 |
| GPU $ | 3MB | 3MB? | 1MB | 1MB? | 
||
| Memory Type | LPDDR4/X | LPDDR4X /LPDDR5 | DDR4 | LPDDR4/X |
| Memory Speed | 3733 MT/s | ~6400 MT/s? | 2400 MT/s | 4266 MT/s |
| Memory Bandwidth | 59.7 GB/s | 102.4 GB/s | 38.4 GB/s | 68.3 GB/s | 
||
| Peak Texture Fill-Rate (GT/s) | 35.2 | 26.4~? | 56.0 | 93.6 ? |
| Peak Pixel Fill-Rate (GP/s) | 17.6 | 17.6~? | 11.2 | 14.4 ? |
| FP16 (TFLOPS) | 2.2 | 3.4~? | 3.6 | 6.0 ? |
| FP32 (TFLOPS) | 1.1 | 1.7~? | 1.8 | 3.0 ? |
