---
title: "LLVM に Navi21/GFX1030 へ向けたパッチが投稿される"
date: 2020-06-16T22:55:37+09:00
draft: false
tags: [ "Navi21", "Sienna_Cichlid", "RDNA_2", "GFX10", "gfx1030" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMDGPU のコンパイラバックエンドとしても用いられる LLVM に、*Navi21/gfx1030* のサポートに向けたパッチが投稿された。  
{{< link >}}[[AMDGPU] Add gfx1030 target · llvm/llvm-project@9ee272f](https://github.com/llvm/llvm-project/commit/9ee272f13d88f090817235ef4f91e56bb2a153d6){{< /link >}}

今回追加されたコードは LLVM 11 に取り込まれるとされ[^1]、その LLVM 11 はリリースサイクルから 2020/09 頃に来るはずだ。  

[^1]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389/diffs?commit_id=7366021e3854532837dc5c569bc8a24bb023c11b>

*Navi21/gfx1030* が対応する機能には、*Navi12/gfx1011* 、*Navi14/gfx1012* の範囲に加えて、`FeatureGFX10_3Insts`、`FeatureGFX10_BEncoding` が追加される。  
後者は新たな命令エンコードのフォーマットとされるが、前者の概要はまだ明らかになっていない。[^2]  
また、*Navi1x* であったバグ等が修正されている。  
*Arcturus* で追加された一部ドット積の混合演算には対応しなかった。 (`FeatureDot3Insts`、`FeatureDot4Insts`)  
そして、`HalfRate64Ops`、`FeatureSRAMECC` にも対応していないことから、大方察しが付いているように *Navi21* は *Navi1x* 同様にゲーミング向けのGPUだろう。{{< comple >}}今後それらに対応した RDNAアーキテクチャ GPUが出てくることはあるのだろうか？ 完全に GCNアーキテクチャが担当することに？{{< /comple >}}  

[^2]: <https://github.com/llvm/llvm-project/blob/9ee272f13d88f090817235ef4f91e56bb2a153d6/llvm/lib/Target/AMDGPU/AMDGPU.td>

他には、`image_msaa_load` 命令が追加されている。[^3]  
グラフィクス系の命令と考えられ、うまく読め解けないため推測となるが、MSAA処理を施したデータをどのレベルでコヒーレントを取るか指定できるようになっている？ (SLC = System Level Coherent, GLC = Globally Coherent)  

[^3]: <https://github.com/llvm/llvm-project/blob/9ee272f13d88f090817235ef4f91e56bb2a153d6/llvm/test/CodeGen/AMDGPU/llvm.amdgcn.image.msaa.load.ll>

以前、OpenGL/Vulkan ドライバーに追加されたコードと同様に、SIMD32ユニットあたりの Wave数が 16エントリとなることを示すコードも追加されている。[^4]  
{{< link >}}[AMD Sienna Cichlid をサポートするパッチが OpenGL、Vulkanドライバーに投稿される | Coelacanth's Dream](/posts/2020/06/09/amd-sienna-cichlid-oss-umd/){{< /link >}}
その時に、*AMD Polaris系* も SIMD16ユニットあたり 8エントリとなっていると書いたが、LLVM のコードにはそういった記述は見つけられなかった。  
グラフィクス系処理のみに向けた最適化の一環だったりするのだろうか？  

[^4]: <https://github.com/llvm/llvm-project/blob/9ee272f13d88f090817235ef4f91e56bb2a153d6/llvm/lib/Target/AMDGPU/Utils/AMDGPUBaseInfo.cpp>

{{< ref >}}

 * <https://developer.amd.com/wp-content/resources/RDNA_Shader_ISA.pdf>

{{< /ref >}}
