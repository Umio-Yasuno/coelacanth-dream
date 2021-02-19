---
title: "LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か"
date: 2021-02-19T13:48:59+09:00
draft: false
tags: [ "MI200", "gfx90a", "CDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMDGPU のコンパイラバックエンドとしても用いられる LLVM に、GPUID *gfx90a* のサポートを追加するパッチが投稿された。  
{{< link >}} [[AMDGPU] gfx90a support · llvm/llvm-project@a8d9d50](https://github.com/llvm/llvm-project/commit/a8d9d50762c42d726274d3f1126ec97ff96e2a22) {{< /link >}}
*Vega/GFX9* 世代であること、*Arcturus/MI100/gfx908* と同様に SRAM ECC、MFMA命令等をサポートしていることから、次世代 *CDNA* GPU、**MI200** の GPUID ではないかと思われる。  

## Packed FP32 / FullRate FP64

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

名前から推測するに、前者は FP32演算を 2つ束ねて (Packed) 実行することで 2倍のレートでする機能、後者はフルレートで FP64演算を行う機能と思われる。  
そう言ってしまうとあっさりしているが実際は凄まじい演算部の強化であり、これまでの AMD GPU における FP64演算は、FP32演算のレートの半分というのが通常であり、ピークFP64演算性能 11.5 TFLOPS を持ち、10 TFLOPS の壁を突破した初の GPU である *Arcturus/MI100* でも FP32演算の半分のレートだった。  {{< link >}} [AMD、初の CDNAアーキテクチャ GPU 「MI100」 を発表 | Coelacanth's Dream](/posts/2020/11/17/amd-cdna-arch-mi100-arcturus/) {{< /link >}}
合わせて FP32演算のパックド実行もサポートしたことから、演算ユニットは FP64 が基本単位となっていると思われる。  
**MI200** の CU数等は当然まだ不明だが、*Arcturus/MI100* と変わらないか多少少ない CU数だとしても、FP64演算性能は大きく向上するとされ、よりコンピュート性能に重きを置いた GPU となることが予想される。  

 >        unsigned GCNTTIImpl::getRegisterBitWidth(bool Vector) const {
 >          return (Vector && ST->hasPackedFP32Ops()) ? 64 : 32;
 >        }
 >
 > {{< quote >}} [llvm-project/AMDGPUTargetTransformInfo.cpp at a8d9d50762c42d726274d3f1126ec97ff96e2a22 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/a8d9d50762c42d726274d3f1126ec97ff96e2a22/llvm/lib/Target/AMDGPU/AMDGPUTargetTransformInfo.cpp) {{< /quote >}}

## TGsplit

*gfx90a* ではフロントエンド部の新たな機能として *TGsplit* モードがサポートされている。  

 * [llvm-project/AMDGPUUsage.rst at a8d9d50762c42d726274d3f1126ec97ff96e2a22 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/a8d9d50762c42d726274d3f1126ec97ff96e2a22/llvm/docs/AMDGPUUsage.rst#memory-model-gfx90a)

*TGsplit* は *Thread Group Split* の略であり、1つの Workgroup は通常同じ CU に割り振られ、CU内の異なる SIMDユニットで実行されるが (GCNアーキテクチャでは CUあたり 4 SIMDユニット、RDNA では 2 SIMDユニット)、  
*TGsplit* では Workgroup を分割し、異なる CU で実行することが可能となる。(Workgroup は GPU が処理するカーネル内のブロックであり、スレッドの塊である Wavefront が最大 16個格納されている。Wavefront の単位は GCN では 64スレッド、RDNA では 32スレッド。RDNA は 64スレッドにも対応している)  
少ない Workgroup 数であっても CU の稼働率を引き上げられる機能に見えるが、CU内の高速なローカルメモリ、LDS (Local Data Share) は 1つの Workgroup内のスレッド間で共有するため、Workgroup を分割し、異なる CU で実行する *TGsplit* モードでは LDS が割り当てられない場合があるとしている。  

しかし、AMD GPUアーキテクチャとして大きな変更点であることは確かだ。  


