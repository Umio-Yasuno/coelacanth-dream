---
title: "DPAS 命令をサポートしない Intel Xe-HPCVG (PVC-VG)"
date: 2023-12-19T14:22:45+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Xe-HPC", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---
先日 Intel がオープンソースで公開、開発している [VC Intrinsics](https://github.com/intel/vc-intrinsics) と [Intel(R) C for Metal Compiler](https://github.com/intel/cm-compiler/blob/cmc_monorepo_110/clang/Readme.md) に *Xe-HPCVG (PVC-VG)* のサポートが追加された。  
VC Intrinsics は Intel GPU 向けの LLVM IR 組み込み関数を使用するためのライブラリ、Intel(R) C for Metal Compiler (cm-compiler) は GPU カーネルのプログラミング言語、C for Metal の Intel GPU 向けコンパイラとなる。  

 * [Add support for XeHPCVG platform · intel/vc-intrinsics@da892e1](https://github.com/intel/vc-intrinsics/commit/da892e1982b6c25b9a133f85b4ac97142d8a3def)
 * [Support PVC-VG platform · intel/cm-compiler@c81407d](https://github.com/intel/cm-compiler/commit/c81407d41d4f2c40b4916867516eb755053dc438)

*Xe-HPCVG (PVC-VG)* は通常の *Xe-HPC* と異なり、行列演算用の `DPAS (Dot Product Accumulate Systolic)` 命令をサポートしないことがパッチから読み取れる。  

 >              "dpas" : { "result" : "anyvector",
 >                         "arguments" : [0,"anyvector","anyvector","int"],
 >         -               "attributes" : "NoMem"
 >         +               "attributes" : "NoMem",
 >         +               "platforms" : [ "XeHP+", "~XeLPG", "~XeHPCVG" ],
 >                       },
 >
 > {{< quote >}} <https://github.com/intel/vc-intrinsics/blob/da892e1982b6c25b9a133f85b4ac97142d8a3def/GenXIntrinsics/include/llvm/GenXIntrinsics/Intrinsic_definitions.py#L1641> {{< /quote >}}

上記の `~XeHPCVG` は、`DPAS` 命令をサポートするプラットフォームを示す `XeHP+` のグループから `XeHPCVG` を除外するという意味になっている。  

 >         #------------ Supported platforms ----------------------
 >         # Every intrinsic has optinal field "platforms" : "CPU"
 >         # CPU can be any from "platforms" in Intrinsics.py or "ALL"
 >         # when field is absent - ALL by default
 >         # additional commands :
 >         # "CPU" = "-Gen9" - unsupported since Gen9
 >         # "CPU" = "Gen11+" - supported from Gen11
 >         # "CPU" = "~XeLP" - unsupported on XeLP
 >         # CPU can be list:
 >         # ["XeLP+", "Gen9"] - supported on Gen9 and all started from XeLP
 >         # ["ALL", "~XeLP"] - supported everyvere except XeLP
 >
 > {{< quote >}} <https://github.com/intel/vc-intrinsics/blob/da892e1982b6c25b9a133f85b4ac97142d8a3def/GenXIntrinsics/include/llvm/GenXIntrinsics/Intrinsic_definitions.py#L1641> {{< /quote >}}

Intel(R) C for Metal Compiler へのパッチでも同様のことを読み取ることができる。  
また *Xe-HPCVG (PVC-VG)* の PCI DeviceID は `0x0BD4` とされており、この ID の Intel GPU 製品はまだ発表されていない。  

 >         +  { encodeGmdId(12, 61, 7),
 >         +    { /*.HasFP64 =*/ true,
 >         +      /*.HasBFloat16 =*/ true,
 >         +      /*.HasSLMCasInt64 =*/ true,
 >         +      /*.HasDpas =*/ false,
 >         +      /*.HasDpasw =*/ false,
 >         +      /*.HasDpasFp16 =*/ false,
 >         +      /*.HasDpasBf16 =*/ false,
 >         +      /*.HasDpasTf32 =*/ false,
 >         +      /*.GrfWidth =*/ 512,
 >         +      /*.SupportedGrfNums =*/ {128,256},
 >         +      /*.MaxSLMSize =*/ 128, }, },
 >
 > {{< quote >}} [Support PVC-VG platform · intel/cm-compiler@c81407d](https://github.com/intel/cm-compiler/commit/c81407d41d4f2c40b4916867516eb755053dc438) {{< /quote >}}
 >
 >          static const std::unordered_map<std::string, uint32_t> ReleaseId = {
 >            {"xe-lpg-lg", encodeGmdId(12, 71, 4)},
 >            {"xe-lpg-md", encodeGmdId(12, 70, 4)},
 >         +  {"xe-hpc-vg", encodeGmdId(12, 61, 7)},
 >            {"xe-hpc", encodeGmdId(12, 60, 7)},
 >            {"xe-hpg", encodeGmdId(12, 57, 0)},
 >            {"xe-hp", encodeGmdId(12, 50, 4)},
 >
 > {{< quote >}} [Support PVC-VG platform · intel/cm-compiler@c81407d](https://github.com/intel/cm-compiler/commit/c81407d41d4f2c40b4916867516eb755053dc438) {{< /quote >}}
 >
 >            {"0x7d40", encodeGmdId(12, 70, 4)},
 >            {"0x7d45", encodeGmdId(12, 70, 4)},
 >            {"0x7d60", encodeGmdId(12, 70, 4)},
 >         +  {"0x0bd4", encodeGmdId(12, 61, 7)},
 >
 > {{< quote >}} [Support PVC-VG platform · intel/cm-compiler@c81407d](https://github.com/intel/cm-compiler/commit/c81407d41d4f2c40b4916867516eb755053dc438) {{< /quote >}}

現状 Intel から公開されているのは *Xe-HPCVG (PVC-VG)* が `DPAS` 命令をサポートしないということだけであり、既存の *Xe-HPC (Ponte Vechhio)* から `DPAS` 命令を無効化したプラットフォームなのか、  
それとも XMX (Xe Matrix eXtension) ユニットを持たないダイを再設計し、それを使用したプラットフォームなのかは不明。  
`DPAS` 命令をサポートせずとも、*Xe-HPCVG (PVC-VG)* には FP64 演算性能やメモリ帯域といった特徴が *Xe-HPC* から残るとは思うが、それが製品として既存の Intel Deta Center GPU Max Series や AMD と NVIDIA のデータセンター向け GPU と比較してどれだけ有効かは疑問である。  

