---
title: "Intel Graphics Compiler で Meteor Lake のサポートが進み始める ―― Xe-HPG、XMXユニットは非搭載か、再度 FP64 に対応"
date: 2022-07-06T15:50:54+09:00
draft: false
categories: [ "Intel", "GPU", "Hardware" ]
tags: [ "Meteor_Lake", "Xe-HPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel の Pete Chou 氏より、[Intel Graphics Compiler (IGC)](https://github.com/intel/intel-graphics-compiler) のサポートターゲット、vISA (virtual ISA) に *Meteor Lake* を追加するコミットが追加されている。  

 * [vISA: Add MTL target. · intel/intel-graphics-compiler@20ef9c5](https://github.com/intel/intel-graphics-compiler/commit/20ef9c5a42f4154c9e18c65cafd55e8af5f67c17)

## Xe-HPG {#xe-hpg}
*Meteor Lake GPU* は IGC の内部的なアーキテクチャ、プラットフォームとしては *{{< xe class="hpg" >}}* となる。  
他の部分を読むに現行の *DG2/Alchemist ({{< xe class="hpg" >}})* をそのまま iGPU 化したのではなく、そこからいくつかの変更が施されている。  

 > 		     case Xe_DG2:
 > 		+    case Xe_MTL:
 > 		         platform = Platform::XE_HPG;
 >
 > {{< quote >}} [vISA: Add MTL target. · intel/intel-graphics-compiler@20ef9c5](https://github.com/intel/intel-graphics-compiler/commit/20ef9c5a42f4154c9e18c65cafd55e8af5f67c17) {{< /quote >}}

### DPAS 命令には非対応、XMXユニット非搭載か {#nodpas}
EU あたり 8スレッド、EU あたり 4スレッドにも設定可能であり、その場合スレッドあたりの利用可能レジスタファイルが 256エントリ (8KiB) になるという点は *Meteor Lake* と *{{< xe class="hp/hpg" >}}* と共通する。  
EU のネイティブでの実行サイズが SIMD8 という点も変わらない。  
従来から大幅に設計が見直されたデータポート、LSC (Load Store Cache) も *{{< xe class="hpg/hpc" >}}}* と同様に採用している。  
{{< link >}} [Intel Xe-HP EU に追加されるパイプラインと増加するスレッド/レジスタファイル | Coelacanth's Dream](/posts/2021/06/08/intel-xe_hp-thread-reg-pipe/) {{< /link >}}

だが行列積和演算を行う `DPAS (Dot Product Accumulate Systolic)` 命令、XMX ({{< xe >}} Matrix eXtension) とも呼ばれるユニットの有無を判定する `hasDPAS()` において `getPlatform() != Xe_MTL` が追加されている。  
このことから *Meteor Lake GPU* では XMXユニットが搭載されていないことが考えられる。  
XMXユニットは主に推論処理の高速化を目的としており、用途の一つに Intel はアップスケーリング技術 {{< xe >}}SS (Super Sampling) を挙げている。  
{{< xe >}}SS は DP4a (INT8) にも対応しているため、タイルアーキテクチャを採用するとはいえ iGPU としてダイ面積、面積あたりの性能を *Meteor Lake GPU* で重視した結果、XMXユニットを搭載しなかったのではないかと考えられる。  
*Meteor Lake* は VPU (Vision Processing Unit) を搭載するとされているため、GPU との連携が重要な推論処理以外は VPU で実行することができる。  
{{< link >}} [Intel Movidius VPU に関するメモ ―― Keem Bay, Thunder Bay, Meteor Lake | Coelacanth's Dream](/posts/2022/01/11/intel-kmb-thb/) {{< /link >}}

 > 		     bool hasDPAS() const
 > 		     {
 > 		-        return getPlatform() >= Xe_XeHPSDV;
 > 		+        return getPlatform() >= Xe_XeHPSDV && getPlatform() != Xe_MTL;
 > 		     }
 >
 > {{< quote >}} [vISA: Add MTL target. · intel/intel-graphics-compiler@20ef9c5](https://github.com/intel/intel-graphics-compiler/commit/20ef9c5a42f4154c9e18c65cafd55e8af5f67c17) {{< /quote >}}

### FP64 に部分的に対応 {#fp64}
Intel GPU では *Gen11 アーキテクチャ* から Int64/FP64 の対応がハードウェア的には外され、ソフトウェアエミュレータでの対応となり、これは *DG2/Alchemist* にも引き継がれているが、*Meteor Lake GPU* では FP64 に部分的ではあるが再度ハードウェアが対応する様子を見せている。  
[visa/HWCaps.inc](https://github.com/intel/intel-graphics-compiler/blob/20ef9c5a42f4154c9e18c65cafd55e8af5f67c17/visa/HWCaps.inc) において、`noInt64()` の判定部に `Xe_MTL` が追加されているが、`noFP64()` には追加されていない。FP64 にのみ対応し、Int64 には対応しないものと思われる。  

 > 		    bool noInt64() const
 > 		    {
 > 		        return getPlatform() == GENX_ICLLP || isXeLP() || getPlatform() == Xe_DG2 ||
 > 		               getPlatform() == Xe_MTL;
 > 		    }
 > 		
 > 		    bool noFP64() const
 > 		    {
 > 		        return getPlatform() == GENX_ICLLP || isXeLP() || getPlatform() == Xe_DG2;
 > 		    }
 >
 > {{< quote >}} [intel-graphics-compiler/HWCaps.inc at 20ef9c5a42f4154c9e18c65cafd55e8af5f67c17 · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/20ef9c5a42f4154c9e18c65cafd55e8af5f67c17/visa/HWCaps.inc#L514-L523) {{< /quote >}}

実行パイプは複雑な関数を実行する EM (Extend Math) パイプとされ、{{< xe class="hp/hpc" >}} のように専用の Longパイプは持たない。  
また、*Gen9 アーキテクチャ* のドキュメントによれば、*Gen9* においても FP64 演算は EMパイプで実行されていた。[^gen9]そのため、FP64 演算に関しては *Meteor Lake GPU* で従来の方式に戻したとも見られる。  
グラフィクス性能の最適化から *Gen11 アーキテクチャ* でハードウェアから対応を外したが、互換性の問題などからハードウェアの対応はやはり必要と判断したのだろうか。  
同様の変更が dGPU においても *DG2/Alchemist* の次世代で適用される可能性が考えられる。  

FP64 演算性能について、*Gen9* では使用可能な FPパイプ (SIMD4) が片方 1基であり、スループットが半分ということから、FP32 演算性能に対して 1/4 のピーク性能となっていた。  
*{{< xe class="lp" >}}* では FP/INTパイプ が SIMD8 なのに対し、EMパイプは SIMD2 となっており、この構成を引き継いでいるとすると、*Meteor Lake GPU* では FP32 演算性能に対して 1/8 の FP64演算性能になるのではないかと思われる。  
なおスループットに関しては 1/2 以外である可能性もある。  

 > 		         ThreeDistPipe         = 2, // XeHP/XeHPG: 3 distance pipe
 > 		         FourDistPipe          = 3, // XeHPC (early variant): 4 distance pipes
 > 		         FourDistPipeReduction = 6, // XeHPC variation: 4 distance pipes with Long pipe reduction
 > 		+        ThreeDistPipeDPMath   = 7, // MTL: 3 distance pipe with DP operations in Math pipe
 >
 > {{< quote >}} [vISA: Add MTL target. · intel/intel-graphics-compiler@20ef9c5](https://github.com/intel/intel-graphics-compiler/commit/20ef9c5a42f4154c9e18c65cafd55e8af5f67c17) {{< /quote >}}

[^gen9]: [the-compute-architecture-of-intel-processor-graphics-gen9-v1d0.pdf](https://www.intel.com/content/dam/develop/external/us/en/documents/the-compute-architecture-of-intel-processor-graphics-gen9-v1d0.pdf)

GPU の FP32:FP64 演算性能レートは、AMD GPU では FP64演算性能を高めたサーバー向けを除けば *RDNA 2/GFX10.3 アーキテクチャ* までの長い間 16:1 を保っていたが、*RDNA 3/GFX11* では 32:1 になることが明らかにされている。  
{{< link >}} [GFX11 では FP64 演算性能が FP32 の 1/32 に | Coelacanth's Dream](/posts/2022/06/18/gfx11-dpfp-rate/) {{< /link >}}

{{< ref >}}
 * [Intel® Iris® Xe GPU Architecture](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-gpu-optimization-guide/top/xe-arch.html)
 * [インテル® Arc™ - Xe スーパーサンプリング](https://www.intel.co.jp/content/www/jp/ja/products/docs/arc-discrete-graphics/xess.html)
 * [Intel® Processor Graphics Xᵉ-LP API Developer and Optimization Guide](https://www.intel.com/content/www/us/en/developer/articles/guide/lp-api-developer-optimization-guide.html)
 * [Re: What processors have an internal Gen11 GPU *with* FP64 (double precision) enabled? - Intel Communities](https://community.intel.com/t5/Graphics/What-processors-have-an-internal-Gen11-GPU-with-FP64-double/m-p/691217)
 * [the-compute-architecture-of-intel-processor-graphics-gen9-v1d0.pdf](https://www.intel.com/content/dam/develop/external/us/en/documents/the-compute-architecture-of-intel-processor-graphics-gen9-v1d0.pdf)
{{< /ref >}}
