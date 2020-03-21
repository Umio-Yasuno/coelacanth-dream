---
title: "さらなるIntel TGL/XeのPCI IDが追加 & Intelにデスクトップ向け TGL を出すつもりはあるのか"
date: 2020-03-21T19:04:35+09:00
draft: true
tags: [ "Gen12", "DG1" ]
keywords: [ "", ]
categories: [ "Hardware", "GPU", "CPU", "Intel", ]
noindex: false
---

## Mesa3DにさらなるTGL/X<sup>e</sup>のPCI IDが追加
[intel: add new TGL pci ids (58deebe5) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/58deebe547014e64d8db3f8cc5e963efe7e0f743)  
[Update some TGL strings & add a TGL pci-id (!4267) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/4267/diffs)  

Mesa3DへのパッチにてTGL GT2のPCI IDが5個追加された。  
それと同時に `0x9A40`、`0A9A49` のデバイス名が、 `Intel(R) Graphics` から `Intel(R) Xe Graphics` へと変更されている。  
それら2つの *TGL GT2* だが、その他は変更されていないため、iGPUタイプではないdGPUタイプの Intel DG1 のPCI IDかもしれない。  
または、AMD Radeonにおける *RX* のようなブランドとして、UHDよりも上であることを示すのに X<sup>e</sup> を使うつもりか。  

