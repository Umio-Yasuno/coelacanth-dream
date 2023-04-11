---
title: "LLC をサポートせず、ADM/L4キャッシュをサポートする Meteor Lake GPU"
date: 2023-04-12T00:41:17+09:00
draft: false
categories: [ "Intel", "GPU" ]
tags: [ "Linux_Kernel", "Meteor_Lake" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Fei Yang 氏より、intel-gfx メーリングリストに公開されたパッチにて *Meteor Lake GPU* では LLC (Last Level Cache) をサポートせず、新たに ADM/L4キャッシュをサポートすることが明かされた。  

 >         On MTL, GT can no longer allocate on LLC - only the CPU can.
 >         This, along with addition of support for ADM/L4 cache calls a
 >         MOCS/PAT table update.
 >
 > {{< quote >}} [[Intel-gfx] [PATCH 2/9] drm/i915/mtl: Define MOCS and PAT tables for MTL](https://lists.freedesktop.org/archives/intel-gfx/2023-April/323891.html) {{< /quote >}}

 * [[Intel-gfx] [PATCH 1/9] drm/i915/mtl: Set has_llc=0](https://lists.freedesktop.org/archives/intel-gfx/2023-April/323893.html)
 * [[Intel-gfx] [PATCH 2/9] drm/i915/mtl: Define MOCS and PAT tables for MTL](https://lists.freedesktop.org/archives/intel-gfx/2023-April/323891.html)

従来の Intel iGPU では CPU と GPU で LLC を共有可能な構成となっており、MOCS (Memory Object Control State) の値に応じて GPU 側のキャッシュに割り当てることが可能だった。  
それが *Meteor Lake GPU* では、LLC は CPU のみが使用するようになり、GPU 側のキャッシュに割り当てることができなくなる。  
CPU と GPU で共有可能なキャッシュを持たないという点で Intel dGPU、AMD APU のキャッシュ構成に近くなったと言える。  
余談だが、少し前に **Ryzen 9 7950X3D** の内蔵 GPU 性能が **7950X** の 4倍という話が流れたが (実際はそのレビュワーが以前の検証データを使い回したことによる勘違い)[^7950x3d]、AMD APU のキャッシュ構成を知っていれば間違いに気付くことができた。  

[^7950x3d]: [Ryzen 9 7950X3Dの内蔵GPUは非X3Dの4倍速い？【3D V-Cache】 - ニコニコ動画](https://www.nicovideo.jp/watch/sm41891118)

そして *Meteor Lake GPU* では ADM とも呼ばれる L4キャッシュをサポートする。  
パッチでは ADM が何の略称なのか、ADM/L4キャッシュのサイズについては触れていない。  

Intel iGPU における L4キャッシュというと、*Gen9 アーキテクチャ* でサポートしていた eDRAM が思い出される。  
しかし eDRAM がメモリサイドキャッシュ、SoC 全体で使用可能なキャッシュであったのに対し、ADM/L4キャッシュはあくまで GPU 側でも割り当て可能キャッシュとなっている。従来の LLC の役割が移ったキャッシュなのだろう。  
また、GPU 専用のキャッシュではなく、CPU からも使用可能ではないかと思われる。  

気になるのは ADM/L4キャッシュが *Meteor Lake* のどこに搭載されているかだが、恐らく Base Tile と思われる。  
Intel は *Meteor Lake* のタイル構成を公開しており、Base Tile 上に GPU, SoC, CPU, IO Extender Tile を積層する構成となっている。ADM/L4キャッシュを搭載した Tile をオプションで追加可能な構成になっているようには見えない。  
メモリコントローラーが搭載されている SoC Tile の可能性もあるが、HotChips34 で発表された SoC Tile の内部ブロックにキャッシュは載っていない。  

Intel は *Ponte Vecchio* にてすでにキャッシュの一部を Base Tile に搭載している。*Ponte Vecchio* の L3キャッシュは、Base Tile 内の L3キャッシュと、Base Tile 上に積層された RAMBO キャッシュを組み合わせる構成になっている。  
そのため、現時点では *Meteor Lake* の ADM/L4キャッシュが搭載されている可能性が高いのは Base Tile と考えられる。  

Base Tile にあると仮定すると、*Meteor Lake* の SKU 全体で Base Tile は共通なのか、GPU, CPU Tile の規模に応じて ADM/L4キャッシュサイズも調整されるのか、ADM/L4キャッシュサイズの違いが与える GPU, CPU 性能への影響……等、気になる点は色々出てくる。  

{{< ref >}}
 * [the-compute-architecture-of-intel-processor-graphics-gen9-v1d0-807584.pdf](https://www.intel.com/content/dam/develop/external/us/en/documents/the-compute-architecture-of-intel-processor-graphics-gen9-v1d0-807584.pdf)
 * [ISSCC 2022: Wie Intel für Ponte Vecchio 63 Tiles in ein Package bringt - Hardwareluxx](https://www.hardwareluxx.de/index.php/news/hardware/grafikkarten/58176-isscc-2022-wie-intel-fuer-ponte-vecchio-63-tiles-in-ein-package-bringt.html)
 * [Next Gen Client - Meteor_Lake_Hotchips _ Wilfred _ final_submit (1).pdf](https://hc34.hotchips.org/assets/program/conference/day2/Mobile%20and%20Edge/Meteor_Lake_Hotchips%20_%20Wilfred%20_%20final_submit%20(1).pdf)
{{< /ref >}}
