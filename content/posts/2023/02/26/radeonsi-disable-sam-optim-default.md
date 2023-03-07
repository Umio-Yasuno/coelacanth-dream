---
title: "RadeonSI ドライバーの Smart Access Memory 向け最適化がデフォルトで無効に"
date: 2023-02-27T01:09:55+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "Smart_Access_Memory", "RadeonSI" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日 Mesa3D にマージされた AMD GPU 向け OpenGL ドライバー、*RadeonSI* への変更の中で、ResizableBAR (Base Address Register) /Smart Access Memory (SAM) 向けの最適化をデフォルトで無効化するものがあった。  
元々 *RadeonSI* では ResizableBAR/SAM が有効な場合、実際の判定としては CPU-accessible/visible VRAM のサイズを見て充分なサイズがある場合にそれ向けの最適化を有効化していたが、後に安定してすべてのプラットフォームで性能向上を得られる訳ではないとして、とりあえず *>=Zen 3 CPU + >=RDNA 2 GPU* の組み合わせでのみデフォルトで有効化する処置が取られた。[^sam-zen3]  
それが今回、ゲームによっては最適化が逆に働き、著しく遅くなることが報告されたため、どのプラットフォームであってもデフォルトで無効化することとなった。  

[^sam-zen3]: [RadeonSI では Smart Access Memory をオプションで切り替える方式に、 Zen 3 + RDNA 2 ではデフォルトで有効 | Coelacanth's Dream](/posts/2020/12/24/change-of-radeonsi-sam/)

 * [amd,radeonsi: lots of small changes and fixes for gfx11 and others, some cleanups (!21403) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/21403)
    * [radeonsi: disable Smart Access Memory because CPU access has large overhead (d8b17b17) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/d8b17b17526b46d69e4102a883ba451e7f1db148?merge_request_iid=21525)
 * [radeonsi: Hyperdimension Neptunia Re;Birth1, very slow (#8176) · Issues · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/issues/8176)

性能が低下する原因として当該 issue では、GPU VRAM への CPU アクセスによるオーバヘッドが増え、それにより GPU の使用率が上がらないことが挙げられている。  

 >         +   sscreen->info.smart_access_memory = false; /* VRAM has slower CPU access */
 >         +
 >
 > {{< quote >}} [radeonsi: disable Smart Access Memory because CPU access has large overhead (d8b17b17) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/d8b17b17526b46d69e4102a883ba451e7f1db148) {{< /quote >}}

同様の現象は Direct3D 12 (D3D12) の Vulkan 実装、VKD3D と *RADV* ドライバーの組み合わせでも報告されているが、オプションの `VKD3D_CONFIG=no_upload_hvv` により BAR 領域 (`VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT | VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`) を使わないようにすることで回避可能ということで VKD3D 側の問題として close された。  

 * [Smart Access Memory / Resizable Bar not working on B550 boards (#7993) · Issues · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/issues/7993)

AMD が ResizableBAR を *Smart Access Memory* という名前を付けてマーケティング的に打ち出してから 2年以上経つが、ResizableBAR/SAM とそれ向けの最適化が実際に性能を向上させるかは一部 (主に Linux 環境) で話題になり続けていると感じる。  
VKD3D の `CHANGELOG.md` にある記述によれば、D3D12 では BAR 領域を明示的に使用可能とするヒープメモリタイプが無いとしている。このことが主に Linux 環境でのみ話題になっていることと関係しているかもしれない。[^vkd3d]  

多くのアプリケーション、ゲームでは性能が向上する一方で、逆に性能が低下するものもあるという点がこの問題の難しい部分だろう。  
VKD3D の同様部分では、ResizableBAR/SAM の効果はゲームによって異なるとしつつも、ベストケースでは 10-15% の性能向上が確認できるとしている。それほどの性能向上を見ながら活用しないのは、性能を重視するユーザーや開発者にとっては惜しい話だ。  
最初に取り上げた AMD の Marek Olšák 氏によるマージリクエストも、当初は最適化のデフォルトでの無効化だけでなく、最適化に関するコードを削除するコミットも含まれていたが、後に更新されて無効化のみとなった。  
今後調査が進めば、*RadeonSI* ドライバーで ResizableBAR/SAM 向けの最適化が再度有効化されるかもしれない。  

{{< ins >}}
その後、別のマージリクエストで ResizableBAR/SAM の最適化に関するコードが削除された。

 * [amd,radeonsi: reviewed commits from !21403 (!21641) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/21641)
{{< /ins >}}

[^vkd3d]: [vkd3d-proton/CHANGELOG.md at ac8760c00534b0c28a1de51af38844718e068119 · HansKristian-Work/vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton/blob/ac8760c00534b0c28a1de51af38844718e068119/CHANGELOG.md#performance)

{{< ref >}}
 * [The Catastrophe of Reading from VRAM – Bas Nieuwenhuizen – Open Source GPU Drivers](https://www.basnieuwenhuizen.nl/the-catastrophe-of-reading-from-vram/)
 * [Using Vulkan® Device Memory - AMD GPUOpen](https://gpuopen.com/learn/vulkan-device-memory/)
 * [Vulkan Memory Types on PC and How to Use Them](https://asawicki.info/news_1740_vulkan_memory_types_on_pc_and_how_to_use_them)
{{< /ref >}}
