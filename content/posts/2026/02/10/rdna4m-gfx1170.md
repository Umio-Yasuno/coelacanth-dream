---
title: "LLVM に RDNA 4m/GFX1170 APU のサポートが追加される ――専用の行列積和演算ユニットを削減か?"
date: 2026-02-10T01:06:19+09:00
draft: false
categories: [ "AMD", "APU" ]
tags: [ "LLVM", "GFX11" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

先日、AMD の Mirko Brkusanin 氏により、LLVM に RDNA 4m/GFX1170 のサポートを追加するパッチが公開された。  
何故か LLVM を Linux (Linux Kernel) と同一視する人がいるが、それぞれ別物である。  
LLVM は AMD GPU に主にコンパイラバックエンドとして、Linux Kernel はデバイスドライバとして関係している。  

 * [[AMDGPU] Define new target gfx1170 by mbrkusanin · Pull Request #180185 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180185)

 >         +     **GCN GFX11.7 (RDNA 4m)**
 >         +     -----------------------------------------------------------------------------------------------------------------------
 >         +     ``gfx1170``                 ``amdgcn``   APU   - cumode          - Architected                   *TBA*
 >         +                                                    - wavefrontsize64   flat
 >         +                                                                        scratch                       .. TODO::
 >         +                                                                      - Packed
 >         +                                                                        work-item                       Add product
 >         +                                                                        IDs                             names.
 >         +
 >
 > {{< quote >}} [[AMDGPU] Define new target gfx1170 by mbrkusanin · Pull Request #180185 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180185) {{< /quote >}}

GFX1170 は GFX バージョンとしては RDNA 3 系列 (GFX11xx) だが、ドキュメントでは RDNA 4m だとされている。  
ISA としても RDNA 3/GFX11 をベースとしている。  

 >         +def FeatureISAVersion11_7_0 : FeatureSet<
 >         +  !listconcat(FeatureISAVersion11_Common.Features,
 >         +    [FeatureSALUFloatInsts,
 >         +     FeatureDPPSrc1SGPR])>;
 >         +
 > {{< quote >}} [[AMDGPU] Define new target gfx1170 by mbrkusanin · Pull Request #180185 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180185) {{< /quote >}}

だが RDNA 3/GFX11, RDNA 3.5/GFX11.5 から追加されている命令があり、現時点では FP8/BF8 関連の命令をサポートすることが明らかにされている。  
RDNA 4m/GFX1170 は FP8/BF8 と FP32 の変換命令とドット積命令に対応する。  

 * [[AMDGPU] Add fp8/bf8 conversion instructions for gfx1170 by mbrkusanin · Pull Request #180191 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180191)
 * [[AMDGPU] Add dot4 fp8/bf8 instructions for gfx1170 by mbrkusanin · Pull Request #180516 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/180516)

現時点で RDNA 4m/GFX1170 について推測すると、RDNA 4/GFX12 から低精度データフォーマットの性能を減らし、APU 向けに必要な実装面積を減らしたバージョンではないかと思われる。  
RDNA 4/GFX12 と同じ実装ならば素直に GFX12xx にすればいいが、RDNA 4/GFX12 からどこか削減した部分があり、そして RDNA 3 系列から追加した部分があるから RDNA 4m/GFX1170 というバージョンになったのだろう。  

RDNA 4/GFX12 では専用の行列積和演算ユニットの導入により、RDNA 3/GFX11 と比較してクロックあたりの性能は FP16/BF16 だと 2倍、INT4/INT8 は 4倍となった。FP8/BF8 のサポートも追加され、FP8/BF8 の FP32 に対する性能レートは INT8 と同じ 8倍となっている。  
RDNA 3 系列では行列積和演算命令、WMMA (Wave Matrix Multiply Accumulate) 命令を内部的に複数のドット積命令に分解し、実行する方式を取っている。  
おそらく RDNA 4m/GFX1170 も同様の方式で FP8/BF8 の行列積和演算に対応すると思われる。  
これまでの RDNA 3 系列と同じ性能レートであれば、FP8/BF8 の性能は FP32 に対して 2倍、RDNA 4/GFX12 と比較して 1/4 になるだろう。  
RDNA 4/GFX12 の行列積和演算ユニットを丸ごと省略しているのであれば、他の低精度データフォーマットでの性能はこれまでの RDNA 3 系列と同じだろう。  
当然ながら、今後公開されるパッチでこの推測は変わるかもしれない。  

低精度データフォーマットでの性能が RDNA 4/GFX12 より低いとしても、ハードウェア的に FP8/BF8 に対応している意味は大きく、ソフトウェア側で変換処理を追加することなく FSR4 のサポートが可能になり、実行性能への悪影響も無くなる。  
