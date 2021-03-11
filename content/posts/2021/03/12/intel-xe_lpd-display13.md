---
title: "Intel、Display13 改め \"XE_LPD\" をサポートするパッチを投稿　―― 今後は独立して各 IP が更新されるように"
date: 2021-03-12T02:09:49+09:00
draft: false
tags: [ "Gen13", ]
# keywords: [ "", ]
categories: [ "Intel", "GPU", "Hardware" ]
noindex: false
# summary: ""
toc: false
---

Intel GPU のディスプレイエンジンIP *XE_LPD* をサポートするパッチが Linux Kernel (intel-gfx) に投稿された。  
パッチの中身としては、以前投稿された *Display13* をサポートするパッチと同じであり、パッチのタイトルやコード中のコメントにあった *Display13* を *XE_LPD* にリネームしたのがほとんどとなる。  
{{< link >}} [Linux Kernel に Intel "Display13" をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/29/intel-display13/) {{< /link >}}
パッチの目的はむしろ、新たなディスプレイIPをサポートすることよりも、将来の GPU のため、IP のバージョン管理を変更することにある。  

 * [[Intel-gfx] [PATCH v2 00/23] Separate display version numbering and add XE_LPD (version 13)](https://lists.freedesktop.org/archives/intel-gfx/2021-March/262071.html)

## 今後 GPUコア、メディア、ディスプレイは独立して更新するように

Intel の GPUアーキテクチャは基本 *Gen11* とか *Gen12* といったように、単一のバージョン番号で語られることが多いが、ハードウェア的には GPUコア、メディアエンジン、ディスプレイエンジンで分かれている。  

{{< figure src="/image/2021/03/12/dg1-ip-ver.webp" title="DG1 IP version" caption="画像出典: [Intel® Iris® Xe MAX Graphics Open Source Programmer's Reference Manual For the 2020 Discrete GPU formerly named \"DG1\" Volume 4: Configurations - intel-gfx-prm-osrc-dg1-vol04-configurations.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-dg1-vol04-configurations.pdf)" >}}

[DG1](/tags/dg1) はまだそれらのバージョン番号が統一されており、それまでも基本そうだったが、今後の Intel GPU では別々に独立してハードウェアIPが更新されるようになるため、そうもいかなくなった。  
バージョンが統一されていない特殊な例として、GPUコアは *Gen9 アーキテクチャ* だが、ディスプレイエンジンは HDMI 2.0 に対応していた、*Gemini Lake* が存在する。  

今後は、「GPUコア (graphics version)」、「メディアエンジン (media version)」、「ディスプレイエンジン (display version)」を、ドライバー側がまず GPU の DeviceID (PCI ID) を読み取り、それから判断するのではなく、MMIOレジスタからそれらバージョンを直接読み取れるよう変更する予定にあるとのこと。  
これらはハードウェアチームから提案されたものだという。  

また、*XE_LPD* という呼称はマーケティングにも導入するとしている。  
*Gen12LP アーキテクチャ* も、発表の場では *{{< xe class="lp" >}}アーキテクチャ* と呼ばれており、他の {{< xe >}}系列も *{{< xe class="hp" >}}、{{< xe class="hpg" >}}、{{< xe class="hpc" >}}* として紹介されていたため、マーケティングでは *XE_LPD* の方が自然かもしれない。  

パッチでは同時に、 *間もなく* 登場するハードウェアプラットフォームでは、GPUコアは *Gen12* 、ディスプレイエンジンは新たな *XE_LPD/Display13* が採用されることを明らかにしている。  

 >        The next hardware platform we'll be upstreaming (very soon!) will have a
 >        display version of 13 and a graphics version of 12, so let's just the
 >        first step of breaking out DISPLAY_VER(), but leaving the rest of
 >        INTEL_GEN() untouched for now.  For now we'll automatically
 >        derive the display version from the platform's INTEL_GEN() value except
 >        in cases where an alternative display version is explicitly provided in
 >        the device info structure.
 >
 > {{< quote >}} [[Intel-gfx] [PATCH v2 02/23] drm/i915: Add DISPLAY_VER()](https://lists.freedesktop.org/archives/intel-gfx/2021-March/262073.html) {{< /quote >}}

それがディスクリートGPUなのか、それともハードウェアプラットフォームという呼び方から統合グラフィクスなのかも、まだはっきりしないが、*間もなく (very soon!)* と言うくらいなのだから、そう期待の熱が冷めない内にパッチか正式リリースの形で発表してくれることだろう。  

*Display13* 改め *XE_LPD* の主な新機能には、16K 解像度への対応、仮想機能 (Intel VT-d) の対応強化、パワードメインの変更、Watermak 数の増加による電力効率の向上等が挙げられる。  
ディスプレイエンジンの更新ということで、体感しやすい性能への影響は小さいが、解像度や仮想機能、電力効率向上と、着実にユーザーの利便性を向上してくれる機能が並んでいる。  

