---
title: "ChromiumOSへのパッチから Alderlake-S / Alderlake-P を確認"
date: 2020-05-02T05:30:43+09:00
draft: false
tags: [ "Alder Lake", ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU" ]
noindex: false
---

ChromimuOS の Verified Boot[^1] に関するパッチから *Alderlake-S* 、*Alderlake-P* の名が確認された。  
{{< link >}}[flashrom: Add flashrom support for Alderlake (I17da991d) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/flashrom/+/2166876){{< /link >}}
末尾の "S" は基本デスクトップ向けプロセッサであることを意味するが、 "P" は不明。  
ただ気になるのは、ここでのIDはチップセットのホストブリッジを指し、他のプロセッサではわざわざどのプラットフォーム向けプロセッサであるかを記述していない。  
つまり "S"、 "P" がプラットフォームを意味するにしても、*Alderlake* とひと括りにせず、2つを記述していること不自然に見える。  

[^1]: [Verified Boot - The Chromium Projects](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot)

もう1つ *Alderlake* のGPIOチップセットIDを追加するパッチが投稿されているが、  
{{< link >}}[crossystem: add support for ADL gpiochip (I7c8ead83) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/vboot_reference/+/2167671){{< /link >}}
IDが他から外れた `INT105x` となっており、ここからも *Alderlake* が異色のプロセッサであることが窺える。  

前回の Intel拡張命令リファレンスのアップデートにおいて、*Alder Lake* の名が追加されていたが、*Tiger Lake* が対応する命令と一致しないことから、  
*Alder Lake* はデスクトップ向けやモバイル向けといったプロセッサの後継、新世代ではなく、また別の用途向けのプロセッサである可能性が考えられる。  
{{< link >}}[Intel、拡張命令リファレンスをアップデート (Sapphire Rapids /Alder Lake /ハイブリッドプロセッサ) | Coelacanth's Dream](/posts/2020/04/01/intel-isa-extensiton-update-sapphirerapids-alderlake/){{< /link >}}
そうなると、上記の "S" がこれまでと同様の意味を持つどうかも勘繰ってしまう。  

まだまだ *Alder Lake* は謎に包まれており、今後の動きが気になる所だ。  
引き続き情報を追っていきたい。
