---
title: "AWS、RDNAアーキテクチャ GPU 「Radeon Pro V520」を採用する G4adインスタンスを発表"
date: 2020-12-02T20:06:58+09:00
draft: false
tags: [ "Navi12", "RDNA" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AWS(Amazon Web Servises) は EC2 の新たなコンピュートインスタンスを複数発表した。  
その中の G4adインスタンスは、第二世代 AMD EPYC プロセッサ (*Rome*) と *RDNAアーキテクチャ* を採用する **AMD Radeon Pro V520** で構築されている。  
{{< link >}} [AWS Announces Multiple New Compute Innovations, Including Five New Instance Types for Amazon EC2, Two New AWS Outposts SKUs, and Three New AWS Local Zones Locations Across the US | Amazon.com, Inc. - Press Room](https://press.aboutamazon.com/news-releases/news-release-details/aws-announces-multiple-new-compute-innovations-including-five) {{< /link >}}
{{< link >}} [Amazon EC2 G4 Instances — Amazon Web Services (AWS)](https://aws.amazon.com/jp/ec2/instance-types/g4/) {{< /link >}}

Amazon AWS は 2020/11/30 から 2020/12/18 に掛けて、バーチャルイベント [AWS re:Invent](https://reinvent.awsevents.com/) を開催している。  

G4adインスタンスも近日提供が開始され、クラウド上でグラフィクス処理を行なうアプリケーションにとって最高のコストパフォーマンスを実現するとしている。  
インスタンスは最大で 64 vCPU、256GB Memory、4 GPU、NVMeストレージ 2.4TB の規模を選択可能。  

## AMD Radeon Pro V520

今回発表された **AMD Radeon Pro V520** についてまだ仕様等は AMD、AWS どちらからも明かされていないが、  
*RDNAアーキテクチャ*、クラウド向け GPU として仮想化機能に対応しているであろうことから、[Navi12 (gfx1011)](/tags/navi12) ベースではないかと思われる。  
*Navi12* は、*RDNAアーキテクチャ* を採用する GPU の中では唯一 SR-IOV (Single-Root I/O Virtualization)[^mxgpu] に対応し、また専用の DeviceID も持つ。  
{{< link >}} [Navi12情報近況 (2020-03-09) & 推測 | Coelacanth's Dream](/posts/2020/03/09/navi12-recent-info/) {{< /link >}}
また、AMD が Github上で公開している、[AMD-CloudGPU/Gibraltar-LinuxGuestKernel-Public](https://github.com/AMD-CloudGPU/Gibraltar-LinuxGuestKernel-Public/tree/2020_Q3_Release) レポジトリを見ても、*Navi12* に向けた対応が継続的に行なわれている。  

クラウド向け AMD GPU としては、[Vega10](/tags/vega10) をベースとする **Instinct MI25**[^mi25] ことが多かった。{{< comple >}}*Vega20* ベースの **Instinct MI50** を採用した話はほとんど聞かない。グラフィクス処理よりもコンピュート処理に向いた製品だからだろうか。{{< /comple >}}  
*Navi12* はメモリインターフェイスに HBM2 2048-bit を採用しており、この点では *Vega10* と一致するため、グラフィクス性能の向上、省電力化を目的としたリプレース先として優秀なのだと思われる。  
{{< link >}} [AMD、HBM2 を採用した Navi GPU、Radeon Pro 5600M を 16インチMacBook Pro向けに発表 | Coelacanth's Dream](/posts/2020/06/15/amd-radeon-pro-5600m-mbp/) {{< /link >}}

[^mxgpu]: [GPU Virtualization Solution | MxGPU Technology | AMD](https://www.amd.com/en/graphics/workstation-virtual-graphics)
[^mi25]: [Radeon Instinct™ MI25 Accelerator | Server Accelerators| AMD](https://www.amd.com/en/products/professional-graphics/instinct-mi25#product-specs)
