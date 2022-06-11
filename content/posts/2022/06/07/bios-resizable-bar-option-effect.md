---
title: "UEFI/BIOS の Resizable BAR オプションが与える影響"
date: 2022-06-07T21:48:24+09:00
draft: false
categories: [ "Software", "AMD", "GPU" ]
tags: [ "Smart_Access_Memory", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMDGPUドライバーのバグを報告するレポジトリである [drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd) に、System UEFI/BIOS の設定によって BAR (Base Address Register) 領域への書き込みが遅くなる問題が報告されている。  
報告者の [Tatsuyuki Ishi](https://github.com/ishitatsuyuki) 氏はベンダー非依存な NVIDIA Reflex の代替ミドルウェア [LatencyFleX](https://github.com/ishitatsuyuki/LatencyFleX) の開発者であり、Mesa3D RADVドライバーを始め多くの OSSプロジェクトに参加、開発を行っている。  

 * [BAR writes are slow with certain machine configurations (#1864) · Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues/1864)

## Above 4G Decoding, Resizable BAR, Smart Access Memory ... {#term}

先に用語整理。  
`Above 4G Decoding` は 4GB を超える MMIO (Memory Mapped IO) を許可、可能にする機能。  
`Resizable BAR` は PCIe の仕様にある機能であり、BAR のサイズを 64-bitアドレス範囲で指定、また可変させられるようになる。(基本 GPU VRAM、デバイスのローカルメモリに合わせられる)  
`Smart Access Memory` はほとんど AMD のマーケティング用語であり、CPU から VRAM がすべて見えている状態、BAR のサイズが VRAM 以上の状態を指している。  
ただ Mesa3D (RadeonSI/RADV) においては `Resizable BAR` 向けの最適化オプションを有効化した状態を `Smart Access Memory` と呼んでもいる。  
{{< link >}} [RadeonSI/RADVドライバーに Smart Access Memory に向けた最適化が行なわれる | Coelacanth's Dream](/posts/2020/12/07/radeonsi-sam-optimization/) {{< /link >}}

AMD AM4プラットフォームの UEFI/BIOS (AMD AGESA >1.1.0.0) では、`Above 4G Decoding [Enabled/Disabled]` と `Resizable BAR [Auto/Enabled/Disabled]` のオプションがある。  
`Above 4G Decoding [Enabled]` だけでも `Smart Access Memory` の状態になるが、AMD 公式サイトでは `Resizable BAR [Enabled]` も `Smart Access Memory` の有効に必要だとしている。[^sam-option]  
正直 `Resizable BAR` オプションで何が変更されるのかは不明であり、AMD や M/Bベンダーも詳細を公開していないが、このオプションが結果的に BAR への書き込み速度に影響する。  

[^sam-option]: [AMD Smart Access Memory | AMD](https://www.amd.com/en/technologies/smart-access-memory)

## BAR への書き込み速度 {#bar-write}

Tatsuyuki Ishi 氏は Vulkan API を用いて BAR の領域に書き込むようにしたベンチマークを開発し、検証した結果、`Resizable BAR [Enabled]` 設定時に書き込み速度が明確に遅くなることを報告している。  

ベンチマークのソースコードは以下のリンク先で公開されている。自分の理解では、と前置くと、当該ベンチマークではメモリタイプに `DEVICE_LOCAL | HOST_COHERENT` のフラグを立てることで BAR 領域を使うようにし、8MiB ごとに書き込み (zero fill)、その速度を測定している。  
ベンチマークでは VRAM の 90% を使うことを想定しており、実行するにはソースコードの Line 31 を変更して搭載 GPU の VRAM サイズに合わせる必要があると氏はコメントしている。2048 は 16GiB (2048 \* 8MiB) を想定した値となる。  

 * <https://gist.github.com/ishitatsuyuki/e86aa70879a8de8ed5b398b68b1ddfbc>

Tatsuyuki Ishi 氏の GIGABYTE X570 AORUS ELITE + Ryzen 3 3700X + RX 5700 XT を用いた検証では、速い状態で ~1.5ms、遅い状態で ~20ms と、書き込み速度に約 10倍の違いが出ている。UEFI/BIOS アップデート後に多少改善され、遅い状態で ~10ms となったが、まだまだ充分な速度ではないとしている。  
Tatsuyuki Ishi 氏は、X370 + RDNA 2 を用いた他のユーザーの検証でも同じ問題が再現できたことを報告している。  
自環境 (GIGABYTE A520 S2H + Ryzen 5 5600G + RX 560 4GB) で検証したところでは、速い状態 (`Resizable BAR [Disbaled]`) で ~2ms、遅い状態 (`Resizable BAR [Auto]`) で ~15ms という結果になった。RX 560 (Polaris11) という数世代前の AMDGPU でも再現できたことになる。  

Tatsuyuki Ishi 氏はこの問題に関して、AMDGPUドライバーのソースコードも調査している。  
氏の調査によれば `amdgpu_device_resize_fb_bar` 関数内の以下引用部にパッチを当てることで (恐らくは無効化、コメントアウトするものと思われる)、Linux Kernel側で BAR のリサイズ、再マッピングが行われ、問題が回避できる。  
System UEFI/BIOS で既に Resizable BAR (large BAR) が有効化、マッピングされている場合は、Linux Kernel側の処理をスキップするようになっている。  
また先に書いたように、UEFI/BIOS に `Resizable BAR [Disabled]` を設定することでも回避できる。  
自環境でも `Resizable BAR [Disabled]` の場合に、`pci_release_resource, pci_assign_unassigned_bus_resources` 関数が呼ばれたことによるログメッセージが出力されていることが確認できた。  
`Resizable BAR [Auto]` の場合は `(pci_resource_len(adev->pdev, 0) >= adev->gmc.real_vram_size)` が `true` になり、スキップされてしまうのだとは思うが、それ以上の詳細は自分には分からない。  

 > 			/* skip if the bios has already enabled large BAR */
 > 			if (adev->gmc.real_vram_size &&
 > 			    (pci_resource_len(adev->pdev, 0) >= adev->gmc.real_vram_size))
 > 				return 0;
 >
 > {{< quote >}} <https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/gpu/drm/amd/amdgpu/amdgpu_device.c?h=v5.18.2#n1199> {{< /quote >}}

`Resizable BAR [Enabled]` によって System UEFI/BIOS 側でマッピングされている場合に BAR への書き込み速度は遅くなり、`Resizable BAR [Disabled]` によって Linux Kernel側でリサイズ、再マッピング処理が行われた場合は速くなる。  
Tatsuyuki Ishi 氏は Windows環境でも問題が再現できるとしている。  
また、氏は Linux Kernel にパッチを適用し、BAR が問題無い範囲で自由にマッピングできるようにした調査を行ない、アドレス範囲が `0xfc00000000..0xfdffffffff` では上位半分のみが遅く、`0xfe00000000..0xffffffffff` では全体が遅くなることを発見している。  
そのため、厳密には System UEFI/BIOS、Linux Kernel どちらでマッピングするかではなく、どこにマッピングされるかで問題は発生している。  
このことから、マッピングのコンフリクトか PCIコントローラのバグではないかと氏は推測している。  
AMD の Alex Deucher 氏は当該 issue 内で、System UEFI/BIOS が誤った CPUキャッシュマッピングを設定しているのではと推測、プラットフォームベンダーに報告するとコメントしている。  
原因の判明にはもう少し時間が掛かるものと思われる。  

この問題がゲームのような実アプリケーションの性能にどれだけ影響するかも不明。  
BAR への書き込み速度と同様の影響があるのであれば、もっと以前から他で話題になっていいように思う。  
また Tatsuyuki Ishi 氏の環境も自環境も GIGABYTE M/B を用いており、この問題が他ベンダーの M/B、`Resizable BAR [Auto/Enabled/Disabled]` オプションを備える AMD AGESA >1.1.0.0 の UEFI/BIOS すべてでも発生するのかは判然としない。  

{{< ref >}}
 * [Vulkan Memory Types on PC and How to Use Them](https://asawicki.info/news_1740_vulkan_memory_types_on_pc_and_how_to_use_them)
 * [Using Vulkan® Device Memory - GPUOpen](https://gpuopen.com/learn/vulkan-device-memory/)
 * [Quick Heads up about something I discovered relating to Resizable BAR, you might be missing out on a huge performance uplift : linux_gaming](https://old.reddit.com/r/linux_gaming/comments/v58ts5/quick_heads_up_about_something_i_discovered/)
{{< /ref >}}
