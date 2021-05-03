---
title: "メモリバス幅 32-bit の Polaris12 をベースにした GPU が存在するらしい"
date: 2021-05-04T01:34:01+09:00
draft: false
tags: [ "GFX8", ]
# keywords: [ "", ]
categories: [ "AMD", "GPU", "Hardware" ]
noindex: false
# summary: ""
---

しばらくぶりのメモリバス幅 32-bit AMD GPU であり、ほぼ確実に Polaris世代の製品でも最も下に位置する Discrete GPU だろう。  

[linux-firmware.git](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/about/)レポジトリに、メモリバス幅を 32-bit とする *Polaris12* ベース GPU のためのファームウェアが追加され、Linux Kernel における AMD GPU ドライバーにもそのファームウェアに対応するパッチが投稿されている。  

*Polaris12* は *GFX8* 世代の dGPU で、Polaris世代 ( *Polaris10/11/12* ) の中では最も小規模の GPUチップ。  
総 SE (ShaderEngine) 数は 2基、SE あたりの CU数は 5基で総数 10基、総 RenderBackend 数は 4基 (ROP: 16基)。メモリインターフェイスは GDDR5 128-bit (4ch x 32-bit) に対応し、その点では *Polaris11* と同じだが、メモリサイドキャッシュである L2キャッシュはブロック (メモリチャネル) あたり 128KB となっており、他 Polaris GPU の 256KB よりも半分のキャッシュサイズである。[^polaris12-l2]  

[^polaris12-l2]: [ac/gpu_info: conceal L2 cache sizes (29ca71e1) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/29ca71e10e58077fb847a914b5051e69a4add352)

 * [[PATCH] drm/amdgpu: add new MC firmware for Polaris12 32bit ASIC](https://lists.freedesktop.org/archives/amd-gfx/2021-April/062711.html)
 * [amdgpu: add new polaris 12 MC firmware](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/?id=3f23f5125b1fef5ed2103c0236a5657966e30e4d)

| GFX8 | Polaris10 | Polaris11 | Polaris12 |
| :--  | :--: | :--: | :--: |
| ShaderEngine | 4 SE | 2 SE | 2 SE |
| CU per SE / Total CU | 9 / 36 CU | 8 / 16 CU | 5 / 10 CU |
| Toatl RB | 8 RB<br>(32 ROP) | 4 RB<br>(16 ROP) | 4 RB<br>(16 ROP) |
| Memory Bus width | 256-bit | 128-bit | 128-bit |
| L2cache size | 2048 KB | 1024 KB | 512 KB |

メモリバス幅が 32-bit の場合、通常とは違う特殊なメモリコントローラーに関するファームウェアが必要としており、その理由については GPU のメモリコントローラーが基本 64-bit単位であることが考えられる。  

dGPU であり、メモリコントローラーも 64-bit単位でありながら 32-bit構成を採ることはまさに特異だと言える。他の *Polaris12* をベースにする GPU製品を見ても、メモリバス幅は小さくて 64-bit となっている。  
TechPowerUp の [GPU Specs Database](https://www.techpowerup.com/gpu-specs/?buswidth=32%20bit&sort=name) を参照してみたが、ATI/AMD GPU で最後に 32-bit構成の dGPU がリリースされたのは 2007/03 の **ATI Mobility Radeon X2300 HD** であり、実に 14年前となる。オープンソースコードから得られる AMD GPU の情報は GCN世代からが主ということもあるが、正直全く知らない GPU だ。  
NVIDIA GPU では 2006/09 の **GeForece Go 7200** が最後の 32-bit構成 dGPU らしい。[^tpu-32bit-gpu]  

[^tpu-32bit-gpu]: <https://www.techpowerup.com/gpu-specs/?buswidth=32%20bit&sort=name>

メモリバス幅が 32-bit だと、GDDR5 7Gbps メモリチップを使用しても、ピークメモリ帯域は 24GB/s。  
単純に比較すれば、メモリインターフェイスに DDR4 128-bit を採る APU では、2133MT/s のメモリモジュールでもピークメモリ帯域は 32GB/s となるから、dGPU で GDDR5 32-bit というのは明らかに小さく、性能を追い求める向きは無いと見られる。恐らくは省電力で映像出力さえこなしてくれればいいような組み込み向けや、グラフィクス機能を持たない CPU と組み合わせることを想定した製品だろう。PCIeカード型ではなく、オンボード実装が主と思われる。  

そういう訳でメモリバス幅 32-bit の *Polaris12* GPU が存在しても、一般に出回る可能性は低いと思われるが、32-bit幅の AMD GPU が 14年振りに登場し、さらには 14nmプロセスで製造される *Polaris12* がそのベースとなる。  
マイニング需要、半導体製品の供給不足が影響していることもあるが、Polaris世代の GPU は長く、かつ幅広く活躍する働き者だと言えるだろう。  
{{< link >}} [働き者のPolaris | Coelacanth's Dream](/posts/2020/03/11/polaris-hard-worker/) {{< /link >}}


{{< ref >}}
 * [pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp)
 * [The Polaris Architecture | - polaris-whitepaper.pdf](https://www.amd.com/system/files/documents/polaris-whitepaper.pdf)
{{< /ref >}}
