---
title: "RadeonSI/RADVドライバーに Smart Access Memory に向けた最適化が行なわれる"
date: 2020-12-07T05:32:34+09:00
draft: false
tags: [ "RadeonSI", "Smart_Access_Memory", "RADV" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMD GPU のオープンソース OpenGL ドライバー **RadeonSI** に、*Smart Access Memory / Resizeable PCI BAR* に向けた最適化パッチが投稿された。  
{{< link >}} [ac,radeonsi: optimizations for Smart Access Memory (all VRAM visible) (!7951) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7951/commits) {{< /link >}}

パッチを投稿した AMD の [Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏は、CPU から一度に GPU VRAM へのアクセスが可能な (CPU から見えている) 状態を *Smart Access Memory* としている。  
既に各メディア、テクニカルライターへの AMD による説明から、PCI Express の仕様に存在する *Resizeable PCI BAR* を利用したものという話が出ていたが、それは Linux環境においても同様で、*Smart Access Memory* 自体は *Resizeable PCI BAR* 以上に特別なことはしていないと捉えてよさそうだ。  

今回 **RadeonSI** ドライバーには、一度に最大 256MB までしかアクセスできない状況を想定した、バッファを確保するコードを *Smart Access Memory / Resizeable PCI BAR* が有効な場合はスキップするといった最適化が行なわれている。  

以前検証を行なった時は、Vulkan API を用いるソフトウェアのみを試したが、OpenGL API でもある程度の効果はあるのだろう。  
一連のパッチがメインラインに組み込まれた時は再度検証するつもりだが、やっぱり Linux でも動作する OpenGLベンチマークというのは少なかったりする。  

その後 **RADV (Vulkan)** ドライバーにも同様の最適化パッチが投稿された。  
{{< link >}} [radv: Use VRAM for CS & upload buffer if all VRAM is CPU-visible. (!7979) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7979) {{< /link >}}

**RADV** の開発者である [Bas Nieuwenhuizen](https://gitlab.freedesktop.org/bnieuwenhuizen) 氏は検証も行なっており、Basemark では +3%、SotTR (Shadow of the Tomb Raider) では +1fps (0-4%) の性能向上を確認できたとコメントしている。  

{{< ref >}}

 * [ASCII.jp：「Radeon RX 6800 XT/6800」で強いRadeonが久々に戻ってきた！【前編】 (2/7)](https://ascii.jp/elem/000/004/034/4034588/2/)
 * [西川善司の3DGE：「Radeon RX 6000」詳報。高性能の鍵となる「Infinity Cache」と「Smart Access Memory」の仕組みとは](https://www.4gamer.net/games/461/G046171/20201124135/)

{{< /ref >}}
