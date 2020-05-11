---
title: "RadeonSI が Trust Memory Zone をサポート"
date: 2020-05-11T22:11:12+09:00
draft: false
tags: [ "", ]
keywords: [ "", ]
categories: [ "", ]
noindex: false
---

AMD GPU向けのオープンソースOpenGLドライバー、[RadeonSI](/tags/radeonsi)が最新コミットで TMZ(Trust Memory Zone)機能をサポートした。  
{{< link >}}[radeonsi: tmz support (!4401) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/4401){{< /link >}}

## Trust Memory Zone
Trust Memory Zone はセキュリティに関する機能であり、メモリをページレベルで暗号化することでメモリの内容を保護する。  
カーネルドライバーはそれにより BO(Buffer Object、カーネルドライバーが使用するGPUメモリ単位) を暗号化し、暗号化された内容は GPU内の信頼されたIPブロック(GFX, SDMA, ビデオデコーダー/エンコーダー)のみ復号化することが可能となる。  
そうでないCPUから読み取ることはできない。[^2]  

[^2]:[[PATCH 00/14] drm/amdgpu: introduce secure buffer object support (trusted memory zone)](https://lists.freedesktop.org/archives/amd-gfx/2019-September/039928.html)

RadeonSI で TMZ を使用するには、それをサポートする Linux Kernel と libdrm が必要となるが、どちらもまだメインラインには組み込まれておらず、それぞれ開発中のリポジトリからビルドする必要がある。  
{{< link >}}<https://cgit.freedesktop.org/~agd5f/linux/log/?h=drm-next-5.8>{{< /link >}}
{{< link >}}[Pierre-Eric Pelloux-Prayer / drm · GitLab](https://gitlab.freedesktop.org/pepp/drm/-/tree/tmz){{< /link >}}
Linux Kernel のメインラインに組み込まれるのは v5.8 からと見られている。(現在のメインラインのバージョンは v5.7-rc5)  

今回サポートされたのは RadeonSI だけであり、Vulkanドライバーの RADV はまだサポートされていない。  

また、この TMZ は具体的に何が動機となって開発が進められているかは明かされていないが、[Phoronix](https://www.phoronix.com/scan.php?page=home)の[Michael Larabel](https://www.phoronix.com/scan.php?page=michaellarabel)氏は、AMD APU/GPU を搭載した [Chromebook](/tags/chromebook) の開発が起因になっているのではないかと見ている。[^1]  

[^1]: [AMDGPU Trusted Memory Zone Support Could Soon Be Enabled By Default - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=AMDGPU-TMZ-Default-Soon)

{{< ref >}}

 * [RadeonSI Driver Now Supports AMD Trusted Memory Zone - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=RadeonSI-TMZ-Mesa-20.2)
 * [AMDGPU Trusted Memory Zone Support Could Soon Be Enabled By Default - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=AMDGPU-TMZ-Default-Soon)

{{< /ref >}}
