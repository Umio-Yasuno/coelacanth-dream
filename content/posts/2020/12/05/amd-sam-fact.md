---
title: "AMD Smart Access Memory 再調査　―― Bulldozer世代の CPU/APU でも有効可能"
date: 2020-12-05T20:28:15+09:00
draft: false
tags: [ "Smart_Access_Memory", ]
# keywords: [ "", ]
categories: [ "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

約 1ヶ月前に検証した機能、*AMD Smart Access Memory* だが、思う所があったため、再度情報を調査、整理することとした。  
{{< link >}} [Linux環境と Ryzen 5 2600、Radeon RX 560 で "Smart Access Memory" 機能を試す | Coelacanth's Dream](/posts/2020/11/05/linux-amd-smart-access-memory/) {{< /link >}}
性能については自環境を特に更新してないため見送る。  

## Linuxには 2017年に実装

前回は何年前と書いたが、Linux Kernelのレポジトリ、パッチアーカイブを調査する中で、*Smart Access Memory* の実態である `Resizeable PCI BAR` を実装するパッチは 2017/03 に投稿されていた。  

 * [Resizeable PCI BAR support V3](https://lists.freedesktop.org/archives/amd-gfx/2017-March/006316.html)
    * [[PATCH 1/4] PCI: add resizeable BAR infrastructure v3](https://lists.freedesktop.org/archives/amd-gfx/2017-March/006319.html)
    * [[PATCH 4/4] drm/amdgpu: resize VRAM BAR for CPU access](https://lists.freedesktop.org/archives/amd-gfx/2017-March/006320.html)

{{< ins >}}

PCI部への最初のパッチは 2015/12 に投稿されていた。  
{{< link >}} [Resizeable PCI BAR support - Patchwork](https://patchwork.kernel.org/project/linux-pci/patch/5662C61F.50303@amd.com/) {{< /link >}}
だがその時は組み込まれず、AMD GPUドライバーに合わせた 2017/03 にまたパッチが投稿されたようだ。  
{{< link >}} [[1/5] PCI: add resizeable BAR infrastructure v2 - Patchwork](https://patchwork.kernel.org/project/linux-pci/patch/1488800428-2854-1-git-send-email-deathsimple@vodafone.de/) {{< /link >}}

その後もバージョンを重ね、Linux Kernel のレポジトリを検索したところでは、2017/12 頃メインラインに組み込まれている。  
{{< link >}} [Merge branch 'drm-next-4.16' of git://people.freedesktop.org/~agd5f/linux into drm-next](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v5.10-rc6&id=9c606cd4117a3c45e04a6616b1a0dbeb18eeee62) {{< /link >}}

{{< /ins >}}

一連のパッチを投稿した Christian König 氏は AMD GPU のオープンソースドライバー等を長年担当しているソフトウェアエンジニアである。[^christian]  
氏は Resizeable PCI BAR を実装する構想を持ってから、1年以上経ってようやく完成させることができた、とコメントしている。  
従来の PCI BAR は 32-bit システムとの互換性を理由に、CPU から GPU へ一度にアクセス可能なサイズを 256MB としていたが、Resizeable PCI BAR をサポートすることにより、GPU が持つ数GBのローカルメモリに CPU がアクセス可能となる。  
また、Christian König 氏はドライバー側のリサイズ機能と同時に、AMD Family 15h (*Bulldozer* 世代の CPU/APU) で 64-bit bar を有効する機能も追加しており、。現時点のコードを見てもそれは残っている。[^fam15h-large-bar]  
{{< link >}} [[PATCH 3/4] x86/PCI: Enable a 64bit BAR on AMD Family 15h (Models 30h-3fh) Processors](https://lists.freedesktop.org/archives/amd-gfx/2017-March/006318.html) {{< /link >}}

[^christian]: [Christian König – Senior Software Development Engineer – AMD | LinkedIn](https://de.linkedin.com/in/christian-k%C3%B6nig-35b7bbaa)<br> [FOSDEM 2020 - Christian König](https://archive.fosdem.org/2020/schedule/speaker/christian_konig/)
[^fam15h-large-bar]: [linux/fixup.c at 3789af9a13e5561738c0f2114e3a5e22c843ca3e · torvalds/linux](https://github.com/torvalds/linux/blob/3789af9a13e5561738c0f2114e3a5e22c843ca3e/arch/x86/pci/fixup.c#L684)


*RDNA /GFX10* 世代以降で有効にするパッチは 2020/05 に投稿されていた。[^gfx10-large-bar]  
それと同時期に一度、`Above 4G decoding` のような 64-bit bar が無効になっている場合、Linux Kernel にメッセージを出力するパッチが投稿されたが、多くの人にとってはスパムになってしまうだろう、という理由から組み込まれずに取り下げられている。[^drop]  
また、APU {{< comple >}} 厳密には APU 内の統合 GPU {{< /comple >}} に対してはこの機能が有効とされないよう判定が行なわれている。  

 >        	if (!(adev->flags & AMD_IS_APU)) {
 >        		r = amdgpu_device_resize_fb_bar(adev);
 >        		if (r)
 >        			return r;
 >        	}
 >
 > {{< quote >}} [gmc_v9_0.c\amdgpu\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/gmc_v9_0.c?h=amd-staging-drm-next&id=187453ff5924a98042fe2a5fb8aeacf51892d545#n1246) {{< /quote >}}

[^gfx10-large-bar]: [[PATCH 1/2] drm/amdgpu: resize VRAM BAR for CPU access on gfx10](https://lists.freedesktop.org/archives/amd-gfx/2020-May/049749.html)
[^drop]: [[PATCH 2/2] drm/amdgpu: Advise if unable to resize BAR](https://lists.freedesktop.org/archives/amd-gfx/2020-May/049752.html)

自分が *Smart Access Memory/Resizeable PCI BAR* 、`Above 4G decoding` 等の機能を知ったきっかけは、AMD GPUドライバーの開発者である [Alex Deucher](https://gitlab.freedesktop.org/agd5f) 氏が、[Phoronix](https://www.phoronix.com/scan.php?page=home) による **Radeon RX 6000シリーズ** についての記事に投稿したコメントだったが、  
こうしたドライバーへの実装を行なった開発者方にとって、ユーザー層がこうした機能に今更騒いでいるのはいささか不思議に思われたかもしれない。  

改めて AMD 公式サイトの *Smart Access Memory* を解説したページを見ると、{{< comple >}} ある意味で {{< /comple >}} 丁寧に、  
**従来型のWindowsベースのPCシステムでは** (引用元: [AMD Smart Access Memory | AMD](https://www.amd.com/ja/technologies/smart-access-memory) )、  
とある。 AMD としては Linux 等のシステムでは既に実装していることは分かっており、それを Windowsにも実装する時、マーケティング的な価値が十分にあると考えたのだろう。  

また、KTU こと加藤勝明 氏による検証で、Intel環境でも *Smart Access Memory/ Resizeable PCI BAR* を有効可能であり、効果が確かに出ていることが判明している。Linux でもこれは同様と思われる。  
{{< link >}} [ASCII.jp：Intel環境でもRadeon RX 6000シリーズのSmart Access Memoryが使えるのか検証してみた (1/2)](https://ascii.jp/elem/000/004/036/4036051/) {{< /link >}}

前回も書いたが、Linux環境で *Smart Access Memory / Resizeable PCI BAR* が有効にされているかは、  

     $ AMD_DEBUG="info" glxinfo -B | grep vram_vis_size

または以下のコマンドで確認できる。上記の場合は値が VRAM と同じサイズとなっていれば、下記の場合は `BAR=` の後の値が VRAM と同じサイズであれば有効化されている。  

    % dmesg | grep "Detected VRAM"

## 個人的結論

 * Smart Access Memory は PCI Express に存在する Re-Size/Resizeable PCI BAR を有効活用したもの
 * Windowsの実装は不明
 * AMD の言う VRAM への CPU フルアクセスは Linux では既に実装されていた
 * それは Bulldozer(15h) 世代 から有効可能
