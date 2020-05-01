---
title: "Intel、Rocket Lake 一連の Linux Kernelパッチを投稿"
date: 2020-05-02T04:22:34+09:00
draft: false
tags: [ "Rocket_Lake", "Gen12" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "GPU" ]
noindex: false
---

Intel は次世代プロセッサ *Rocket Lake* をサポートする Linux Kernelパッチを投稿した。  
OSSのコードに *Rocket Lake* の名が出るのはこれが初だ。  
{{< link >}}[[Intel-gfx] [PATCH 00/23] Introduce Rocket Lake](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238498.html){{< /link >}}

GPU部は *Tiger Lake* と同世代、[Gen12](/tags/gen12)であることが明記されており、ディスプレイ出力が4つすべてコンボPHYを使用する、つまりは4つのType-C/TB3(4?)からディスプレイ出力が可能なものと考えられる。  

追加された PCI ID は6つ。 ( `0x4C80` 、`0x4C8A` 、 `0x4C8B` 、`0x4C8C` 、`0x4C90` 、`0x4C9A` )  
{{< link >}}[[Intel-gfx] [PATCH 01/23] drm/i915/rkl: Add RKL platform info and PCI ids](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238499.html){{< /link >}}

また、これまでのプラットフォームとは異なるメモリ特性を有するとしている。  
{{< link >}}[[Intel-gfx] [PATCH 06/23] drm/i915/rkl: Update memory bandwidth parameters](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238505.html){{< /link >}}

## PCH
一連のパッチにはPCHに関するパッチも含まれており、*Comet Lake PCH* と *Tiger Lake LP PCH* 両方のプラットフォームに *Rocket Lake* が今後追加されることを示唆している。  
{{< link >}}[[Intel-gfx] [PATCH 05/23] drm/i915/rkl: Add PCH support](https://lists.freedesktop.org/archives/intel-gfx/2020-May/238515.html){{< /link >}}

*Tiger Lake LP PCH* はモバイル向けとなるが、*Comet Lake PCH* に関してはデスクトップ向けも含まれるため、*Rocket Lake* がデスクトップ市場に登場する可能性が考えられるだろう。  
また、*Rocket Lake* のGPUは *Tiger Lake* と同世代であることから、CPU部で差別化を図るはずだ。果たしてそれがどこまでのものとなるかは気になる所。
