---
title: "LLVM に GFX1031 へ向けたパッチが投稿される"
date: 2020-08-06T07:07:26+09:00
draft: false
tags: [ "RDNA_2", "Navy_Flounder" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
summary: "AMDGPU のコンパイラバックエンドとしても用いられる LLVM に、GPUID *gfx1031* のサポートに向けた初のパッチが投稿された。"
---

AMDGPU のコンパイラバックエンドとしても用いられる LLVM に、GPUID *gfx1031* のサポートに向けた初のパッチが投稿された。  
{{< link >}} [[AMDGPU] gfx1031 target · llvm/llvm-project@ea7d0e2](https://github.com/llvm/llvm-project/commit/ea7d0e2996ec6b72a08dbef26dadf217458ab382) {{< /link >}}

概要としては、オープンソースドライバーの **RadeonSI (OpenGL)** 、**RADV (Vulkan)** に [Navy Flounder](/tags/navy_flounder) のサポートが追加された時と同様に、  
大きな追加点、変更点は *gfx1030 /Sienna Cichlid* の時に追加されているため、基本 *gfx1031* の名を対応させたものとなっている。  
{{< link >}} [RadeonSI、RADV が AMD Navy Flounder をサポート | Coelacanth's Dream](/posts/2020/07/29/amd-navy_flounder-umd-patch/) {{< /link >}}

各 GPUID、AMDGPUマイクロアーキテクチャの詳細が記述された [llvm/AMDGPU.td](https://github.com/llvm-mirror/llvm/blob/master/lib/Target/AMDGPU/AMDGPU.td)にはコードの追加、変更が行なわれなかったため、*gfx1031* が持つバグといったような詳細はまだ不明であるが、  
[前回](/posts/2020/07/29/amd-navy_flounder-umd-patch/#gfx1030)疑問に覚えられた *Navy Flounder* が、*Sienna Cichlid* と同じ *gfx1030* に関連付けられていたことへの解答があった。  

まず、現時点で *gfx1030* と *gfx1031* との間に対応する命令範囲に違いはない。  

 >       case GK_GFX1031:
 >       case GK_GFX1030:
 >         Features["ci-insts"] = true;
 >         Features["dot1-insts"] = true;
 >         Features["dot2-insts"] = true;
 >         Features["dot5-insts"] = true;
 >         Features["dot6-insts"] = true;
 >         Features["dl-insts"] = true;
 >         Features["flat-address-space"] = true;
 >         Features["16-bit-insts"] = true;
 >         Features["dpp"] = true;
 >         Features["gfx8-insts"] = true;
 >         Features["gfx9-insts"] = true;
 >         Features["gfx10-insts"] = true;
 >         Features["gfx10-3-insts"] = true;
 >         Features["s-memrealtime"] = true;
 >         break;
 >
 > 引用元: <cite>[llvm-project/AMDGPU.cpp at ea7d0e2996ec6b72a08dbef26dadf217458ab382 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/ea7d0e2996ec6b72a08dbef26dadf217458ab382/clang/lib/Basic/Targets/AMDGPU.cpp#L177)</cite>

ただ、`gfx10-3-insts` に関してはまだ詳細が明かされて(記述されて)いない。  

次に、[llvm-project/GCNProcessors.td](https://github.com/llvm/llvm-project/blob/master/llvm/lib/Target/AMDGPU/GCNProcessors.td)においても、*gfx1031* は `FeatureISAVersion10_3_0` に関連付けられている。  

 >      def : ProcessorModel<"gfx1030", GFX10SpeedModel,
 >        FeatureISAVersion10_3_0.Features
 >      >;
 >      
 >      def : ProcessorModel<"gfx1031", GFX10SpeedModel,
 >        FeatureISAVersion10_3_0.Features
 >      >;
 >
 > 引用元: <cite>[llvm-project/GCNProcessors.td at ea7d0e2996ec6b72a08dbef26dadf217458ab382 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/ea7d0e2996ec6b72a08dbef26dadf217458ab382/llvm/lib/Target/AMDGPU/GCNProcessors.td)</cite>

つまり、[前回](/posts/2020/07/29/amd-navy_flounder-umd-patch/#gfx1030)書いたように、LLVM のバージョンが追い付いていないため、*Navy Flounder* を *gfx1030* に関連付けた、  
もしくは、AMD GPU のソフトウェア開発者が OSS である LLVM に *gfx1031* のパッチを投稿できる段階になく、LLVM にしても *gfx1031* を *gfx1030* に関連付けるしかない、と考えられる。  

何にしても、現段階では *gfx1031* は機能的に *gfx1030* と変わらないとされ、*Navy Flounder* は *gfx1030* に関連付けられている。  
そして、*Navy Flounder* は `chipRevision` の範囲から [Navi22](/tags/navi22) とされ、[Navi22](/tags/navi22)の GPUID は *gfx1031* であると考えられている。  

新 GPU のソフトウェアサポートの初期における、情報開示の制限によるボタンの掛け違いが起こっただけであり、自分としては以下の表であるように思う。  
ボタンを掛け違えるのは開発者側ではなく、自分たちの側である。  

| GFX10.3 ASIC Code Name | GPUID |
| :-- | :--: |
| Sienna Cichlid /Navi21 | gfx1030 |
| Navy Flounder /Navi22 | gfx1031 |
