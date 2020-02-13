---
title: "drm amdgpu.idsアップデート (2020-01-31)"
date: 2020-01-31T12:44:37+09:00
draft: false
tags: [ "Radeon", ]
keywords: [ "Radeon" ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

例の通り、まだマージリクエストの段階だがまとめ。  
[amdgpu: add new marketing names from 19.50 - Mesa / drm GitLab](https://gitlab.freedesktop.org/mesa/drm/merge_requests/44/diffs)  

追加されたGPUは以下、

 * AMD Radeon Pro W5700
 * AMD Radeon RX 5600M
 * AMD Radeon RX 5700M
 * AMD Radeon RX 5600 XT
 * AMD Radeon RX 5600 OEM
 * AMD Radeon RX 5500 XT
 * AMD Radeon Pro W5500
 * AMD Radeon Pro W5500M

最近発表されたNavi系が主だが、未発表の**AMD Radeon Pro W5500/M**が入っている。  
それらの *[DeviceID : RevisionID]* は7341:00、7347:00であり、前々から確認されていたNavi14 Pro-XL、Navi14 Pro-XTMのものと一致する。  
以前からあるプロプライエタリ・ドライバー内のファイルには記述されていたが、今回でオープンソース・ドライバー関連のファイルにも追加された。正式発表が近いのかもしれない。  

[前回](/posts/2019/12/04/amdgpu-ids-updated/)は追加されなかった AMD Radeon Pro W5700 があるが、USB-Cコントローラーのファームウェアは未だなく、AMD公式のドライバーページにもまだWindows用しかない。  
[Radeon™ Pro W5700 Drivers & Support | AMD](https://www.amd.com/en/support/professional-graphics/radeon-pro/radeon-pro-w5000-series/radeon-pro-w5700)  

**Radeon RX 5600M** / **Radeon RX 5700M**のDeviceIDは共通で734F、RevisionIDはC2/C3となっているが、これらは以前Linux Kernelへのパッチで確認されており、  
見つけた時はどういった意味かわからなかったが、今は[Smart Shift](https://www.amd.com/en/technologies/smartshift)のためだと考えられ、  
[drm/amdgpu: enable gfxoff feature for navi10 asic - Linux Kernel](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=a9e45a6f90b42703ae24adac662e9dd4c2c0d68f)  
gfxoffというのはAMDのScott Stankard氏の発言とも一致する。  

 > ――グラフィックス処理の負荷に応じて，CPUとGPUの間で電力を動的に振り分ける「AMD SmartShift」において，iGPUと単体GPUを同時に稼動させることはできるのか。

 > Stankard氏：
　iGPUと単体GPUを同時に動かすことはできない。iGPUと単体GPUのどちらか一方が動いているとき，稼動していないGPUはほぼ完全に動作を停止する。

 > (引用元: [西川善司の3DGE:Ryzen 4000とRadeon RX 5600 XTの気になるところをAMDにアレコレ聞いてみた - 4Gamer.net](https://www.4gamer.net/games/446/G044684/20200115091/))

ついでとなるが、Navi10でGDDR6 12Gbpsを採用したSKUにて不具合があったらしく、Linux Kernleへそれを修正するパッチが投稿された。  
[\[PATCH\] drm/amd/powerplay: fix navi10 system intermittent reboot issue - amd-gfx](https://lists.freedesktop.org/archives/amd-gfx/2020-January/045418.html)  
その中でだいぶ直接的に 5600M /5700M /5600 XT /5600 OEM が指定されており、{{< comple >}}テスト用かネットに公開されるベンチマーク対策かはわからないが{{< /comple >}}もう1つのIDも記述されている。  
AMDにそれらを隠す気はなくなったのだろうか？  

最後にどうでもいいことだが、OEM向けの**Radeon RX 5500**に対して、同じくOEM向けの**Radeon RX 5600**には OEM が末尾に付いているのはどういった理由からなのだろう。  
