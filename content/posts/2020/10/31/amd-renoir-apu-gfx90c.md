---
title: "AMD Renoir APU の GPU ID が \"gfx90c\" に"
date: 2020-10-31T00:24:51+09:00
draft: false
tags: [ "Renoir", "VanGogh", "gfx90c" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

前の記事で、[LLVM に新たな GPU ID "gfx90c" が追加され](/posts/2020/10/30/amd-gpuid-gfx90c/)、それが *Renoir* 以降の APU、*Lucienne / Green Sardine / Cezanne* のどれか、もしくはいくつかに採用されている GPUコア部 (GPUアーキテクチャ) を指すのではないかと書いたが、  
その後のパッチへのコメントにて、*gfx90c* は *Renoir* の GPU部を指すことが判明した。  
{{< link >}} [[AMDGPU] Add gfx90c target · llvm/llvm-project@ee3e642](https://github.com/llvm/llvm-project/commit/ee3e642627575bc8fb0a7008f8a02c891e172b7b) {{< /link >}}

これまで TSMC 7nmプロセスで製造される *Renoir* の GPU ID は *gfx909* とされ、これは GF 14nmプロセスで製造される省電力 APU *Raven2 APU (Dali /Pollock)* の GPU ID と同じだった。  
だが実際には、*Renoir* と *Raven2* の GPUアーキテクチャには若干の差異があり、今回の *gfx90c* 追加により区別されることとなる。  
なおも *gfx909* と *gfx90c* の間にどういった違いがあるのかは不明だが、バージョンとしてはステッピングが進んだものであるため、そう大きくは変わらないものと思われる。  

このサイトではこれまで *Renoir* の GPU ID は *gfx909* として解説してきたが、当時の情報を一連の流れとして、ある種の物語として記録する意味があると考えているため、過去の記事は基本そのままとし、データベースとしている一部の記事のみに今回の変更を適用するつもりでいる。  

下記は Zen系 APU とその GPU ID をまとめた表。  
[VanGogh](/tags/vangogh) の GPU ID とされる *gfx1033* を LLVM に追加するパッチも投稿されており、対応命令は現段階で *gfx1030 / Sienna Cichlid* と同じとなっている。  
レイトレーシング系の命令を *APU* でサポートするとして、どれ程活用できるかは少し疑問に思うところがあるが、*RDNA 2 /GFX10.3* では CU内に *RA (Ray Accelarator)* を搭載する構造であるため、ダイサイズへの影響はそう大きくはないのかもしれない。  
{{< link >}} [⚙ D90447 [AMDGPU] Add gfx1033 target](https://reviews.llvm.org/D90447) {{< /link >}}
{{< link >}} [[AMDGPU] Add gfx1033 target · llvm/llvm-project@89d41f3](https://github.com/llvm/llvm-project/commit/89d41f3a2b820c17edeb86a2e9d458ca69eca34e) {{< /link >}}

| Zen APU | GPU ID |
| :-- | :--: |
| Raven | gfx902 |
| Picasso | gfx902 |
| Raven2 | gfx909 |
| Renoir | *gfx90c* |
| Lucienne | gfx90c? |
| Green Sardine | gfx90c? |
| Cezanne | gfx90c? |
| VanGogh | gfx1033? |
