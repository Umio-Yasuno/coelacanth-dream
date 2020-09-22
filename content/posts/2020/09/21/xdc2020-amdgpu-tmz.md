---
title: "【XDC2020】AMD GPU のメモリ暗号化機能 TMZ　―― DRMへの活用も"
date: 2020-09-21T23:03:53+09:00
draft: false
tags: [ "Linux_Kernel", "RadeonSI" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

**XDC2020** にて、AMD GPU のドライバーに実装されている TMZ(Trust Memory Zone) 機能について、AMD の Ray Huang 氏が講演を行なった。  

 * [XDC 2020 - Day 1 - September 16, 2020 - YouTube](https://www.youtube.com/watch?v=b2mnbyRgXkY&t=14910s)
 * [X.Org Developers Conference 2020 (16-18 September 2020): Secure Buffer Object Support with Trusted Memory Zone · Indico](https://xdc2020.x.org/event/9/contributions/630/)


メモリの暗号化は不正なアプリケーションがあるコンテンツにアクセスできないよう保護する重要な機能である。  
AMD GPU で開発されている TMZ はメモリに書き込まれる内容、メモリから読み出される内容を AES暗号化によって保護する。  
Linux Kernel(amd-gfx) へのパッチは 2020/09 に投稿され、ユーザーモードのドライバーでもサポートされている。  

ページテーブルへの TMZ bit のセット、信頼できるトランザクション処理という複数の保護が提供され、TMZ bit が誤っていたとしても生の内容が漏れることはない。  
TMZ は専用VRAMを持つ dGPU、CPUとGPUでメモリを共有する APU、どちらのプラットフォームでもサポートされており、システムメモリに対しても暗号化を行なえる。  

CPUは信頼できないドメインとされ、TMZ によって暗号化された内容を復号化することができない。  
復号化が可能なのは、GPU内の信頼できるハードウェアブロック *GFX, SDMA, Media Engine(VCN), Display Engine* 部のみとなる。  
CPUからの復号化を出来なくすることによって保護を強化するとともに、VRAMからGPUアクセラレーションを有効にしたブラウザの表示内容を読み取るといった、GPUの脆弱性への対策となるのではないかと思われる。  

スライド内では触れられていないがコードを読むに、全ての AMD GPU で TMZ 機能を使用可能という訳ではなく、*Vega /GFX9* 世代の APU (*Raven/Picasso/Raven2/Renoir*)、*RDNA /GFX10* 世代の GPU (*Navi10/Navi12/Navi14*) のみでサポートされているようだ。[^asic-tmz]  

[^asic-tmz]: [drm/amdgpu: Fine-grained TMZ support](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=b71a564e2509e1000044a9873cbee6d6a6a5ab90)

TMZ は Widevine DRM に必要とされる重要な機能であり、TMZ と HDCP(High Bandwidth Digital Content Protection) の組み合わせは、AMDプラットフォームにおいて Widevine がセーフチャネルを構築するのに活用することができる。  
そして、AMD のソフトウェア開発者達は TMZ によるエンドツーエンドでの DRMソリューションに取り組んでいるとし、講演を結んでいる。  




