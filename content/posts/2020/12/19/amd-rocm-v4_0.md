---
title: "AMD、MI100 を正式にサポートした ROCm v4.0.0 をリリース & RDNA/GFX10系は 2021年にサポート予定"
date: 2020-12-19T06:55:51+09:00
draft: false
tags: [ "ROCm", "CDNA", "RDNA", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "AMD", "Software", "GPU" ]
noindex: false
# summary: ""
toc: false
---

*CDNA1 アーキテクチャ* を採用する **AMD Instinct™ MI100** を正式にサポートする **ROCm v4.0.0** がリリースされた。  

 * [RadeonOpenCompute/ROCm at roc-4.0.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-4.0.x)

更新内容もそれに沿っており、各種ツールへの **MI100** サポート追加、ライブラリの *xGMI (inter-chip Global Memory Interconnect)* サポートが行なわれている。  
また、オープンソースドライバー Mesa3D のサポートが追加された。  
ただ、Mesa3D のマルチメディア部がメインであり、これは **MI100** が、グラフィクス処理用の固定ユニットは持たないが、デコード/エンコード処理を行なうマルチメディアエンジンを搭載しているからと思われる。  
**MI100** のマルチメディアエンジンは機械学習のオブジェクト検出といった用途が想定されている。  


## 2021年に RDNA/GFX10系もサポート予定

12月の初めに [Navi12 (gfx1011)](/tags/navi12) をベースとする **Radeon Pro V520** と、それを採用する AWS EC2 G4adインスタンスが発表された。[^aws-g4ad]  
そして、ROCm の一部ツールが *Navi12 (gfx1011)* がサポートされていないことに対しあるユーザーから issue が出された。  
ROCmのサポーターはそれに、現在 *RDNA/GFX10* GPU を正式にサポートしていないが、2021年にサポートする予定があると返答している。  

 >  Hi @amrragab8080  
 >  Thanks for reaching out.  
 >  We are not officially supporting Navi series of cards at present.  
 >  We have plans to support in 2021 and we will update accordingly.  
 >  Thank you.  
 >  
 >  {{< quote>}} <https://github.com/RadeonOpenCompute/ROCm/issues/1341#issuecomment-747851746> {{< /quote >}}

[^aws-g4ad]: [AWS、RDNAアーキテクチャ GPU 「Radeon Pro V520」を採用する G4adインスタンスを発表 | Coelacanth's Dream](/posts/2020/12/02/aws-radeon-pro-v520/) <br> [AMD、Radeon Pro V520 を正式発表 & 仕様公開 | Coelacanth's Dream](/posts/2020/12/03/amd-radeon-pro-v520/)

ただ、*RDNA アーキテクチャ* とか *GFX10* といった断定する言い方ではなく、*Navi series* と少しぼかした言い方をしている。  
既に *RDNA 2/GFX10.3* GPU である [Sienna Cichlid](/tags/sienna_cichlid) は一部 **ROCm** ライブラリが対応を進めており、それを指している可能性はある。  
{{< link >}} [ROCmソフトウェアが Sienna Cichlid (gfx1030) サポートに向けて動き始める | Coelacanth's Dream](/posts/2020/10/09/rocm-initial-support-gfx1030/) {{< /link >}}
しかしこれは邪推であり、*Navi12 (gfx1011)* のサポートに対しての返答であるのだから、*RDNA/GFX10* GPU もサポートされると考えるのが自然だろう。  

過去に *RDNA/GFX10* GPU のサポートについて、公式的なサポートはサーバー向けブランド **Radeon Instinct** を用いて検証が行なわれるとコメントされていた。  
そのため、*RDNA/GFX10* をベースとしたカードがリリースされていないため、**ROCm** の公式サポートへの追加は望み薄という状態だった。  
**Radeon Pro V520** もサーバー向け GPU ではあるが、**Radeon Instinct** が GPGPU、コンピューティング向けの位置にあるのに対し、**Radeon Pro V520** はクラウドゲーミングといったクラウド上でのグラフィクス処理向けとなる。  
それでも **ROCm** の *RDNA/GFX10* GPU サポートは確かに需要があると認識されており、エクサスケールスパコンにおける機械学習、HPC を満たすソフトウェアスタックとなる **ROCm v4.0.0** リリースが達成されたこともあって、サポートを追加する余裕と目処が立った、ということと考えられる。  

