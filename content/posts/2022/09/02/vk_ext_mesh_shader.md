---
title: "Vulkan Mesh Shader のクロスベンダ拡張 VK_EXT_mesh_shader がリリース"
date: 2022-09-02T14:00:17+09:00
draft: false
categories: [ "Software", "AMD", "Intel", "GPU" ]
tags: [ "RDNA_2", "Xe-HPG", "RADV" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Vulkan API における Mesh Shader のクロスベンダ拡張 `VK_EXT_mesh_shader` がリリースされた。  

 * [Mesh Shading for Vulkan - Khronos Blog - The Khronos Group Inc](https://www.khronos.org/blog/mesh-shading-for-vulkan)
 * [Vulkan-Docs/VK_EXT_mesh_shader.adoc at main · KhronosGroup/Vulkan-Docs](https://github.com/KhronosGroup/Vulkan-Docs/blob/main/proposals/VK_EXT_mesh_shader.adoc)

Mesh Shader にはクロスベンダ拡張よりも前に `VK_NV_mesh_shader` が定義されていたが、`VK_NV_mesh_shader` は DirectX 12 版と比較した時に主要な違いがあり、互換性において難を抱えていた。  
このことは *RADV (Vulkan)* ドライバーに `VK_NV_mesh_shader` の試験的なサポートを追加する際、Timur Kristóf 氏が解説している。[^radv-ms]  
`VK_EXT_mesh_shader` は DirectX 12 版との互換性を重視し、同時にいくつかのデバイスプロパティが追加されている。  

[^radv-ms]: [Mesh Shader への対応が進む RADVドライバー | Coelacanth's Dream](/posts/2021/12/20/radv-mesh-shader/)

Khronos Blog では、互換性を重視したことで Vulkan と DirectX 12 間の移植性の改善は達成したが、ベンダ間の性能という点は移植がずっと難しく、注意が必要だとしている。`KHR` 拡張としてリリースされなかった理由が性能の問題だとも語っている。  

## RADV, ANV {#vk-drv}
Mesa3D の AMD GPU 向け Vulkan ドライバー *RADV* と Intel GPU 向けである *ANV (Anvil)* では現在開発者たちによって `VK_EXT_mesh_shader` への対応が進められている。  

 * [WIP: spirv, nir: Add support for VK_EXT_mesh_shader (!18366) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/18366)
 * [WIP: radv: Add support for VK_EXT_mesh_shader (!18367) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/18367)
 * [anv: add support for VK_EXT_mesh_shader (!18371) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/18371)

*RADV* における実装では、Kernel Mode Driver (KMD) 側の AMDGPU ドライバーに *Gang Submit* が実装されていないと安全に動作しないため、環境変数を設定しないと `VK_EXT_mesh_shader` 対応が有効化されないと Timur Kristóf 氏はコメントしている。  
*Gang Submit* は以前に取り上げたが、AMD のソフトウェアエンジニア Christian König 氏によって実装が進められており、複数の IB (Indirect Buffer) を複数の異なるエンジンで同時に実行可能であることを保証する機能となる。[^gang-submit]  
複数のプロセスが同時に Task Shader を使用した場合、GPU がデッドロックする可能性があるが、*Gang Submit* があれば効果的に防止することが可能となる。  

[^gang-submit]: [AMDGPUドライバーに「Gang Submit」を実装するパッチ | Coelacanth's Dream](http://localhost:1313/posts/2022/03/04/amdgpu-gang-submission/)
