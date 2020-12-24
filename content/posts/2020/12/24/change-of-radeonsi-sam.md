---
title: "RadeonSI では Smart Access Memory をオプションで切り替える方式に、 Zen 3 + RDNA 2 ではデフォルトで有効"
date: 2020-12-24T22:53:44+09:00
draft: true
tags: [ "RadeonSI","Smart_Access_Memory" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

オープンソースで開発される AMD GPU の OpenGLドライバー **RadeonSI** に、*Smart Access Memory* に関する新たなパッチが投稿された。  

 * [util,ac,radeonsi: add Zen3 detection, disable SAM w/out Zen3 & gfx10.3, add color interpolation fixes (!8225) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/8225)

内容は、ドライバー内の GPU情報に *Smart Access Memory* の値を追加。有効とする判定に *Above 4G Decoding* が有効かつ、*Zen 3 + RDNA 2* であること、というものを用いている。  
パッチを投稿した AMD の [Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏は、(*Zen 3 + RDNA 2* 以外の) システムでは、多くの人が性能の低下を体験している、とコメント。[^sam-perf]  
それをデフォルトで回避すると同時に、ユーザーが選択できるよう driconf に *Smart Access Memory* のオプションを追加している。  
{{< link >}} [Mesa3Dドライバーの DriConf について | Coelacanth's Dream](/posts/2020/11/28/driconf/) {{< /link >}}

[^sam-perf]: [util,ac,radeonsi: add Zen3 detection, disable SAM w/out Zen3 & gfx10.3, add color interpolation fixes (!8225) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/8225/diffs?commit_id=bd0d5bad5f842debd2e868f35a735f6416d8b044)

しかし、*Smart Access Memory* を無効と言っても、パッチを読む限り、重要な要素である CPU が GPU の VRAM 全範囲に一度にアクセス可能という点では変わりはなく、以前行なわれた *Smart Access Memory* に向けた最適化を無視する (以前と同様の状態にする) というように見られる。  
{{< link >}} [RadeonSI/RADVドライバーに Smart Access Memory に向けた最適化が行なわれる | Coelacanth's Dream](/posts/2020/12/07/radeonsi-sam-optimization/) {{< /link >}}
*Zen 3 + RDNA 2* 以外のシステムでは最適化が逆に働いてしまったのかもしれない。  

*Smart Access Memory* の中身である *Resizeable PCI BAR* は、2017年には Linux Kernel に実装されていたが、  
{{< link >}} [AMD Smart Access Memory 再調査　―― Bulldozer世代の CPU/APU でも有効可能 | Coelacanth's Dream](/posts/2020/12/05/amd-sam-fact/) {{< /link >}}
性能への最適化やサポートの強化は最近になって集中的に行なわれており、[^resizeable-bar]


[^resizeable-bar]: [[PATCH 0/7] amdgpu, pci: improved BAR resizing support](https://lists.freedesktop.org/archives/amd-gfx/2020-December/057339.html)

