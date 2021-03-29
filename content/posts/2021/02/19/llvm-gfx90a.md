---
title: "LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か"
date: 2021-02-19T13:48:59+09:00
draft: false
tags: [ "MI200", "gfx90a", "CDNA_2", "CDNA", "Aldebaran" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMDGPU のコンパイラバックエンドとしても用いられる LLVM に、GPUID *gfx90a* のサポートを追加するパッチが投稿された。  
{{< link >}} [[AMDGPU] gfx90a support · llvm/llvm-project@a8d9d50](https://github.com/llvm/llvm-project/commit/a8d9d50762c42d726274d3f1126ec97ff96e2a22) {{< /link >}}
*Vega/GFX9* 世代であること、*Arcturus/MI100/gfx908* と同様に SRAM ECC、MFMA命令等をサポートしていることから、 *CDNA 2* (次世代 *CDNA* )  GPU、**MI200** の GPUID ではないかと思われる。  

{{< pindex >}}

 * [Packed FP32 / FullRate FP64](#pkfp32-fullfp64)
 * [TgSplit](#tgsplit)

{{< /pindex >}}

## Packed FP32 / FullRate FP64 {#pkfp32-fullfp64}

以下は *gfx90a* がサポートする ISA と機能の範囲だが、`FeaturePackedFP32Ops` と `FullRate64Ops` が存在する。  

 >        def FeatureISAVersion9_0_A : FeatureSet<
 >          [FeatureGFX9,
 >           FeatureGFX90AInsts,
 >           FeatureFmaMixInsts,
 >           FeatureLDSBankCount32,
 >           FeatureDLInsts,
 >           FeatureDot1Insts,
 >           FeatureDot2Insts,
 >           FeatureDot3Insts,
 >           FeatureDot4Insts,
 >           FeatureDot5Insts,
 >           FeatureDot6Insts,
 >           Feature64BitDPP,
 >           FeaturePackedFP32Ops,
 >           FeatureMAIInsts,
 >           FeaturePkFmacF16Inst,
 >           FeatureAtomicFaddInsts,
 >           FeatureMadMacF32Insts,
 >           FeatureSupportsSRAMECC,
 >           FeaturePackedTID,
 >           FullRate64Ops]>;
 >
 > {{< quote >}} [llvm-project/AMDGPU.td at 0db938312a06b8aa3b6c6c0272e7f28167bbd16a · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/0db938312a06b8aa3b6c6c0272e7f28167bbd16a/llvm/lib/Target/AMDGPU/AMDGPU.td) {{< /quote >}}

名前から推測するに、前者は FP32演算を 2つ束ねて (Packed) 実行することで 2倍のレートで処理する機能、後者はフルレートで FP64演算を行う機能と思われる。  
そう言ってしまうとあっさりしているが実際は凄まじい演算部の強化であり、これまでの AMD GPU における FP64演算は、FP32演算のレートの半分というのが通常だった。  
ピークFP64演算性能 11.5 TFLOPS を持ち、10 TFLOPS の壁を突破した初の GPU である *Arcturus/MI100* でも FP32演算の半分のレートであり、GPU内部のクラスタとなる ShaderEngine 8基、総 CU 120基という、それまでの AMD GPUアーキテクチャを踏襲しつつ、規模を増やすことでその性能を達成している。  
{{< link >}} [AMD、初の CDNAアーキテクチャ GPU 「MI100」 を発表 | Coelacanth's Dream](/posts/2020/11/17/amd-cdna-arch-mi100-arcturus/) {{< /link >}}
合わせて FP32演算のパックド実行もサポートしたことから、演算ユニット、ベクタレジスタは FP64 が基本単位となっていると思われ、 *Arcturus/MI100* から倍のリソースとなる。  

 >        unsigned GCNTTIImpl::getRegisterBitWidth(bool Vector) const {
 >          return (Vector && ST->hasPackedFP32Ops()) ? 64 : 32;
 >        }
 >
 > {{< quote >}} [llvm-project/AMDGPUTargetTransformInfo.cpp at a8d9d50762c42d726274d3f1126ec97ff96e2a22 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/a8d9d50762c42d726274d3f1126ec97ff96e2a22/llvm/lib/Target/AMDGPU/AMDGPUTargetTransformInfo.cpp) {{< /quote >}}

MFMA (Matrix Fused Multiply-Add) 命令も拡張され、強化された演算ユニットに向けた命令がいくつか追加されている。  

 >        def : SourceOfDivergence<int_amdgcn_mfma_f32_32x32x4bf16_1k>;
 >        def : SourceOfDivergence<int_amdgcn_mfma_f32_16x16x4bf16_1k>;
 >        def : SourceOfDivergence<int_amdgcn_mfma_f32_4x4x4bf16_1k>;
 >        def : SourceOfDivergence<int_amdgcn_mfma_f32_32x32x8bf16_1k>;
 >        def : SourceOfDivergence<int_amdgcn_mfma_f32_16x16x16bf16_1k>;
 >        def : SourceOfDivergence<int_amdgcn_mfma_f64_16x16x4f64>;
 >        def : SourceOfDivergence<int_amdgcn_mfma_f64_4x4x4f64>;
 >
 > {{< quote >}} [llvm-project/AMDGPUSearchableTables.td at a8d9d50762c42d726274d3f1126ec97ff96e2a22 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/a8d9d50762c42d726274d3f1126ec97ff96e2a22/llvm/lib/Target/AMDGPU/AMDGPUSearchableTables.td) {{< /quote >}}

**MI200** の CU数等は当然まだ不明だが、CUあたりの演算性能は向上しており、*Arcturus/MI100* と変わらないか多少少ない CU数だとしても、全体的なピーク演算性能は大きく向上すると考えられ、 **CDNA 2/MI200** はよりコンピュート性能に重きを置いた GPU となることが予想される。  

## TgSplit {#tgsplit}

*gfx90a* ではフロントエンド部の新たな機能として *TgSplit* モードがサポートされている。  

 * [llvm-project/AMDGPUUsage.rst at a8d9d50762c42d726274d3f1126ec97ff96e2a22 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/a8d9d50762c42d726274d3f1126ec97ff96e2a22/llvm/docs/AMDGPUUsage.rst#memory-model-gfx90a)

*TgSplit* は *Thread group Split* の略とされる。  
1つの Workgroup は通常同じ CU に割り振られ、CU内の異なる SIMDユニットで実行されるが、  
*TgSplit* では Workgroup を分割、異なる CU での実行を可能とする。  
(CU内の SIMDユニット数は、GCNアーキテクチャでは CUあたり 4 SIMDユニット、RDNA では 2 SIMDユニット。  
Workgroup は GPU が処理するカーネル内のブロックであり、スレッドの塊である Wavefront が最大 16個格納されている。Wavefront の単位は GCN では 64スレッド、RDNA では 32スレッド。RDNA は 64スレッドにも対応している。)  

*TgSplit* モードは少ない Workgroup 数であっても CU の稼働率を引き上げられる機能に見えるが、CU内の高速なローカルメモリ、LDS (Local Data Share) は 1つの Workgroup内のスレッド間で共有するため、Workgroup を分割し、異なる CU で実行する *TgSplit* モードでは LDS が割り当てられない場合があるとしている。  
そのため、*TgSplit* モードを活用し、性能を引き出すにはソフトウェア側の新たな最適化が必要と考えられるが、そこは ROCmソフトウェアが担当するのだろう。  

実行フローにおける新たな *TgSplit* モードの追加は、AMD GPUアーキテクチャとして大きな変更点だと言える。  
*gfx90a* では演算部が大きく強化されているが、それはユニット面積、GPUチップの製造コストにも響く。*gfx90a* の CU はこれまでの AMD GPU より大きくなるだろう。*Arcturus/gfx908* から増設された miSIMDユニット (Machine Intelligence SIMD) と AccVGPR (Accumulation VGPR) の存在もある。  
そういった点では、*TgSplit* モードは増強された CU の稼働率を上げ、ダイに掛かるコストパフォーマンスを高めるために必要だったのではないか、という見方もできる。  

また、 *XGMI/Infinity Fabric Link* によるメモリ共有が可能なマルチGPU構成時のメモリモデルも改良され、仮想アドレスが割り当てられている L2キャッシュを、必要以上に無効化することを減らす工夫が採り入れられている。  


| AMDGPU | GPU ID |
| :-- | :--: |
| Vega10 | gfx900 |
| Raven/Picasso | gfx902 |
| Vega12 | gfx904 |
| Vega20 | gfx906 |
| Arcturus | gfx908 |
| Raven2 | gfx909 |
| *MI200?* | *gfx90a?* |
| Renoir/Lucienne<br>/Green Sardine (Cezanne) | gfx90c |
| Navi10 | gfx1010 |
| Navi12 | gfx1011 |
| Navi14 | gfx1012 |
| Sienna Cichlid (Navi21) | gfx1030 |
| Navy Flounder (Navi22) | gfx1031 |
| Dimgrey Cavefish (Navi23) | gfx1032 |
| VanGogh | gfx1033? |
