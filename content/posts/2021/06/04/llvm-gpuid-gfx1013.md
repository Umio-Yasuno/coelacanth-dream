---
title: "RDNA/GFX10.1世代に新たな GPUID、gfx1013?"
date: 2021-06-04T23:49:17+09:00
draft: false
tags: [ "RDNA", "LLVM", "GFX10" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

まだレビュー段階ではあるが、LLVM に新たな GPUID: *gfx1013* を追加するパッチが投稿された。  
投稿したのは [bcahoon (Brendon Cahoon)](https://reviews.llvm.org/p/bcahoon/) 氏であり、氏は過去にも AMDGPU に関連したパッチを投稿しており、また過去のコミットを見ればわかるが amd\\.com ドメインのメールアドレスを使用している。LLVM における AMDGPU サポートに尽力しているソフトウェアエンジニア方がレビューに参加していることもあり、私的にパッチは信頼できるものと考えている。  
 
 * [⚙ D103663 [AMDGPU] Add gfx1013 target](https://reviews.llvm.org/D103663)

## gfx1013

AMDGPU における GPUID は Major, Minor, Stepping の 3つで構成されており、*gfx1013* の場合は Major: 10, Minor: 1, Stepping: 3 となる。  
{{< link >}} [AMD GPU の GPU ID は何を意味するか | Coelacanth's Dream](/posts/2020/06/22/amdgpu-gpuid-mean/) {{< /link >}}
Major: 10, Minor: 1 (*gfx101x*) には RDNA(1) 世代、または *Navi1x* 世代とも呼べる AMD GPU に割り当てられており、*gfx1013* は新しく追加された GPUID ではあるが、*RDNA(1)/Navi1x* 世代の GPU に向けた GPUID だと考えられる。  
疑問なのは、*RDNA(1)/Navi1x* 世代の GPU には [Navi10 (gfx1010)](/tags/navi10), [Navi12 (gfx1011)](/tags/navi12), [Navi14 (gfx1012)](/tags/navi14) が存在するが、*gfx1013* が割り当てられるような AMD GPU に思い当たるものが無く、Linux Kernel や RadeonSI、RADV といったオープンソースドライバーにも現時点ではそうした AMD GPU のサポートに向けたパッチは投稿されていない。  

### Navi10ベース? {#navi10}

また、奇妙なのは *gfx1013* は *Navi10 (gfx1010)* をベースとしているらしいことだ。  
*Navi12/Navi14* ではサポートしているドット積命令 (`FeatureDot[1,2,5-7]Insts`) を *Navi10* ではサポートしていないが、*gfx1013* では *Navi10* 同様サポートしていないとされている。  
ドット積命令は推論処理や深層学習の高速化に活用されることを想定しており、*RDNA(1)/Navi1x* 世代では前述にように対応が分かれていたが、*RDNA 2/gfx103x* 世代では確認できる限りすべての GPU/APU でドット積命令をサポートしている。  
それらドット積命令はパックド実行に対応しており、(U)INT4/8/16,FP16 のデータならば FP32 で処理する場合と比べて理論上では最大2-8倍性能を出すことができる。  

 > 		def FeatureISAVersion10_1_3 : FeatureSet<
 > 		  !listconcat(FeatureGroup.GFX10_1_Bugs,
 > 		    [FeatureGFX10,
 > 		     FeatureGFX10_AEncoding,
 > 		     FeatureLDSBankCount32,
 > 		     FeatureDLInsts,
 > 		     FeatureNSAEncoding,
 > 		     FeatureWavefrontSize32,
 > 		     FeatureScalarStores,
 > 		     FeatureScalarAtomics,
 > 		     FeatureScalarFlatScratchInsts,
 > 		     FeatureGetWaveIdInst,
 > 		     FeatureMadMacF32Insts,
 > 		     FeatureDsSrc2Insts,
 > 		     FeatureLdsMisalignedBug,
 > 		     FeatureSupportsXNACK])>;
 >
 > {{< quote >}} [⚙ D103663 [AMDGPU] Add gfx1013 target](https://reviews.llvm.org/D103663) {{< /quote >}}

*gfx1013* が持つ新機能として、 `GFX10_AEncoding` をサポートしているが、現時点のパッチではどう活用するのかは示されていない。  

 > 		def FeatureGFX10_AEncoding : SubtargetFeature<"gfx10_a-encoding",
 > 		  "GFX10_AEncoding",
 > 		  "true",
 > 		  "Encoding format GFX10_A"
 > 		>;
 >
 > {{< quote >}} [⚙ D103663 [AMDGPU] Add gfx1013 target](https://reviews.llvm.org/D103663) {{< /quote >}}

ドキュメントにパッチにて追加された部分では、*gfx1013* は dGPU だとしているが、確定している情報かは不明。  
まだレビュー段階のパッチであるため今後変更される可能性は十分あり、これは他の点にも言える。  


 > 		     ``gfx1013``                 ``amdgcn``   dGPU  - cumode          - Absolute      - *rocm-amdhsa* *TBA*
 > 		                                                    - wavefrontsize64   flat          - *pal-amdhsa*
 > 		                                                    - xnack             scratch       - *pal-amdpal*
 > {{< quote >}} [⚙ D103663 [AMDGPU] Add gfx1013 target](https://reviews.llvm.org/D103663) {{< /quote >}}


まだまだ正体不明の *gfx1013* だが、今後さらなる情報が AMD のソフトウェアエンジニアよりパッチの形で公開された時はまだ記事にしたい。  

