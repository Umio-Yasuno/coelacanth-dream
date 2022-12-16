---
title: "AMDVLK ドライバーが RDNA 3/GFX11, Navi31 に対応"
date: 2022-12-16T15:18:52+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", "GPUOpen" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

2022-12-15 付で、*RDNA 3/GFX11*、*Navi31* に対応した AMDVLK ドライバー v-2022.Q4.4 がリリースされた。  
AMDVLK を構成する各種ライブラリも同時に更新され、*Navi31* に対応するコミットが公開されている。  
AMDVLK ドライバーは AMD が公式にオープンソースで開発しており、ハードウェアに密接した情報がコードやコメントの形で公開されることがある。そうして得られた情報が Mesa3D RADV (Vulkan) ドライバーに取り込まれることもある。  

ここではその中から自分でもある程度読み取れる部分だけを取り上げる。  

 * [Release v-2022.Q4.4 · GPUOpen-Drivers/AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK/releases/tag/v-2022.Q4.4)
    * [Update llpc from commit 427fb590e · GPUOpen-Drivers/llpc@37dcb2e](https://github.com/GPUOpen-Drivers/llpc/commit/37dcb2e5cedb00bb025c84238d816f19c93b3060) 
    * [Update xgl from commit 79803ba8 · GPUOpen-Drivers/xgl@8aa0e76](https://github.com/GPUOpen-Drivers/xgl/commit/8aa0e76a110fa264608ee1b4e412aa8fb40286d3)
    * [Update pal from commit 1cecbdfb · GPUOpen-Drivers/pal@287ef68](https://github.com/GPUOpen-Drivers/pal/commit/287ef684bc36a86af55d4ed1c4c4f4c35577e21e)
    * [Update gpurt from commit 682bcc39 · GPUOpen-Drivers/gpurt@1f0c4f7](https://github.com/GPUOpen-Drivers/gpurt/commit/1f0c4f7e9cea22452e5e20a6cdfc4a84a2bf5bac)

{{< pindex >}}
 * [Mesh Shader](#ms)
 * [RT IP 2.0](#rtip2_0)
 * [DispatchInterleaveSize](#interleave)
 * [misc](#misc)
{{< /pindex >}}

## Mesh Shader {#ms}
*RDNA 3 アーキテクチャ* では Mesh Shader に関する改良も施され、Mesh Shader パイプラインで必要とする WorkgroupID を SGPR (スカラレジスタ) を介してハードウェアから返すことが可能になったため、*RDNA 2 アーキテクチャ* で必要としていた追加のエミュレーション処理を省くことができる。  

 >         +#if LLPC_BUILD_GFX11
 >         +  // NOTE: For GFX11+, HW will provide workgroup ID via SGPRs. We don't need flat workgroup ID to do emulation.
 >         +  if (pipelineState->getTargetInfo().getGfxIpVersion().major >= 11)
 >         +    return false;
 >         +#endif
 >
 > {{< quote >}} [Update llpc from commit 427fb590e · GPUOpen-Drivers/llpc@37dcb2e](https://github.com/GPUOpen-Drivers/llpc/commit/37dcb2e5cedb00bb025c84238d816f19c93b3060) {{< /quote >}}

## RT IP 2.0 {#rtip2_0}
AMD GPU の RayTracing Unit には、*RDNA 2 アーキテクチャ* では *RT IP 1.1* が採用されていたが、*RDNA 3 アーキテクチャ* ではバージョンが進んだ *RT IP 2.0* が採用されている。  
*RT IP 2.0* では BoxSort 機能への [Largest, MidPoint, LargestFirstOrClosest, LargestFirstOrClosestMidPoint] の追加、DXR Ray Flag に準拠したポインターフラグ機能が追加されている。  
RayTracing 関連では、BVH ノードスタックの管理を最適化する `ds_bvh_stack_rtn` 命令が追加されている。(`ds_` から始まる命令は共有メモリ {LDS/GDS} にデータを転送するための命令。)  

 >              RtIp1_0 = 0x1,     ///< First Implementation of HW RT
 >              RtIp1_1 = 0x2,     ///< Added computation of triangle barycentrics into HW
 >         +#if PAL_BUILD_GFX11
 >         +    RtIp2_0 = 0x3,     ///< Added more Hardware RayTracing features, such as BoxSort, PointerFlag, etc
 >         +#endif
 >
 > {{< quote >}} [Update pal from commit 1cecbdfb · GPUOpen-Drivers/pal@287ef68](https://github.com/GPUOpen-Drivers/pal/commit/287ef684bc36a86af55d4ed1c4c4f4c35577e21e) {{< /quote >}}
 >
 >          enum class BoxSortHeuristic : uint32
 >          {
 >              Closest = 0x0,
 >         +#if GPURT_BUILD_RTIP2
 >         +    Largest = 0x1,
 >         +    MidPoint = 0x2,
 >         +#endif
 >              Disabled = 0x3,
 >         +#if GPURT_BUILD_RTIP2
 >         +    LargestFirstOrClosest = 0x4,
 >         +    LargestFirstOrClosestMidPoint = 0x5,
 >         +#endif
 >              DisabledOnAcceptFirstHit = 0x6
 >          };
 >
 > {{< quote >}} [Update gpurt from commit 682bcc39 · GPUOpen-Drivers/gpurt@1f0c4f7](https://github.com/GPUOpen-Drivers/gpurt/commit/1f0c4f7e9cea22452e5e20a6cdfc4a84a2bf5bac) {{< /quote >}}
 >
 >         +#if GPURT_BUILD_RTIP2
 >         +//=====================================================================================================================
 >         +// Instance base pointer layout from the HW raytracing IP 2.0 spec:
 >         +// Zero                         [ 2: 0]
 >         +// Tree Base Address (64B index)[53: 3]
 >         +// Force Opaque                 [   54]
 >         +// Force Non-Opaque             [   55]
 >         +// Disable Triangle Cull        [   56]
 >         +// Flip Facedness               [   57]
 >         +// Cull Back Facing Triangles   [   58]
 >         +// Cull Front Facing Triangles  [   59]
 >         +// Cull Opaque                  [   60]
 >         +// Cull Non-Opaque              [   61]
 >         +// Skip Triangles               [   62]
 >         +// Skip Procedural              [   63]
 >         +//
 >         +// Since GPU VAs can only be 48 bits, only 42 bits of the Tree Base Address field are used:
 >         +// Used Address                 [44: 3]
 >         +// Unused Address               [53:45]
 >         +#endif
 >
 > {{< quote >}} [Update gpurt from commit 682bcc39 · GPUOpen-Drivers/gpurt@1f0c4f7](https://github.com/GPUOpen-Drivers/gpurt/commit/1f0c4f7e9cea22452e5e20a6cdfc4a84a2bf5bac) {{< /quote >}}

## DispatchInterleaveSize {#interleave}
ShaderEngine (SE) に送る Thread-group (Work-group) を制御する機能が *RDNA 3 アーキテクチャ* では追加されている。  
この機能は Compute Shader, Task Shader 用の機能とされ、グラフィクスパイプラインでは Task Shader が使用されていない場合は無視される。  

 >         +#if PAL_BUILD_GFX11
 >         +/// Constant used for dispactch shader engine interleave. Value is the number of thread groups sent to one SE before
 >         +/// switching to another.  Default is 64. Other programmable values are: 128, 256, or 512. You can also disable
 >         +/// interleaving altogether.
 >         +enum class DispatchInterleaveSize : uint32
 >         +{
 >         +    Default = 0x0,
 >         +    Disable = 0x1,
 >         +    _128    = 0x2,
 >         +    _256    = 0x3,
 >         +    _512    = 0x4,
 >         +    Count
 >         +};
 >         +#endif
 >
 > {{< quote >}} [Update pal from commit 1cecbdfb · GPUOpen-Drivers/pal@287ef68](https://github.com/GPUOpen-Drivers/pal/commit/287ef684bc36a86af55d4ed1c4c4f4c35577e21e) {{< /quote >}}
 >
 >         +#if PAL_BUILD_GFX11
 >         +    DispatchInterleaveSize    taskInterleaveSize; ///< Ignored for pipelines without a task shader. For pipelines with
 >         +                                                  ///  a task shader, controls how many thread groups are sent to one
 >         +                                                  ///  SE before switching to the next one.
 >         +#endif
 >
 > {{< quote >}} [Update pal from commit 1cecbdfb · GPUOpen-Drivers/pal@287ef68](https://github.com/GPUOpen-Drivers/pal/commit/287ef684bc36a86af55d4ed1c4c4f4c35577e21e) {{< /quote >}}

## misc
一部の Atomic 操作が *RDNA 3 アーキテクチャ* では削除されたらしいが理由は不明。  

 >         +#if PAL_BUILD_GFX11
 >         +    else if (IsGfx11(pInfo->gfxLevel))
 >         +    {
 >         +        pInfo->gfx9.supportAddrOffsetDumpAndSetShPkt = 1;
 >         +        pInfo->gfx9.supportAddrOffsetSetSh256Pkt     = (cpUcodeVersion >= Gfx10UcodeVersionSetShRegOffset256B);
 >         +        pInfo->gfx9.supportPostDepthCoverage         = 1;
 >         +
 >         +        //       FP32 image atomic operations are removed in Gfx11
 >         +        pInfo->gfxip.supportFloat32BufferAtomics     = 1;
 >         +        pInfo->gfxip.supportFloat32ImageAtomics      = 0;
 >         +        //       FP64 atomic operations are removed from GL2 in Gfx11
 >         +        pInfo->gfx9.supportFloat64Atomics            = 0;
 >         +
 > {{< quote >}} [Update pal from commit 1cecbdfb · GPUOpen-Drivers/pal@287ef68](https://github.com/GPUOpen-Drivers/pal/commit/287ef684bc36a86af55d4ed1c4c4f4c35577e21e) {{< /quote >}}

Mesa3D の RadeonSI (OpenGL) ドライバーでは *RDNA 3 アーキテクチャ*、*Navi31/33, Phoenix APU* の A0 リビジョンのいくつかでは命令のプリフェッチ機能が無効化するようにされているが[^prefetch]、AMDVLK ではそうした回避策が見られない。  
RadeonSI ドライバーでは正確には GPU の内部レジスタから読み出せる `rev_id` を参照し、`rev_id` が 0 の場合に無効化しているため、条件に当てはまらない *Navi31/33, Phoenix* もあるということだろうか。  
`rev_id` の値は RadeonSI (OpenGL) ドライバーであれば `AMD_DEBUG=info glxinfo -B` といったコマンドで確認することができる。  

 >         +        pInfo->gfxip.shaderPrefetchBytes    = 3 * ShaderICacheLineSize;
 >         +        pInfo->gfxip.supportsSwStrmout      = 1;
 >         +        pInfo->gfxip.supportsHwVs           = 0;
 >         +
 >         +        pInfo->gfxip.gl1cSizePerSa          = 256_KiB;
 >         +
 >         +    }
 >
 > {{< quote >}} [Update pal from commit 1cecbdfb · GPUOpen-Drivers/pal@287ef68](https://github.com/GPUOpen-Drivers/pal/commit/287ef684bc36a86af55d4ed1c4c4f4c35577e21e) {{< /quote >}}

[^prefetch]: [radeonsi/gfx11: enable shader prefetch except for initial chip revisions (2ed9eb1b) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/2ed9eb1b633f214ff8900ab3be9e639f87cebaef)

*RDNA 3* のハードウェアに存在するバグについて、自分なりに公開されている範囲で触れると、まず SGPR (スカラレジスタ) の初期化処理に関するバグは *Navi31 (GFX1100), Navi33 (GFX1102)* が影響を受ける。  
`V_MAD_U64/I64` 命令のフォワーディングに関するバグは現状 *RDNA 3* 全体で影響を受けるとしている。  

 >         def FeatureISAVersion11_0_0 : FeatureSet<
 >           !listconcat(FeatureISAVersion11_Common.Features,
 >             [FeatureGFX11FullVGPRs,
 >              FeatureUserSGPRInit16Bug])>;
 >         
 >         def FeatureISAVersion11_0_1 : FeatureSet<
 >           !listconcat(FeatureISAVersion11_Common.Features,
 >             [FeatureGFX11FullVGPRs])>;
 >         
 >         def FeatureISAVersion11_0_2 : FeatureSet<
 >           !listconcat(FeatureISAVersion11_Common.Features,
 >             [FeatureUserSGPRInit16Bug])>;
 >         
 >         def FeatureISAVersion11_0_3 : FeatureSet<
 >           !listconcat(FeatureISAVersion11_Common.Features,
 >             [])>;
 >
 > {{< quote >}} [llvm-project/AMDGPU.td at e58b116843315085363207f0c1be841b4da1b496 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/e58b116843315085363207f0c1be841b4da1b496/llvm/lib/Target/AMDGPU/AMDGPU.td#L1311-L1326) {{< /quote >}}
 >
 >          def FeatureUserSGPRInit16Bug : SubtargetFeature<"user-sgpr-init16-bug",
 >           "UserSGPRInit16Bug",
 >           "true",
 >           "Bug requiring at least 16 user+system SGPRs to be enabled"
 >         >;
 >
 > {{< quote >}} [llvm-project/AMDGPU.td at e58b116843315085363207f0c1be841b4da1b496 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/e58b116843315085363207f0c1be841b4da1b496/llvm/lib/Target/AMDGPU/AMDGPU.td#L180-L184) {{< /quote >}}
 >
 >         def FeatureMADIntraFwdBug : SubtargetFeature<"mad-intra-fwd-bug",
 >           "HasMADIntraFwdBug",
 >           "true",
 >           "MAD_U64/I64 intra instruction forwarding bug"
 >         >;
 >
 > {{< quote >}} [llvm-project/AMDGPU.td at e58b116843315085363207f0c1be841b4da1b496 · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/e58b116843315085363207f0c1be841b4da1b496/llvm/lib/Target/AMDGPU/AMDGPU.td#L282-L286) {{< /quote >}}

RadeonSI ドライバーで公開されているものでは、おそらくは最終的な色の出力機能に関するバグ (`export_conflict_bug`) があり、これも *RDNA 3* 全体が影響を受けるとしている。[^export-conflict]  
しかし、程度の差はあれどハードウェア的なバグは基本どの世代でも存在し、バグとそれに対するソフトウェア側の回避策が実性能にどれだけ影響するかはそれぞれ異なる。  
オープンソースソフトウェアの範囲で公開されている上記のバグが、どれだけ影響するかも不明である。  

[^export-conflict]: [ac,radeonsi: many changes incl. gfx11, some fixes (reviewed commits from !17782) (!17864) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/17864/diffs?commit_id=f129db911bdd58a0f02a2161b34b38d93ce54ab9)

{{< ref >}}
 * [RDNA 3 / Radeon RX 7000シリーズ Deep Dive - 新情報を基に内部構造と性能を解剖してみる | マイナビニュース](https://news.mynavi.jp/article/20221114-2512523/)
 * [RAY_FLAG enumeration - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3d12/ray_flag)
 * [radeonsi/gfx11: enable shader prefetch except for initial chip revisions (2ed9eb1b) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/2ed9eb1b633f214ff8900ab3be9e639f87cebaef)
 * [⚙ D133012 [AMDGPU] Add subtarget feature for MAD_U64/I64 bug on GFX11](https://reviews.llvm.org/D133012)
{{< /ref >}}
