---
title: "Sapphire がファンレス RDNA GPU、GPRO X070 をリリースしていた"
date: 2021-01-15T19:49:44+09:00
draft: false
tags: [ "RDNA", "Navi10" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMD の AIB (Add-in-Board) パートナーである [Sapphire](https://www.sapphiretech.com/en) がブロックチェーン、GPUコンピューティング向けにファンレス RDNA GPU、**GPRO X070 8GB GDDR6** をリリースしていた。  

 * [GPRO X070 8GB GDDR6 GPU Compute](https://www.sapphiretech.com/en/commercial/gpro-x070-gpu-compute-graphics)

GPUスペックは **[RX 5700 XT (Navi10)](https://www.amd.com/en/products/graphics/amd-radeon-rx-5700-xt#product-specs)** をベースとしており、WGP 20基 (CU 40基)、GDDR6メモリ、メモリバス幅 256-bit、メモリ速度 14Gbps となっている。  
補助電源には 8-pin + 6-pin を必要とする。  

冷却には、ファンを搭載しないパッシブクーリングを採用している。  
さらにブロックチェーン向けであるが、映像出力が 4本実装されており (1x HDMI, 3x DP)、一般用途においても問題無く使えるカードとなっている。  
そういった市場に出すかはびみょうだが、ファンレスモデルは GPUカードの中で故障しやすい可動部であるファンを搭載していないため、信頼性は高いという点や、小型ファンがないため、静粛性に優れるという特徴がある。  
2スロットであるから普通のケースにも搭載でき、需要自体はあるように思われる。  

以前、Linux Kernel (amd-gfx) に *Navi10* のブロックチェーン向け SKU をサポートするパッチが投稿されていたが、その 1つが **GPRO X070** かどうかは不明。  
{{< link >}} [ブロックチェーン向けの Navi10 SKU とそこから読める文脈 | Coelacanth's Dream](/posts/2020/10/21/navi10-sku-for-blockchain/) {{< /link >}}
映像出力が実装されていることから、既存のカードのクーラーを換えたモデルなのかもしれない。  
ただこうしたブロックチェーン向けのモデルを Sapphire が用意したコンテクストは一緒だろう。  

TSMC の 7nmプロセスは *Zen 2/3 CPU* 、*Renoir/Lucienne/Cezanne APU* 、 *RDNA/2 GPU* 、 *MI100 GPU* 、 *PS5 SoC* 、*XSX/S SoC* 、*NVIDIA A100 GPU* ……等、多くのプロセッサが採用しており、  
それらが、パッケージに使われる材料の供給不足により生産が遅れているという話が出ている。[^tsmc]  
そんな中でビットコインを始めとした仮想通貨の価格が上昇してきた。  
ビットコインは専用ASIC によるマイニングが主流になったが、イーサリウムは ASIC耐性があることから、マイニングには GPU が使われる。  
最新の CPU/GPU、ゲーム機を求める人にとってはまだしばらく辛い状況となりそうだ。  

[^tsmc]: [ASCII.jp：Ryzen 5000Gシリーズに2つのコアが混在する理由　AMD CPUロードマップ (3/3)](https://ascii.jp/elem/000/004/039/4039743/3/) <br> [Report: Packaging Issues, PS5 Demand May Be Hurting TSMC Production - ExtremeTech](https://www.extremetech.com/computing/318937-report-packaging-issues-ps5-demand-may-be-hurting-tsmc-production)
