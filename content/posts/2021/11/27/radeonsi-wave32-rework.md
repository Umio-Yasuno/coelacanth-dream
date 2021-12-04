---
title: "RadeonSIドライバーで一部シェーダーを再び Wave32モードで実行するように"
date: 2021-11-27T00:00:00+09:00
draft: false
tags: [ "RadeonSI", "RDNA", "RDNA_2", "NGG" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

1年半近く前 (2020/07) に AMD のソフトウェアエンジニア、[Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏によって投稿されたパッチから、**RadeonSI (OpenGL)** ドライバーでは、Wave32 に対応する RDNA系アーキテクチャであっても、各シェーダーステージを Wave64モードで実行するようになっていた。[^radeonsi-wave64]  
{{< link >}} [RadeonSI ドライバーでは RDNA GPU も Wave64モードで各シェーダーを実行するように | Coelacanth's Dream](/posts/2020/07/02/radeonsi-shader-wave64-with-rdna/) {{< /link >}}

[^radeonsi-wave64]: [ac,radeonsi: use Wave64 for HS/GS/VS, gpu_info fix (!5524) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5524)

それが今回、いくつかのマージリクエストに分割された新たなパッチによって、一部のシェーダーでは再び Wave32モードで実行されるようになり、同時に Wave32モードへの最適化も行われた。  
シェーダーステージによって Wave32/64 を切り替えるのではなく、Workgroup (Thread Block) のサイズが小さい場合は Wave32モード、またプロファイリングによる結果から特定のシェーダーは Wave64モードで実行する等、きめ細かい切り替えが行われるようになるため、フラグの最適化も施されている。  

 * [mesa,nir: propagate shader source SHA1 to gallium drivers, a precursor to shader profiles (!13869) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13869)
 * [radeonsi: rework to make the wave size configurable per shader for fine-grained Wave32/64 selection (!13878) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13878)
 * [nir,glsl: serialize divergent fields, handle more instrinsics in divergence analysis, add nir_has_divergent_loop, fix source_sha1 (!13890) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13890)
 * [radeonsi: add Wave32 heuristics and shader profiles (!13966) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13966)

[Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏は、シェーダープロファイルと発見 (heuristics) による手直し、クリーンアップだとコメントしている。  

 > 		This is a rework and cleanup to be able to enable Wave32 based on shader profiles and heuristics.  
 > 		No functional change.
 >
 > {{< quote >}} [radeonsi: rework to make the wave size configurable per shader for fine-grained Wave32/64 selection (!13878) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13878) {{< /quote >}}

{{< pindex >}}
 * [Wave32](#wave32)
    * [Compute Shader](#cs)
    * [Pixel Shader](#ps)
    * [Divergent loops](#divergent)
    * [Shader Profile](#shader-profile)
{{< /pindex >}}

## Wave32 {#wave32}

各シェーダーステージを Wave64 で実行するようにした際、[Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏 Wave32 より性能面で優れていると考えた理由として以下の項目を挙げていた。  

 > * greater chance of L0 cache hits, because more threads are assigned to the same CU  
 > * scalar instructions are only executed once for 64 threads instead of twice  
 > * VGPR allocation granularity is half of Wave32, so 1 Wave64 can sometimes use fewer VGPRs than 2 Wave32  
 > * TessMark X64 with NGG culling is faster with Wave64  
 >
 > {{< quote >}} [ac,radeonsi: use Wave64 for HS/GS/VS, gpu_info fix (!5524) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5524) {{< /quote >}}

これはコード内にコメントで残されていたが、今回のリワークに伴い、実際のところは正しくなかった (*The big comment was not really true.*) として削除されている。[^wave32-true]  

[^wave32-true]: [radeonsi: rework to make the wave size configurable per shader for fine-grained Wave32/64 selection (!13878) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13878/diffs?commit_id=676d4ddcf83a62973aad8062a34c7c838bfc8a4f#68ea09d61709a8690d922a47246019c3819579d7)

### Compute Shader {#cs}

[RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf) では元より Compute Shader は通常 Wave32 を選択されるとしていたが、Wave32 で実行すると OpenGL のテストスイートである [piglit](https://gitlab.freedesktop.org/mesa/piglit) が失敗することから **RadeonSI** では Wave64 で実行するようになっていた。[^piglit-fail-cs32]  

[^piglit-fail-cs32]: [radeonsi/gfx10: enable Wave32 for vertex, geometry, and tessellation shaders (a0d330be) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/a0d330bedb9eb5668bc73c60e525f3c76d23a93a)

それが今回、Workgroup のサイズが小さい場合は Wave32 で実行するように変更された。  
Workgroup が十分に大きく、かつ Waveサイズを指定するデバッグフラグが設定されていない場合は、以前同様 Compute Shader は Wave64 で実行される。  

 > 		+   /* Small workgroups use Wave32 unconditionally. */
 > 		+   if (stage == MESA_SHADER_COMPUTE && info &&
 > 		+       !info->base.workgroup_size_variable &&
 > 		+       info->base.workgroup_size[0] *
 > 		+       info->base.workgroup_size[1] *
 > 		+       info->base.workgroup_size[2] <= 32)
 > 		+      return 32;
 >
 > {{< quote >}} [radeonsi: add Wave32 heuristics and shader profiles (!13966) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13966/diffs?commit_id=d5d3106ddde9b711ed3231c91f97a5a407836ba6#91b7eb4e8feffa75abd6fdc6f3cd66c9ccd30707) {{< /quote >}}

### Pixel Shader {#ps}

[RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf) では Pixel Shader のみ、通常 Wave64 が選択されるとしていたが、今回で一部は Pixel Shader も Wave32 で実行されるようになった。  

LLVM 13/14 では、ピクセルの廃棄処理 (discard) を含む Pixel Shader を Wave32モードでコンパイルすると、エラーを起こすバグがあるため、Wave64 で実行するとしている。  
新たに追加されたデバッグオプション `w32psdiscard (W32_PS_DISCARD)` は、そのエラーを無視して Wave32モードでコンパイル、実行するためのもの。  

 > 		+   /* LLVM 13 and 14 have a bug that causes compile failures with discard in Wave32
 > 		+    * in some cases. Alpha test in Wave32 is luckily unaffected.
 > 		+    */
 > 		+   if (stage == MESA_SHADER_FRAGMENT && info->base.fs.uses_discard &&
 > 		+       !(info && info->options & SI_PROFILE_IGNORE_LLVM_DISCARD_BUG) &&
 > 		+       LLVM_VERSION_MAJOR >= 13 && !(sscreen->debug_flags & DBG(W32_PS_DISCARD)))
 > 		+      return 64;
 >
 > {{< quote >}} [radeonsi: add Wave32 heuristics and shader profiles (!13966) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13966/diffs?commit_id=d5d3106ddde9b711ed3231c91f97a5a407836ba6#91b7eb4e8feffa75abd6fdc6f3cd66c9ccd30707) {{< /quote >}}

そして補間命令の無い Pixel Shader は、Wave32モードにおいて補間性能の低下を受けないため Wave32 で実行する。  
この方針は [GpuTest](https://www.geeks3d.com/gputest/) の PixMark Piano/Voloplosion にて役立つとしている。  

 > 		+   /* Pixel shaders without interp instructions don't suffer from reduced interpolation
 > 		+    * performance in Wave32, so use Wave32. This helps Piano and Voloplosion.
 > 		+    */
 > 		+   if (stage == MESA_SHADER_FRAGMENT && !info->num_inputs)
 > 		+      return 32;
 > 		+
 >
 > {{< quote >}} [radeonsi: add Wave32 heuristics and shader profiles (!13966) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13966/diffs?commit_id=d5d3106ddde9b711ed3231c91f97a5a407836ba6#91b7eb4e8feffa75abd6fdc6f3cd66c9ccd30707) {{< /quote >}}

### Divergent loops {#divergent}

次の部分は少し複雑で、自分には追いつけない部分がある。  
Tessellation Control Shader または Geometry Shader を実行するが、結果を GS copy shader で Pixel Shader に渡すことはしない場合を `merged_shader (true)` とし、  
その条件から外れると同時に、(分岐による？) 発散性 (divergent) のあるループは Wave32 で実行するようになった。  

 * 参考: [GPU向けコンパイラの最適化の紹介と論文のサーベイ - Jicchoの箱](https://juln.hatenablog.com/entry/2021/05/13/181020)

 > 		   /* TODO: Merged shaders must use the same wave size because the driver doesn't recompile
 > 		    * individual shaders of merged shaders to match the wave size between them.
 > 		    */
 > 		   bool merged_shader = sscreen->info.chip_class >= GFX9 && shader && !shader->is_gs_copy_shader &&
 > 		                        (shader->key.ge.as_ls || shader->key.ge.as_es ||
 > 		                         stage == MESA_SHADER_TESS_CTRL || stage == MESA_SHADER_GEOMETRY);
 > 		
 > 		   /* Divergent loops in Wave64 can end up having too many iterations in one half of the wave
 > 		    * while the other half is idling but occupying VGPRs, preventing other waves from launching.
 > 		    * Wave32 eliminates the idling half to allow the next wave to start.
 > 		    */
 > 		   if (!merged_shader && info && info->has_divergent_loop)
 > 		      return 32;
 >
 > {{< quote >}} [radeonsi: add Wave32 heuristics and shader profiles (!13966) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13966/diffs?commit_id=d5d3106ddde9b711ed3231c91f97a5a407836ba6#91b7eb4e8feffa75abd6fdc6f3cd66c9ccd30707) {{< /quote >}}


`merged_shader` の判定に `sscreen->info.chip_class >= GFX9` があるが、Wave のサイズを返すこの `si_determine_wave_size` 関数では GFX9 かそれ以前の世代の場合 64 (threads) が返される。[^return-64]  

[^return-64]: <https://gitlab.freedesktop.org/mesa/mesa/-/blob/d5d3106ddde9b711ed3231c91f97a5a407836ba6/src/gallium/drivers/radeonsi/si_state_shaders.cpp#L46-47>

コメント部の説明と自分の推測による補間を合わせるに、Wave内スレッドの一部が分岐するループを実行する場合、Wave64 だと Wave中の半分が反復する回数が多く、残り半分の Wave がベクタレジスタ (VGPR) を占有したままアイドリング状態となり、次の Wave の実行が妨げられてしまう。  
Wave32 ではアイドリング状態になる半分を排除でき、次の Wave を実行可能になる。  

ここでの Wave64 の半分というのは、*RDNA系 アーキテクチャ* における Wave64 の実行方式を指していると思われる。  
*RDNA系 アーキテクチャ* では Wave64モードで実行する場合、命令を 2つの Wave32 に分割して実行する。  
命令をある程度ブロックにまとめ、それを low と high に分割し、最初の low 群を実行、次に high 群を実行する Sub-Vectorモードも *RDNA系 アーキテクチャ* ではサポートされている。  
2つの Wave32 に分割されるといっても、スカラ命令は 1回のみとなる。また Workgroup の最大サイズはスレッド (work-item) 数で決まっているため、Workgroup 内の Wave数は Wave32 の場合は 32 Wave、Wave64 の場合は 16 Wave となる。  

### Shader Profile {#shader-profile}

シェーダープロファイルによる Wave サイズの選択も、現時点では SPECviewperfベンチマークのみが対象だが、実装されている。  
[Viewperf/Energy](https://www.spec.org/gwpg/gpc.static/energy-02.html) は Pixel Shader の部で挙げた Discard処理を含む Pixel Shader のコンパイルエラーが起きないため Wave32 で実行され、  
[Viewperf/Medical](https://www.spec.org/gwpg/gpc.static/med-02.html) は分岐発散のあるループにおいて上記の Wave32 の恩恵を受けられないため、Wave64 で実行される。プロファイリングの結果による選択とされるが、Marek Olšák 氏はおそらく補間性能の影響を受けているのではないかとしている。  

 > 		+struct si_shader_profile {
 > 		+   uint32_t sha1[SHA1_DIGEST_LENGTH32];
 > 		+   uint32_t options;
 > 		+};
 > 		+
 > 		+static struct si_shader_profile profiles[] =
 > 		+{
 > 		+   {
 > 		+      /* Viewperf/Energy isn't affected by the discard bug. */
 > 		+      {0x17118671, 0xd0102e0c, 0x947f3592, 0xb2057e7b, 0x4da5d9b0},
 > 		+      SI_PROFILE_IGNORE_LLVM_DISCARD_BUG,
 > 		+   },
 > 		+   {
 > 		+      /* Viewperf/Medical, a shader with a divergent loop doesn't benefit from Wave32,
 > 		+       * probably due to interpolation performance.
 > 		+       */
 > 		+      {0x29f0f4a0, 0x0672258d, 0x47ccdcfd, 0x31e67dcc, 0xdcb1fda8},
 > 		+      SI_PROFILE_WAVE64,
 > 		+   },
 > 		+};
 >
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13966/diffs?commit_id=d5d3106ddde9b711ed3231c91f97a5a407836ba6#diff-content-3219867c408aa0515182e3168fad726f47209435> {{< /quote >}}

[NGG (Next Generation Geometry)](/tags/ngg) を使用しない *Legacy GS (pipeline)* は Wave64 のみでサポートされるが、*RDNA系 アーキテクチャ (GFX10 <=)* では *Navi14* 以外はデフォルトで *NGG* が有効化される。  
そう深くコードを追ってはいないが、*NGG* 有効の場合は Wave32 で実行されるものと思われる。  

 > 		   /* Legacy GS only supports Wave64. */
 > 		   if ((stage == MESA_SHADER_VERTEX && shader->key.ge.as_es && !shader->key.ge.as_ngg) ||
 > 		       (stage == MESA_SHADER_TESS_EVAL && shader->key.ge.as_es && !shader->key.ge.as_ngg) ||
 > 		       (stage == MESA_SHADER_GEOMETRY && !shader->key.ge.as_ngg))
 > 		      return 64;
 >
 > {{< quote >}} <https://gitlab.freedesktop.org/mesa/mesa/-/blob/d5d3106ddde9b711ed3231c91f97a5a407836ba6/src/gallium/drivers/radeonsi/si_state_shaders.cpp#L49-53> {{< /quote >}}

{{< ref >}}
 * [RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf)
 * [床井研究室 - 第３回 シェーダプログラム](https://marina.sys.wakayama-u.ac.jp/~tokoi/?date=20090827)
 * [GPU向けコンパイラの最適化の紹介と論文のサーベイ - Jicchoの箱](https://juln.hatenablog.com/entry/2021/05/13/181020)
 * [Divergent Branch](https://docs.nvidia.com/gameworks/content/developertools/desktop/analysis/report/cudaexperiments/sourcelevel/divergentbranch.htm)
{{< /ref >}}
