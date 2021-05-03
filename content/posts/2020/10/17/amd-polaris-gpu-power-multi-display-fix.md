---
title: "Linux Kernel に AMD Polaris GPU の電力管理を改良、修正するパッチが投稿される"
date: 2020-10-17T08:38:11+09:00
draft: false
tags: [ "Polaris10", "Linux_Kernel", "GFX8" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

[先日](/posts/2020/10/11/llvm-add-gfx6_8-gpu/) の LLVM に追加された GPUID といい、前世代の GPU に関するコードの見直しが行なわれているのだろうか。  

Linux Kernel (amd-gfx) に、*Polaris1x* GPU の電力管理を改良、修正する 40個のパッチが投稿された。  
{{< link >}} [[PATCH 00/40] Various Polaris fixes and optimizations](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054852.html) {{< /link >}}
パッチには、ある温度に達するまでファンを停止する Zero RPM 機能のサポート、マルチディスプレイに関する問題の修正が含まれている。  
*Poalris1x* GPU のコアクロック、メモリクロック、ファン制御が改良された。そして、マルチディスプレイ時でもメモリクロックの切り替えが行なわれるようになり、アイドル時の消費電力が削減される。  

**RX 560 (Polaris11/21)** を愛用している自分には嬉しいパッチと言える。  

また、ここまで *Polaris1x* GPU とまとめて呼んでいるが、ハードウェア側の電力管理ユニットの改良が為された *Polaris1x* の後継(kicker)、*Polaris2x* が存在し、それらは **RX 500シリーズ** のベースとなっている。  
それに加え、*Polaris20 (RX 580/570 etc)* の製造プロセスを GF 14nm から GF 12nm に移行させた *Polaris30* も存在し、そちらは **RX 590** のベースとなる。  
{{< link >}} [[PATCH 01/40] drm/amd/pm: correct the checks for polaris kickers](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054853.html) {{< /link >}}

そして、今回のパッチで知ったのだが *Polaris31* も存在するという。ただ、ベースに使われているのは **RX 560X** のみとなる。[^rx-560x]  
**RX 560** と **RX 560X** の違いは正直よく分からず、AMD 公式サイトを見てもベース/ブーストクロックは同じであり、違いとなり得そうな TDP、製造プロセスは記載されていない。[^560-560x]  
一応、*Polaris30/31* の SMC(System Management Controller?) ファームウェアはそれ以外と分けられている。  
さらに、*Polaris30* をベースとするのは **RX 590** だけではなく、**Radeon RX P30PH** も該当するらしいのだが、この **RX P30PH** についての情報が他に無い。[^rx-p30ph]  

ある世代の GPU の後期に生まれやすい謎と言えるかもしれない。  
先日 **Xbox One SoC** と同一ダイをベースとしていることが判明した **A9-9820 (Cato APU)** のように[^xbox-one-cato]、前世代の APU/GPU と言っても全てが明かされている訳ではなく、魅力とも言い換えられるであろう謎はまだまだある。  

[^rx-560x]: <https://github.com/GPUOpen-Tools/common_src_device_info/blob/0db916e3f16632ea797d703462d4905396951eb8/DeviceInfoUtils.cpp#L368>
[^560-560x]: <https://www.amd.com/en/products/specifications/compare/graphics/1261,7511>
[^rx-p30ph]: [data/amdgpu.ids · 5de99aebba44212ff048145a91e66490a98b2233 · Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm/-/blob/5de99aebba44212ff048145a91e66490a98b2233/data/amdgpu.ids#L134)
[^xbox-one-cato]: [A9-9820 は Xbox One SoC と同一ダイ | Coelacanth's Dream](/posts/2020/10/14/a9-9820-silicon/)
