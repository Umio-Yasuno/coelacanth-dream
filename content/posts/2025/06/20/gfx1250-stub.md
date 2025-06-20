---
title: "LLVM に gfx1250 が \"仮に\" 追加される ――Wave32 に対応して MFMA 系命令をサポートしない CDNA APU?"
date: 2025-06-20T19:57:33+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "LLVM", "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Stanislav Mekhanoshin 氏によって LLVM に新たな GFX ID、*gfx1250* が追加された。  
今のところ、これはただの仮サポート (stub) だとされている。  

 * [[AMDGPU] Initial support for gfx1250 target. by rampitec · Pull Request #144965 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/144965)

だがそれを置いても *gfx1250* は今後の AMD GPU を見る上でかなり興味深い存在である。  
まず、*gfx1250 (gfx12.5.0)* の GFX メジャーバージョンは RDNA 4 系と同じ GFX12 であり、Wave32 に対応しているが、WGP (Work-Group Processor) モードには対応せず、CU モードのみをサポートする。  
RDNA 系アーキテクチャにおける WGP モードと CU モードは主に LDS (Local Data Share) の割り当てに影響し、WGP モードではペアとなる 2基の CU の LDS 64KiB を合わせて割り当てることが可能になるが、CU モードでは常に CU 内の LDS のみに割り当てる。  

また、CDNA 系アーキテクチャ特有のパックド FP32 実行や SRAM ECC に対応する一方で、CDNA 系における行列演算専用命令である MFMA (Matrix fused-multiply-add) には対応せず、RDNA 3/4 が対応してる WMMA (Wave Matrix Multiply Accumulate) には対応している。  
他にもフルレートでの FP64 実行の機能フラグも *gfx1250* には設定されていない。  

RDNA 系アーキテクチャとして見ても、*gfx1250* は画像系命令のサポートやレイトレーシング関連命令のサポートを示す機能フラグが設定されていない。  
RDNA 3/4 の Dual issue、VOPD 命令もサポートしていないとされている。  

CDNA 4 アーキテクチャでは LDS を従来の 64KiB (32バンク) から 160KiB (64バンク) に増やしたが、*gfx1250* は 64KiB (32バンク) のままとされている。  

すべて "現時点の内容では" という言葉が前に付くが、*gfx1250* は CDNA と RDNA を融合したような存在となっている。  
以前に AMD の Jack Huynh 氏はそのようなアーキテクチャとなる UDNA の構想を語っていたが、*gfx1250* がそうなのだろうか？
データセンター向けの GPU アーキテクチャとして考えると、*gfx1250* はフルレート FP64 実行に必要なリソースを減らし、FP32 性能と推論処理性能に特化したものなのかもしれない。  

 * [AMD RDNA と CDNA の違いを考える | Coelacanth's Dream](/posts/2024/11/22/rdna-cdna-udna/)

さらに仮と推測に話なってしまうが、*gfx1250* が UDNA とすると、グラフィクス処理に必要な画像系命令やレイトレーシング関連命令はオプション扱いになるのかもしれない。  
RDNA 3 では CU 内の SIMDユニットに 2x SIMD32 という構成を取り、Dual issue、VOPD 命令に対応することで、クロックあたりの FP32 性能を倍増させたが、これらはパックド FP32 実行に置き換えられるのだろうか？  

