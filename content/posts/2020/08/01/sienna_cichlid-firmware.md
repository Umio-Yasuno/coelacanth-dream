---
title: "Radeon™ Software for Linux® 20.30 が公開 & ファームウェアの雑な読み方"
date: 2020-08-01T00:19:19+09:00
draft: true
# tags: [ "Sienna_Cichlid", "Navi21" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

{{< pindex >}}

 * [ファームウェアの雑な読み方](#how-to-read)

{{< /pindex >}}

AMD が、2020/07/30付で、**Radeon™ Software for Linux® 20.30** を公開した。  
{{< link >}} [Radeon™ Software for Linux® 20.30 Release Notes | AMD](https://www.amd.com/en/support/kb/release-notes/rn-amdgpu-unified-linux-20-30) {{< /link >}}


で、それに *Sienna Cichlid* のファームウェアが同梱されていた。  
そのファームウェアの中には、*${ASIC_NAME}_gpu_info.bin* が含まれ、それには GPU ASIC の詳細情報が記述されている。  
*${ASIC_NAME}_gpu_info.bin* は GFX9 世代から導入されたものであり[^gpu-info-bin-gfx9]、それはオープンソースである Linux Kernel とその中の AMDGPUドライバー から外れ、{{< comple >}} それだけでなく、AMDGPUファームウェアすべてがそうだが {{< /comple >}} バイナリのみのクローズドソースで提供される。  
だから、本来読むことは難しく、読むというよりは解析と言えるのだが、ある値がどの文字列に対応するか、意味するかはパーサーがある AMDGPUドライバーに記述されており[^gpu-info-parse]、そこから解析することはそう難しくはない。  
しかし、クローズドソースであるバイナリファイルの解析というのはリスクが存在し、そこから情報を得ても、次のバージョンでは平気で変更される可能性がある。  
クローズドソースなのだから、どこかしら変更があったとしても提供する側としては「知らない」と言えばそれまでだ。  
{{< comple >}}少し話はズレるが、ChangeLog や ReadMe も一緒に提供してくれないと、される側としては更新内容を知ることが出来ない。自分が遭遇したバグが修正されたのかどうかが不明なままだ。{{< /comple >}}  
言ってしまえば、信頼できる情報源としては取り扱えない。  

[^gpu-info-bin-gfx9]: [drm/amdgpu: export more gpu info for gfx9](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=408bfe7c3c5d036947b509356f494dc6b46025ff)
[^gpu-info-parse]: [amdgpu_device.c\amdgpu\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/amdgpu_device.c?h=amd-staging-drm-next&id=997ba18547129e4acb43706ddab29757930a495f#n1674)

<!--
それに、AMD が何でクローズドソースで提供しているかを考えれば、そりゃ IP保護やライセンス、機密保持のためだろう。だから、自分としてはそういったものを解析するような行為はあまり積極的にはしない。  
じゃあ何でこんな記事を書いてるんだと自分でも思うが、知ってしまったからだ。  
*「情報はフリーになるたがる」* というどこかで聞いたような言葉がある。その言葉は元々 *「情報はフリーになりたがる」* だったらしく[^free]、自分はその思想を輝かしく思う。  
だからフリーで広めるし、個人が独占するくらいなら一次情報元もそれの読み方も公開する。  
自分はタイミングをうまく読めるエンターテイナーでもない。  
ただ、同時に情報を守りたい、クローズドソースで提供する側の立場も理解はしているつもりだから、一言やめてくれと提供する側に言われればこの記事は消すし、これまでのようにオープンソースコードをネタにしていく。
-->

## ファームウェアの雑な読み方 {#how-to-read}

前置きが長くなったが、前述したようなリスクがあるため、*Sienna Cichlid* のファームウェアから何が読めるかではなく、*Navi10* の `${ASIC}_gpu_info.bin` を元に読み方のみを記す。  
また、自分も深くは理解していないため、解析の結果こうなっている、というだけであり間違っている可能性も当然存在する。  
そのため、あくまで参考、自己判断としていただきたい。  

[^free]: [「情報はフリーになるべきだ」から「情報はフリーになりたがる」への変遷にみる「モノの式神化」 (1/2)](https://blogos.com/article/92941/)


`navi10_gpu_info.bin` は [linux-firmware.git](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git) レポジトリに置いてある。  
{{< link >}} [navi10_gpu_info.bin « amdgpu - kernel/git/firmware/linux-firmware.git](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git) {{< /link >}}
`hex dump` の結果もあるが、それでは読みにくいため一度ローカルファイルに置き、[Ghex](https://wiki.gnome.org/Apps/Ghex) といったソフトウェアを使うのがいいだろう。Windows とか macOS はわからん。  

パーサー部分のソースコードを読むに、オフセットを経てから GPU ASIC のそれぞれ対応した値となっている。[^parser_1]  
`navi10_gpu_info.bin` は途中に大量の `00` が並んでおり、これがオフセットなのだろう。  
そして、`オフセット: 0x100` からは以下の表のように *なっていた* 。  

[^parser_1]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/amdgpu_device.c?h=amd-staging-drm-next&id=997ba18547129e4acb43706ddab29757930a495f#n1658>

| hex offset | |
| :-- | :--: |
| 0x100 | max\_shader\_engines |
| 0x105 | max\_cu\_per\_sh |
| 0x108 | max_sh_per_se |
| 0x10C | max_backends_per_se |
| 0x110 | max_texture_channel_caches |
| 0x115 | max_gprs |
| 0x118 | max_gs_threads |
| 0x11C | gs_vgt_table_depth |
| 0x121 | gs_prim_buffer_depth |
| 0x125 | double_offchip_lds_buf |
| 0x128 | wave_front_size |
| 0x12C | max_waves_per_simd |

<!--
| 0x130 | max_scratch_slots_per_cu |
| 0x134 | lds_size |
-->

*Navi10* の場合、`max\_shader\_engines` が `0x2` 、`max\_cu\_per\_sh` が `0xA` 、`max_sh_per_se` が `0x2`、 `max_backends_per_se` が `0x8` となっている。  
これを言い換えると、SE(ShaderEngine) 2基、SEあたりの SH(ShaderArray) 2基、SHあたりのCU 10基、SEあたりのRB(RenderBackend)数 8基、となる。  
[pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp) に記されたスペックと一致するため、この対応表が合っているようには思うが、あくまで参考とし、自己判断を行なってもらいたい。  

[^pal^navi10]: [pal/ndDevice.cpp at ea5db60841dab7d067f5010f28a980ef222bdf81 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/ea5db60841dab7d067f5010f28a980ef222bdf81/src/core/os/nullDevice/ndDevice.cpp#L944)
