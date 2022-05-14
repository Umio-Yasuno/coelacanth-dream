---
title: "NVIDIA GPU の Linux Kernel モジュールがオープンソースに"
date: 2022-05-12T20:32:59+09:00
draft: false
categories: [ "NVIDIA", "GPU", "Software" ]
tags: [ "Linux_Kernel", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

NVIDIA は Linux Kernel の NVIDIA GPU モジュールを ver515 よりオープンソースで公開することを発表した。ソースコードは GPLv2/MIT のデュアルライセンスとなっている。  

 * [NVIDIA Releases Open-Source GPU Kernel Modules | NVIDIA Technical Blog](https://developer.nvidia.com/blog/nvidia-releases-open-source-gpu-kernel-modules/)
 * [NVIDIA/open-gpu-kernel-modules: NVIDIA Linux open GPU kernel module source](https://github.com/NVIDIA/open-gpu-kernel-modules)
 * [Chapter 44. Open Linux Kernel Modules](http://us.download.nvidia.com/XFree86/Linux-x86_64/515.43.04/README/kernel_open.html)

オープンソースモジュールは *Turing* 世代より導入された GSP (GPU System Processor) を前提としているため、サポートする GPU も *Turing* とそれ以降の世代が対象となる。  
データセンター向け GPU は本番環境レベルでサポートされているが、GeForce、ワークステーション向けはアルファクオリティのサポートとなっている。  
現時点で配布されるドライバーには、クローズドソース、オープンソース両方のバージョンの GPU モジュールが含まれ、デフォルトではクローズドソースモジュールがインストールされる。オープンソースモジュールはインストール時に `-m=kernel-open` オプションを追加する必要がある。  

今回公開されたのは KMD (Kernel Mode Driver) 部であり、ユーザースペースで動作する OpenGL/Vulkan ドライバー、UMD (User Mode Driver) 部は含まれていない。  
また、オープンソースモジュールをインストールした場合でも、クローズドソースモジュールと同じ UMD が動作可能だが、リリースバージョンは UMD部と合わせる必要がある。  
一方のバージョンを固定し、また一方は別のバージョンを使用するといったことはできない。  

NVIDIA は、最終的にはオープンソースモジュールがクローズドソースモジュールに取って代わるとしている。  
また、Linux Kernel コミュニティや、Canonical, Red Hat, SUSE などのパトーナーと共に、アップストリームへの合流に取り組む計画があることが語られている。  
現在クローズドソースとなっているコードは Linux Kernel の設計に即していないため、アップストリームの候補にはならない。  

今日、NVIDIA GPU のオープンソースドライバーとしては *Nouveau* が存在する。  
*Nouveau* はコミュニティによって開発されているが、NVIDIA が配布するクローズドソースなドライバーと比べて機能、性能では不足しており、また異なるファームウェアを使用している。  
新たな NVIDIA GPU向けドライバーがアップストリームに合流するまでの間、公開されたソースコードは *Nouveau* へのリファレンスとして機能し、ドライバーの改良を助ける。  
それにより、クローズドソースモジュールと同じファームウェアの使用、電力管理 (クロック調整、熱管理) 機能といった多くの機能を *Nouveau* ドライバーにもたらすとしている。  

## Nouveau

NVIDIA GPU モジュールと今後のさらなるオープンソース化による *Nouveau* ドライバーの今後の展望について、Red Hat に所属している Christian F.K. Schaller 氏が GNOME Blogs に記事を投稿している。  

 * [Why is the open source driver release from NVidia so important for Linux? | Christian F.K. Schaller](https://blogs.gnome.org/uraeus/2022/05/11/why-is-the-open-source-driver-release-from-nvidia-so-important-for-linux/)

Christian F.K. Schaller 氏は、オープンソース化されたモジュール、それをリファレンスとして改良されていく KMD を、クローズドな NVIDIA GPU ドライバーと Mesa3D 両方の UMD で共有する計画に取り組むとしているが、実現には数年掛かる可能性があると語っている。  
これには NVIDIA GPU ドライバーと Mesa3D、どちらのニーズにも対応できるよう KMD を設計する必要があり、NVIDIA とオープンソースコミュニティの連携が不可欠となる。  

{{< ref >}}
 * <https://twitter.com/cfkschaller>
 * [Chapter 43. GSP Firmware](https://download.nvidia.com/XFree86/Linux-x86_64/515.43.04/README/gsp.html)
 * [The Mesa drivers matrix](https://mesamatrix.net/)
 * [nouveau - Gentoo Wiki](https://wiki.gentoo.org/wiki/Nouveau)
 * [NvHardwareDocs](https://nouveau.freedesktop.org/NvHardwareDocs.html)
{{< /ref >}}
