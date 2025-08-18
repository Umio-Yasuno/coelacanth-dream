---
title: "Wave32 モードのみをサポートし、LDS を 320KiB 持つ GFX1250"
date: 2025-08-15T18:50:22+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "GFX12.5", "LLVM", "CDNA" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

LLVM にてサポート作業が進められている次世代の CDNA 系 APU *gfx1250* だが、最近の変更にて Wave64 モードをサポートせず、Wave32 モードのみをサポートすることが公開された。  

## Wave32 モードのみサポート {#only-wave32}
Wave とは AMD GPU におけるスレッド (Work-item) の塊であり、SIMD ユニットへの命令の発行単位とも言える。  
GCN アーキテクチャと CDNA 3 までの CDNA 系アーキテクチャでは Wave64 (64 work-item) のみをサポートしており、RDNA 系アーキテクチャから Wave32 (32 work-item) のサポートが追加された。  
Wave64 モードの特徴としては、発行単位が大きく複数回に分けて SIMD ユニットで実行されるため、Wave が終了するまでのレイテンシは大きくなるが、SIMD ユニットの稼働率は高くしやすいことが挙げられる。  

SIMD ユニットの構成によって命令の実行方式は異なるが、Wave64 モードは長年 AMD GPU でサポートされてきた。  
それが *gfx1250* ではついに Wave64 モードが削除され、Wave32 モードのみがサポートされる。  

 * [[AMDGPU] Remove wave64 functions by rampitec · Pull Request #153690 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/153690)
 * [[AMDGPU] Error out in clang if wavefront64 is used on gfx1250 by rampitec · Pull Request #153693 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/153693)

AMD GPU としては象徴的な変更と言えるかもしれないが、RDNA 3 では Wave32 モードでのみサポートされる Dual Issue Wave32 (VOPD 命令) が、RDNA 4 では同様の Dynamic register allocation (Dynamic VGPR) が追加されてきたため、そうした流れで言えば自然なものなのかもしれない。  

## LDS 320KiB {#lds}
AMD GPU が CU (Compute Unit) ごとに持つ高速なローカルメモリ、LDS (Local Data Share) が *gfx1250* では 320KiB へと拡張される。  
CDNA 4 アーキテクチャでは従来の 64KiB から 160KiB と、倍以上のサイズになったが、*gfx1250* ではそこからさらに倍となる。  
CDNA 4 で LDS のサイズと帯域を拡張させた理由について、データの再利用が特に多い行列演算ルーチンにおいて、メモリアクセスを減らし CU の稼働率を上げるのに不可欠と語られているため、*gfx1250* でのさらなる拡張も同様の理由だろう。[^cdna4]  

 * [[AMDGPU] Increase LDS to 320K on gfx1250 by rampitec · Pull Request #153645 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/153645)

[^cdna4]: [amd-cdna-4-architecture-whitepaper.pdf](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-4-architecture-whitepaper.pdf)

## VOPD 命令の制限が緩和 {#vopd}
RDNA 3 からサポートされている Dual issue (VOPD 命令) は 2種類の VOPD 命令を同時に実行する機能だが、正常に機能させるにはいくつかの要件を守る必要がある。  
その一つが 2種類の命令 (opX, opY) のソースオペランドが異なる VGPR バンクであること、というものだ。  
それが *gfx1250* では緩和され、同じ VGPR バンクであっても正常に VOPD 命令が実行可能となる。  
具体的な例は特に思いつかないが、VOPD 命令にコンパイル可能なパターンが増えることで実行性能も向上するケースがあるかもしれない。  

 >         +  // If \p AllowSameVGPR is set then same VGPRs are allowed for X and Y sources
 >         +  // even though it violates requirement to be from different banks.
 >         +  // If \p VOPD3 is set to true both dst registers allowed to be either odd
 >         +  // or even and instruction may have real src2 as opposed to tied accumulator.
 >            bool hasInvalidOperand(std::function<unsigned(unsigned, unsigned)> GetRegIdx,
 >         -                         bool SkipSrc = false) const {
 >         -    return getInvalidCompOperandIndex(GetRegIdx, SkipSrc).has_value();
 >         +                         const MCRegisterInfo &MRI, bool SkipSrc = false,
 >         +                         bool AllowSameVGPR = false, bool VOPD3 = false) const {
 >         +    return getInvalidCompOperandIndex(GetRegIdx, MRI, SkipSrc, AllowSameVGPR,
 >         +                                      VOPD3)
 >         +        .has_value();
 >
 > {{< quote >}} [[AMDGPU] VOPD/VOPD3 changes for gfx1250 by rampitec · Pull Request #147602 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/147602) {{< /quote >}}

## WGP モードもサポートせず {#no-wgp-mode}
*gfx1250* は WGP モードもサポートせず、CU モードのみをサポートするとされている。  

 * [[AMDGPU] Don't allow wgp mode on gfx1250 by rampitec · Pull Request #153680 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/153680)

WGP モード、CU モードは RDNA 1 からサポートされており、Work-group の発行方式に関わる機能である。  
Work-group は Wave の集まりであり、同じ Work-group 内の Wave はバリア命令を介して同期でき、LDS も共有できる。  
CU モードは 1基の CU を Work-group の発行単位とし、SIMD ユニット 2基で実行され、LDS 64KiB が使用可能となる。 (RDNA 1/2/3/4 の場合)  
WGP モードでは 2基の CU をまとめた WGP (Work-group processor) を Work-group の発行単位とし、SIMD ユニット 4基、LDS 128KiB が使用可能となる。  

そして *gfx1250* では CU あたり 4基の SIMD ユニットを持つとされている。  
CU モードのみをサポートすることと SIMD ユニットの数は、GCN アーキテクチャの構成に近いと言える。  
見方を変えれば、LDS のサイズが拡張されたことと合わせ、RDNA 系の CU 2基を 1基に融合させた結果とも見えるだろう。  
しかし、SIMD ユニットの幅は不明である。  

この場合、VOPD 命令はどのように実行されるかだが、  
RDNA 3/4 では、SIMD32 (Float, Int) ユニット 2基に加え、それぞれにサブセットとなる SIMD32 (Float) ユニットがあり、VOPD 命令の実行にはサブセットを活用する方式だった。  
Wave が SIMD ユニットごとに割り当てられることと、現時点で VOPD 命令が拡張されていないことを考えると、*gfx1250* でも RDNA 3/4 と同様にサブセットとなる SIMD32 ユニットを持ち、それを活用する可能性が高いように思う。  

 >          unsigned getEUsPerCU(const MCSubtargetInfo *STI) {
 >            // "Per CU" really means "per whatever functional block the waves of a
 >         -  // workgroup must share". For gfx10 in CU mode this is the CU, which contains
 >         +  // workgroup must share".
 >         +
 >         +  // GFX12.5 only supports CU mode, which contains four SIMDs.
 >         +  if (isGFX1250(*STI)) {
 >         +    assert(STI->getFeatureBits().test(FeatureCuMode));
 >         +    return 4;
 >         +  }
 >         +
 >         +  // For gfx10 in CU mode the functional block is the CU, which contains
 >            // two SIMDs.
 >            if (isGFX10Plus(*STI) && STI->getFeatureBits().test(FeatureCuMode))
 >              return 2;
 >         -  // Pre-gfx10 a CU contains four SIMDs. For gfx10 in WGP mode the WGP contains
 >         -  // two CUs, so a total of four SIMDs.
 >         +
 >         +  // Pre-gfx10 a CU contains four SIMDs. For gfx10 in WGP mode the WGP
 >         +  // contains two CUs, so a total of four SIMDs.
 >            return 4;
 >          }
 >
 > {{< quote >}} [[AMDGPU] Don't allow wgp mode on gfx1250 by rampitec · Pull Request #153680 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/153680) {{< /quote >}}
