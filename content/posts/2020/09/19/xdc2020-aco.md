---
title: "【XDC2020】 ACOバックエンドの今後の計画 ―― RDNA 2, RT, Mesh Shader"
date: 2020-09-19T09:51:24+09:00
draft: false
tags: [ "ACO", "RadeonSI", "RADV" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

先日より **XDC 2020** が開催され、オープンソース・グラフィックに関する発表が数々行なわれている。  
今回は **A year of ACO: from prototype to deafault** と題された発表の中で語られた、[ACOバックエンド](/tags/aco) の今後の計画とサポートする機能について紹介したい。  

 * [XDC 2020 - Day 2 - September 17, 2020 - YouTube](https://www.youtube.com/watch?v=FxFPFsT1wDw&t=1736s)
 * [X.Org Developers Conference 2020 (16-18 September 2020): A year of ACO: from prototype to default · Indico](https://xdc2020.x.org/event/9/contributions/612/)

{{< pindex >}}

 * [今後の計画と機能](#future-plan)
   * [RDNA 2/GFX10.3 サポート](#rdna_2-support)
   * [RadeonSI(OpenGL) でも ACOバックエンドを](#radeonsi-aco)
   * [レイトレーシング](#rt)
   * [Mesh Shader は NGG で実行可能か](#ms-on-ngg)
   * [さらなる最適化](#more-optim)

{{< /pindex >}}

## 今後の計画と機能 {#future-plan}
### RDNA 2/GFX10.3 サポート {#rdna_2-support}
*RDNA 2 /GFX10.3* のサポートは進行中だが、次世代 GPU であるためまだ公式のISAドキュメントがリリースされておらず、AMD の開発者が公式にパッチを投稿する *RadeonSI (OpenGL)* ドライバーや LLVM の変更に追従する形を取っているとのこと。(Page51)  

### RadeonSI(OpenGL) でも ACOバックエンドを {#radeonsi-aco}
現在 *ACOバックエンド* は *RADV(Vulkan)* ドライバーのみをサポートしているが、*RadeonSI(OpenGL)* のサポートも計画されている。(Page52)  
それに向けたコードのシェーダーの引数部の統合、I/O操作部の統合作業は始まっているが、元は *RADV(Vulkan)* への実装だったこともあり、サポートまで長い道のりであると述べている。  

### レイトレーシング {#rt}
Vulkan API でレイトレーシングをサポートする拡張仕様はドラフト(草稿)の段階にあり、先日には、*RDNA 2 /GFX10.3* がサポートするレイトレーシング関連の命令が LLVM に追加されているため、*ACO* でもサポートされる日はそう遠くない……が肝心のハードウェア、*RDNA 2 /GFX10.3* GPU そのものがまだリリースされていない。  

### Mesh Shader は NGG で実行可能か {#ms-on-ngg}

NVIDIA が Turing 世代で新設したプログラムシェーダー、Mesh Shader だが、スライド内にて *RDNA /GFX10* からハードウェア側でサポートされている NGG(Next Generation Geometory) で実行できる *可能性* があると述べている。(Page54)  
まだ Vulkan API での拡張機能がリリースされていないため実装、テストは行なわれていないが、  
もし実行可能であるとして、次世代コンソール機でサポートが分かれていた、PS5 のプリミティブシェーダー、Xbox Series X/S でサポートされている Mesh Shader が、ハードウェア側としてはどちらも NGG によって実行されていることとなる。  

NGG 自体は *Vega /GFX9* から始まったものだが、*RDNA /GFX10* で設計が変更され、*Vega /GFX9* でのサポートは廃止された[^remove-gfx9-ngg]という背景がある。  
これはあくまでも個人の推測であるが、*RDNA /GFX10* で NGG の設計変更を行なった理由の1つに、Mesh Shader のサポートが含まれていたのかもしれない。  

[^remove-gfx9-ngg]: [[PATCH] drm/amdgpu: remove gfx9 NGG](https://lists.freedesktop.org/archives/amd-gfx/2019-September/040258.html)

### さらなる最適化 {#more-optim}

*ACOバックエンド* のさらなる性能向上を実現する手立てに、Rapid-packed math(FP16をFP32の倍のレートで実行する機能、GFX9からサポートされている)、NGG GS への最適化、さらに進んだレジスタ割り付けスケジューラーの実装をあげている。  


<!--
   [WIP aco: rapid packed math (!6680) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6680)
-->

{{< ref >}}

 * [西川善司の3DGE：Mark Cerny氏のPS5技術解説プレゼンテーションを読み解く(前編)。ここまで分かったPS5のSSDとGPUの詳細](https://www.4gamer.net/games/990/G999027/20200319173/)

{{< /ref >}}
