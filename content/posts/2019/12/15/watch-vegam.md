---
title: "AMD VegaMのダイ観察"
date: 2019-12-15T11:24:34+09:00
draft: false
tags: [ "Radeon", "GCN", "VegaM", "GFX8", "gfx804", "Kabylake-G", "DieShot" ]
keywords: [ "Radeon", "VegaM" ]
categories: [ "Hardware", "GPU", "AMD" ]
---

VegaMのダイを観察、ブロックの分析をしてみた。  
![VegaM Diagram](/image/2019/12/15/vegam-diagram.webp)  

元画像は[AMD@14nm@GCN_4th_gen@Polaris_22@Kabylake-G_i7-8705G_Radeon_RX_Vega_M_GL@C735B010_SR3RK___DSCx3_polysilicon_layer@IR_Makro](https://www.flickr.com/photos/130561288@N04/32173677657/)より。  
[Fritzchens Fritz - Flickr](https://www.flickr.com/photos/130561288@N04/)氏は、他にも多数のCPU/GPUのダイをパッケージから引き剥がし、高クオリティな撮影をし、パブリック・ドメインとして公開してくださっている。  
こういったダイの詳細な画像を公式が公開することは稀で、多くは省略されたブロック図に留まる。  
そのため、とても価値のある画像であり、称賛されるべき行為と言える。  

Fritzchens Fritz 氏に敬意を示すとともに、私が編集を行なった画像もCC0、パブリック・ドメインとする。  

分析には[Diagram - Polaris10 | Sashleycat Sthuff](https://www.sashleycat.com/diagram-polaris-10)を参考とした。  
Sashleycat 氏はCPU/GPUの情報、分析を発信しており、集約された情報はデータベースとして洗練されている。  
サイトのデザイン、技術力も自サイトと比べたら段違いだ。  

### 解説的なの
あくまで様々なソースから得た各ブロックの数を元に、それっぽいものを当てはめただけだ。  
ちょっとは確度があると思っているが、AMD公式の人物が解説でもしない限り100%ではない。  

PCIe x8、24CU、6 Displayであることは公式の発表から。  
[Intel、Radeon RX Vega M搭載の第8世代Coreプロセッサの仕様を公開 - マイナビニュース](https://news.mynavi.jp/article/20180108-568850/)  
[拡大画像 006l - マイナビニュース](https://news.mynavi.jp/photo/article/20180108-568850/images/006l.jpg)  
I$ K$はパターンからで、GCNでは最大4CUで共有する構成ができ、VegaMでは3CUで共有するようだ。  

Shader Engine、Render Backendの数は[ac_gpu_info.c#L964 - Mesa/mesa ・ Gitlab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/common/ac_gpu_info.c#L964)より。  
Polaris10とそれらを比較すると、(RB=Render Backend)

| GFX8 | VegaM | Polaris10 |
| :--- | :---: | :---: |
| Shader Engine | 4 | 4 |
| CU | 24 | 36 |
| TMU | 96 | 144 |
| RB | 16 | 8 |

CUこそ2/3の数だが、Render Backend数が倍あり、コンピュート性能よりもグラフィックを取ったと見える。  
ただCUには4基のTexture Mapping Unitが含まれるため、若干アンバランスな気はする。  
実際の製品で比較すると、  
Peak Texture Fill-Rate (GT/s) が、VegaM GHは114.24 (GT/s)、RX 580は172.80 (GT/s)、  
Peak Pixel Fill-Rate (GP/s)が、76.16 (GP/s)、38.40 (GP/s)。  

TMU:RB の比率で言えば、Vega64が19:1、RX 5700 XTが10:1、VegaMが6:1。  
他のAMDGPUのバランスから外れているが、それは転用されることのないグラフィック重視のカスタム品らしいと言える。  
ダイサイズは、VegaMが207.69mm2、Polaris10が243.78mm2、とそのグラフィック重視な構成はコスト削減にも貢献している。  

フロントエンド部はある程度同一のブロックは判別できそうだが、同数のもの（Geometry Processor、ACE）があるため、これと判断することは一旦諦める。  

以前から疑ってはいたが、やはりVegaMもL2キャッシュを2MB(= Polaris10と同数)持っているように見える。  
HBM2を使い、Render Backendを16基搭載しておきながらL2キャッシュを今更ケチるとも思えない。  



今後もこんな感じで画像をいじっていきたいが、いちいち文章を考えるのは面倒くさいため、簡易なデータベース的なものになると思う。  

<hr>
関連

 * [VegaMの正体](/posts/2019/11/06/vegam-identity/)
