---
title: "AMD GPUOpen、Zen 2 CPU、RDNA GPU へのソフトウェア最適化資料を公開"
date: 2020-07-01T17:40:02+09:00
draft: false
tags: [ "Zen_2", "RDNA" ]
keywords: [ "", ]
categories: [ "Hardware", "CPU", "GPU" ]
noindex: false
---

AMD GPUOpen は *Zen 2 アーキテクチャ* 、 *RDNA アーキテクチャ* へ向けたソフトウェア最適化のための資料を公開した。  
{{< link >}}[Let's build... 2020 - GPUOpen](https://gpuopen.com/lets-build/){{< /link >}}
内容としては、以前より **Let's Build…2020** と称し、Youtube で公開したプレゼンテーション動画と同じであるが、PDF/PPTX ファイルも公開されたことでより資料として確認しやすく、そして使いやすくなった。  
{{< link >}}[AMD Ryzen™ Processor Software Optimization - GPUOpen_Let’sBuild2020_AMD Ryzen™ Processor Software Optimization.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_AMD%20Ryzen%E2%84%A2%20Processor%20Software%20Optimization.pdf){{< /link >}}
{{< link >}}[Optimizing for the Radeon RDNA architecture - GPUOpen_Let’sBuild2020_Optimizing for the Radeon RDNA Architecture.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_Optimizing%20for%20the%20Radeon%20RDNA%20Architecture.pdf){{< /link >}}

それ以外にもソフトウェア開発者向けのプレゼンテーション資料を公開している。  

 * [GPUOpen_Let’sBuild2020_A Trip Down the GPU Compiler Pipeline.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_A%20Trip%20Down%20the%20GPU%20Compiler%20Pipeline.pdf)
 * [GPUOpen_Let’sBuild2020_A Review of GPUOpen Effects.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_A%20Review%20of%20GPUOpen%20Effects.pdf)
 * <http://www.gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_Curing%20Amnesia%20and%20Other%20GPU%20Maladies%20with%20AMD%E2%80%99s%20Developer%20Tools.pptx>
 * [2020GDCRpr - GPUOpen_Let’sBuild2020_Radeon ProRender Full Spectrum Rendering 2.0.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_Radeon%20ProRender%20Full%20Spectrum%20Rendering%202.0.pdf)

## 個人的トピック
資料内の個人的なトピックとしては、チップレットアーキテクチャにおいて、同チップレット(CCD) にある 2CCX間の通信でも、I/Oダイ(IOD) 内のデータファブリックを経由する点が明言されたこと、(これより前に為されていたかもしれない)  
そして、*Renoir* の `Data Fabric` と `IO Hub Controller` が 64B/cycle で接続されていることがあげられる。  

後者に関しては、*Matisse* や *Castle Peak* と同じ帯域であり、前世代の *Summit Ridge* 、*Pinnacle Ridge* は 32B/cycle であったことから PCIeGen4 の為に 64B/cycle に増やしたと考えられる。[^gdc2019-ryzen-optim]  
しかし *Renoir* は PCIeGen3 までの対応であるはずで、64B/cycle で接続する必要性は薄い。資料作成者のミスかそれとも制限しているだけか、または PCIe 以外でその帯域で接続する必要があったか。  
AM4 マザーボードでも *Renoir* は PCIeGen3 までであるから制限している可能性は微妙に思う。  

[^gdc2019-ryzen-optim]: <https://gpuopen.com/gdc-presentations/2019/gdc-2019-s2-amd-ryzen-processor-software-optimization.pdf>

それと、*Renoir* では GPUコア部、`Data Fabric` 間の帯域が *Raven /Picasso* から倍に増やされたはずだが、資料中の図からそこまでは読み取れない。  
