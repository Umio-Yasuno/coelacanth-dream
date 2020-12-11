---
title: "AMD、RDNA 2 ISA Reference Guide を公開"
date: 2020-12-08T17:17:33+09:00
draft: false
tags: [ "RDNA_2", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

RDNA 2 Instruction Set Architecture Reference Guide がいつの間にか公開されていた。  

 * [Developer Guides, Manuals & ISA Documents - AMD](https://developer.amd.com/resources/developer-guides-manuals/)
   * ["RDNA 2" Instruction Set Architecture: Reference Guide - RDNA2_Shader_ISA_November2020.pdf](https://developer.amd.com/wp-content/resources/RDNA2_Shader_ISA_November2020.pdf)

ドキュメント内の日付は 2020/11/30 となっているが、[GPUOpen](https://gpuopen.com) のページはこれを書いている現時点で反映されておらず、RDNA 2 ISA Reference Guide へのリンクは無い。[^gpuopen-isa-doc]  

[^gpuopen-isa-doc]: [AMD ISA Documentation - GPUOpen](https://gpuopen.com/documentation/amd-isa-documentation/)

{{< ins datetime="2020/12/10" >}}

GPUOpen の方にも反映された。  

 * [AMD RDNA™ 2 Instruction Set Architecture reference guide is now available - GPUOpen](https://gpuopen.com/rdna2-isa-available/)  
 * [Documentation - GPUOpen](https://gpuopen.com/documentation/)

{{< /ins >}}

RDNA 2 ISA Reference Guide の公開により、オープンソースドライバーの開発、特に [ACOバックエンド](/tags/aco) の *RDNA 2 アーキテクチャ* 最適化が進むと思われる。  
HWレイトレーシング実行のための命令とその仕様も公開された。Windowsドライバーでは既に Vulkan のレイトレーシング拡張に対応しており、オープンソースドライバーもそれに続くことが期待される。[^vk-ray]  

[^vk-ray]: [Vulkan® ray tracing extension support in our latest AMD Radeon™ Adrenalin driver 20.11.3 - GPUOpen](https://gpuopen.com/vulkan-ray-tracing-extensions/)

## RDNA 2 アーキテクチャの変更点 {#rdna_2-change}

### レイトレーシング {#rt}

*RDNA 2 アーキテクチャ* におけるハードウェアレイトレーシングは以下の命令によって実行される。  

 > * IMAGE_BVH_INTERSECT_RAY
 > * IMAGE_BVH64_INTERSECT_RAY

これらの命令は、レイデータを WGP(CU) 内に配置されている (近い) ベクタレジスタから受け取り、BVH (Bounding Volume Hierarchy) は WGP(CU) からは遠いメモリから持ってきて処理する。  
BVH データをメモリから持ってくるからこそメモリキャッシュ、特に *RDNA 2 / GFX10.3* 世代で追加された新たなメモリキャッシュ階層 **Infinity Cache** が効果を発揮するのだろう。  

レイ交差の処理性能は、ボックスに対しては 4個の交差を検出でき、トライアングルに対しては 1個の交差が検出できるものとなっている。  

### 推論、学習向けのドット積命令に対応 {#dot}

主に、推論、機械学習の高層化を目的とした、以下のドット積の混合精度演算命令に新しく対応した。  

 > * V_DOT2_F32_F16 / V_DOT2C_F32_F16
 > * V_DOT2_I32_I16 / V_DOT2_U32_U16
 > * V_DOT4_I32_I8 / V_DOT4C_I32_I8
 > * V_DOT4_U32_U8
 > * V_DOT8_I32_I4
 > * V_DOT8_U32_U4

ただ新しくと言っても、これらの命令は [Vega 7nm / Veag20](/tags/vega20) から対応しており、[Navi12](/tags/navi12)、[Navi14](/tags/navi14) も対応していた。  
その中で [Navi10](/tags/navi10) だけが対応していなかったため、[Navi10](/tags/navi10) からの変更点と言えば嘘ではない。[^dot-insts]  

[^dot-insts]: [Navi12情報近況 (2020-03-09) & 推測 | Coelacanth's Dream](/posts/2020/03/09/navi12-recent-info/)

### ベクタレジスタと LDS の割り当てサイズを倍増 {#vgpr-lds-alloc}

*RDNA 2 アーキテクチャ* の SIMDユニットごとが持つベクタレジスタ、WGP (2CU) 内のユニットで共有する LDS (Local Data Share) の容量、バンク数に変更は無いが、  
ベクタレジスタと LDS を割り当てるユニットサイズが倍増された。  
ベクタレジスタは、Wave64 (Wave32x2) での実行時は 8 Dwords、Wave32 時は 16 DWords のグループに割り当てられる。(DWord: Double Word, 32-bit, 4-Byte)  
LDS は 256 DWords のブロックが Work-group、または Wave ごとに割り当てられる。  
Wave はスレッドの塊の意だが、Work-group は Wave の塊で、Wave間の同期、LDS を用いたデータ共有が可能。  

ただこの部分に関しては正直自信がなく、ベクタレジスタと LDS の割り当てに言及してる資料が他に見つからなかった。  
そのため、ベクタレジスタを 8個か 16個のグループに分けて割り当てるのか、8 Dowrds あるいは 16 DWords ごとのグループに、Wave のサイズによって割り当てるのか {{< comple >}} 自分の中で {{< /comple >}} 曖昧である。  
*RDNA アーキテクチャ* では、Wave64 (Wave32x2) 時に 4 DWords、Wave32 時に 8 DWords のグループに割り当てていたため、前者の解釈だと割り当てられるベクタレジスタの粒度は小さく、後者は大きくなる。  

RadeonSIドライバーに、*RDNA アーキテクチャ* でも Wave64 (Wave32x2) で各シェーダーを実行する変更が為された時のコメントから、後者の粒度が大きくなったという解釈のが正しいようには思える。自信はない。  
{{< link >}} [RadeonSI ドライバーでは RDNA GPU も Wave64モードで各シェーダーを実行するように | Coelacanth's Dream](/posts/2020/07/02/radeonsi-shader-wave64-with-rdna/) {{< /link >}}

<br>
他には、1つのコンポーネントに対して最大 4つのサンプルを、サンプラーを用いずに読み込む `IMAGE_MSAA_LOAD`、  
グローバルメモリへの TID (Thread-ID) によるアドレッシング等が新たにサポートされている。  

以前にそのような話はあったが、*RDNA 2 アーキテクチャ* では MAD を使わず FMA のみを使うようになり、以下の命令は置き換えられた。  
{{< link >}} [AMD GPU の世代における FMA、MAD 命令の微妙な仕様と違い | Coelacanth's Dream](/posts/2020/09/16/amd-gcn-rdna-fma-mad/) {{< /link >}}

 >  * V_MAC_LEGACY_F32 (replaced by V_FMAC_LEGACY_F32)
 >  * V_MAD_LEGACY_F32 (replaced by V_FMA_LEGACY_F32)
 >  * V_MAC_F32, V_MADMK_F32, V_MADAK_F32 (replaced by FMA equivalents)

{{< ref >}}

 * [Vega_7nm_Shader_ISA.pdf](https://developer.amd.com/wp-content/resources/Vega_7nm_Shader_ISA.pdf)
 * [RDNA_Shader_ISA.pdf](https://developer.amd.com/wp-content/resources/RDNA_Shader_ISA.pdf)
 * [AMD PowerPoint- White Template - RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf)

{{< /ref >}}
