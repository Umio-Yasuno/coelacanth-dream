---
title: "Mesh Shader への対応が進む RADVドライバー"
# date: 2021-12-18T13:19:09+09:00
date: 2021-12-20T00:00:00+09:00
draft: false
tags: [ "RADV", "RDNA_2", "NGG", "ACO" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

オープンソースで開発されている AMD GPU向け Vulkanドライバー *RADV* 、そのバックエンドである *ACO* で Mesh Shader への対応が進められている。  
[Timur Kristóf](https://gitlab.freedesktop.org/Venemo) 氏は該当マージリクエスト内で、Mesh Shader と Vulkan における拡張機能 `NV_mesh_shader`、そして RADV/ACO での実装について詳細を語っている。  

 * [nir, spirv: Mesh Shader I/O fixes (!13466) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13466)
 * [radv: Experimental support for Mesh Shaders. (!13580) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13580)
 * [radv: Add VRS support for mesh shaders. (!14193) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/14193)

{{< pindex >}}
 * [Mesh Shader](#ms)
    * [VK_NV_mesh_shader](#nv-ms)
 * [NGG](#ngg)
{{< /pindex >}}

## Mesh Shader {#ms}

Mesh Shader は、Vertex/Geometry パイプライン全体を置き換え可能な Compute Shader に近いシェーダーステージであり、指定した入力データから任意に頂点とプリミティブを作成することができる  
Mesh Shader では、頂点数によらず、指定された数の Workgroup (Threadgroup) が発行され、描画を行う。頂点とプリミティブを出力するが、Workgroup内のスレッドと出力する頂点数、プリミティブ数との間に関連付けはされない。これにより SIMDユニットにおけるレーンの無駄遣いを減らすことができる。  

Mesh Shader のハードウェアサポートは、NVIDIA GPU では Turing世代から、AMD GPU では *RDNA 2/GFX10.3* 世代から実装されている。Intel GPU では *Alchemist/DG2 ({{< xe class="hpg" >}})* からサポートされる。  

### VK_NV_mesh_shader {#nv-ms}

現時点で、Vulakn API では `VK_NV_mesh_shader` 拡張機能が定義されており、RADV/ACO ではそのサポートを進めているが、これは Mesh Shader サポート実装の経験を積むためであり、公式的にサポートすることは無いと Timur Kristóf 氏はコメントしている。  
クロスベンダーバージョンの拡張機能がリリースされ次第、`VK_NV_mesh_shader` のサポートは取り除く予定だとしているが、リリースがいつになるかは不明。  
以下の issue で議論されているが、進展については特に報告されていない。  

 * [Cross-vendor version of `VK_NV_mesh_shader` · Issue #1423 · KhronosGroup/Vulkan-Docs](https://github.com/KhronosGroup/Vulkan-Docs/issues/1423)

Timur Kristóf 氏は RADV/ACO で `VK_NV_mesh_shader` を公式にサポートしない理由として、AMD GPU上では性能が低いことが挙げている。  
同時に、`VK_NV_mesh_shader` には D3D12 (Direct3D 12) と比較して、以下のような問題があることを指摘している。  
日本語部は自分によるざっくりとした意訳。  

 > * The total number of output vertices is not known in runtime. D3D12 solves this with SetMeshOutputCounts which must appear before any outputs are written. NV_mesh_shader doesn't have this guarantee.
 > * Any shader invocation can read the output of any other which is not possible in D3D12.
 > * The NV indirect command buffer format is not supported by the hardware, so we have to emit several copy packets to make it work. Note that D3D12 uses 3D dispatches without an offset: (x, y, z) but NV_mesh_shader uses an 1D dispatch with offset: (taskCount, firstTask).
 >
 > {{< quote >}} [radv: Experimental support for Mesh Shaders. (!13580) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13580) {{< /quote >}}

 * D3D12 では出力する頂点の総数を、出力を書き込む前に `SetMeshOutputCounts` を使って表示することを必須としているが、`VK_NV_mesh_shader` ではそうした機能がなく、頂点の総数を実行時に知ることができない。  
 * D3D12 では不可能にされている、あるシェーダーから他のシェーダーの出力を読み取ることができるようになっている。  
 * NV間接コマンドバッファフォーマットが (AMD GPU) ハードウェアでサポートされていないため、複数のコピーパケットを発行する必要がある。D3D12 ではオフセット無しの 3次元ディスパッチ (x, y, z) を使用するが、`VK_NV_mesh_shader` ではオフセット有りの 1次元ディスパッチ (taskCount, firstTask) を使用する。  

以上のことから、`VK_NV_mesh_shader` では D3D12 と比較して性能が低くなるとしている。  
クロスベンダー拡張でこれらの問題が解決され、D3D12 との互換性がより取られれば、AMD GPU での性能も改善することが期待される。  

## NGG {#ngg}

RADV/ACO では *RDNA/GFX10.1* 世代から有効化されている *NGG (Next Generation Geometry)* ステージを使って Mesh Shader を実行する。  
このことは同じく Timur Kristóf 氏による XDC2020 (2020/09) での発表でも触れられており、その時は *Navi10* でも実行できる可能性があるとしていた。Mesh Shader のクロスベンダー拡張が無いこともこの時に指摘されている。[^xdc2020-aco]  

[^xdc2020-aco]: [X.Org Developers Conference 2020 (16-18 September 2020): A year of ACO: from prototype to default · Indico](https://xdc2020.x.org/event/9/contributions/612/)

だがパッチでは、動作するのは *RDNA 2/GFX10.3* とそれ以降だけとして、*RDNA/GFX10.1* でのサポートは行われていない。  

 > 		+   /* Mesh shading only works on GFX10.3+. */
 > 		+   ASSERTED bool mesh_shading = ctx.stage.has(SWStage::TS) || ctx.stage.has(SWStage::MS);
 > 		+   assert(!mesh_shading || ctx.program->chip_class >= GFX10_3);
 > 		+
 >
 > {{< quote >}} [radv: Experimental support for Mesh Shaders. (!13580) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13580/diffs?commit_id=c2667723a588e14615a6d0cf28a5936ce69e0ff8#diff-content-d0488ddd1da2786fc08fb3c5c5c0b5b0329dc0cd) {{< /quote >}}

*NGG* は Pixel Shader の入力に使うリングバッファ、Parameter Cache (PC) に直接 Geometry Shader の結果を出力することができる。*RDNA/GFX10.1 + NGG* 以前の AMD GPU ではこれができず、一度 Geometry Shader の結果を VRAM に書き込み、そこからコピーする必要があった。  

RADV/ACO の実装では、Task Shader を Compute Shader として実行、結果は VRAM に出力、格納される。  
Mesh Shader は入力を VRAM から読み込み、結果は LDS、それから PC に格納。  
Pixel Shader はこれまでと変わらず、PC から入力を読み込み、実行される。  

 > 		
 > 		+#### Mesh Shading Graphics Pipeline
 > 		+
 > 		+GFX10.3+:
 > 		+
 > 		+* TS will run as a CS and stores its output payload to VRAM
 > 		+* MS runs on NGG, loads its inputs from VRAM and stores outputs to LDS, then PC
 > 		+* Pixel Shaders work the same way as before
 > 		+
 > 		+| GFX10.3+ HW stages      | CS    | NGG   | PS | ACO terminology |
 > 		+| -----------------------:|:------|:------|:---|:----------------|
 > 		+| SW stages: only MS+PS:  |       | MS    | FS | `mesh_ngg`, `fragment_fs` |
 > 		+|            with task:   | TS    | MS    | FS | `task_cs`, `mesh_ngg`, `fragment_fs` |
 > 		+
 >
 > {{< quote >}} [radv: Experimental support for Mesh Shaders. (!13580) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/13580/diffs?commit_id=1b27d018a03656021a273280e9bd3d26a945dee9) {{< /quote >}}

{{< ref >}}
 * [VK_NV_mesh_shader(3)](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VK_NV_mesh_shader.html)
 * [X.Org Developers Conference 2020 (16-18 September 2020): A year of ACO: from prototype to default · Indico](https://xdc2020.x.org/event/9/contributions/612/)
 * [Mesh Shader | DirectX-Specs](https://microsoft.github.io/DirectX-Specs/d3d/MeshShader.html)
 * [Coming to DirectX 12— Mesh Shaders and Amplification Shaders: Reinventing the Geometry Pipeline   - DirectX Developer Blog](https://devblogs.microsoft.com/directx/coming-to-directx-12-mesh-shaders-and-amplification-shaders-reinventing-the-geometry-pipeline/)
 * [PowerPoint Presentation - AMD_RDNA2_DirectX12_Ultimate_SamplerFeedbackMeshShaders.pdf](https://gpuopen.com/wp-content/uploads/slides/AMD_RDNA2_DirectX12_Ultimate_SamplerFeedbackMeshShaders.pdf)
{{< / >}}
