---
title: "ROCmソフトウェアが Sienna Cichlid (gfx1030) サポートに向けて動き始める"
date: 2020-10-09T07:14:45+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2", "ROCm" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMD が開発する HPC や GPUコンピューティングのオープンソースプラットフォーム、[ROCm](https://github.com/RadeonOpenCompute/ROCm) を構成する一部ソフトウェアが *RDNA 2 /GFX10.3* GPU、*Sienna Cichlid (gfx1030)* のサポートに向けて動き始めた。  

現時点では、ROCmプラットフォームにおいて疎行列計算を行なうためのインターフェイス [rocSPARSE](https://github.com/ROCmSoftwarePlatform/rocSPARSE) と機械学習ライブラリ [MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen) に初期的なサポートが追加されている。  
{{< link >}} [gfx1030 enablement (#113) · ROCmSoftwarePlatform/rocSPARSE@1b00b6c](https://github.com/ROCmSoftwarePlatform/rocSPARSE/commit/1b00b6c8a8dd7f3a846c311594747bfc15791be7) {{< /link >}}
{{< link >}} [Update device_name.hpp · ROCmSoftwarePlatform/MIOpen@e309e99](https://github.com/ROCmSoftwarePlatform/MIOpen/commit/e309e99ff77da5b27f2e9f0545e39943365da0af) {{< /link >}}

rocSPARSE では、新たに Wave32 への対応も追加されている。  
Wave32 には *RDNA /GFX10* 世代の GPU *Navi10/Navi12/Navi14* から実装されているが、これら GPU のサポートは rocSPARSE や MIOpen 等の ROCmソフトウェアには追加されておらず、飛ばしての *RDNA 2 /GFX10.3* 、 *Sienna Cichlid* のサポートとなる。  
*RDNA /GFX10* GPU に関しては過去に、それらの公式サポートが ROCm に追加されるか、という質問に対し、公式なサポートはサーバー向け GPU **Radeon Instinct** を用いて検証が行なわれるとコメントされている。[^rocm-gfx10-comment]  
*RDNA /GFX10* をベースとしたサーバー向け GPU がリリースされていない、また今後リリースする計画もない *だろう* ことからサポートを飛ばされたものと思われる。  

[^rocm-gfx10-comment]: <https://github.com/RadeonOpenCompute/ROCm/issues/887#issuecomment-605711172>

ならば、早い段階で *Sienna Cichlid* のサポートが追加されたことは、それをベースとしたサーバー向け GPU の可能性を示している。  
*Sienna Cichlid* は Linux Kernel へのパッチから、デコード担当とエンコード担当で分かれる 2基の VCN3 を持つことや[^dual-vcn3]、XGMI によるマルチGPU[^xgmi] や ECCで検出されたメモリエラーを収集する RAS機能をサポートすることが分かっている。[^ras]  
これらはサーバー向け GPU として有用な機能と言え、*Sienna Cichlid* ではゲーミング向けだけでなく、サーバー向けも想定して設計されたと考えられるだろう。  

[^dual-vcn3]: [AMD の次世代GPU、Sienna Cichlid の Linux Kernelパッチが投稿される | Coelacanth's Dream](/posts/2020/06/02/amd-sienna_cichlid/#vcn3)
[^xgmi]: [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/)
[^ras]: [Navi21 /Sienna Cichlid のメモリは GDDR6 か HBM2 か、新たに出てきたパッチを読む | Coelacanth's Dream](/posts/2020/07/27/what-sienna_cichlid-hbm2/)
