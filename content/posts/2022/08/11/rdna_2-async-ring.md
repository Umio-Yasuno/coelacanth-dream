---
title: "AMDGPUドライバーにて RDNA 2 アーキテクチャの GFX Async Ring が有効化"
date: 2022-08-11T09:07:41+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "RDNA_2", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Linux Kernel の次回バージョンアップ、5.20/6.0 に向けた AMDGPUドライバーの更新では、*RDNA 2/GFX10.3 アーキテクチャ* の GFX Async Ring (MicroEngine1, Pipe1) を有効化するパッチが取り込まれる。  

 * [[pull] amdgpu, amdkfd, radeon drm-next-5.20](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081068.html)

 > 		Arunpravin Paneer Selvam (3):
 > 		      drm/amd/amdgpu: Enable high priority gfx queue
 > 		      drm/amd/amdgpu: add pipe1 hardware support
 > 		      drm/amd/amdgpu: Fix alignment issue
 >
 > {{< quote >}} [[pull] amdgpu, amdkfd, radeon drm-next-5.20](https://lists.freedesktop.org/archives/amd-gfx/2022-July/081068.html) {{< /quote >}}

## GFX Async Ring {#async-ring}
GFX Ring/ME は AMD GPU に搭載されているマイクロコントローラの一種であり、GFX エンジンへのグラフィクスキューをコントロールする役割を持つ。  
Compute エンジン、コンピュートキュー用のマイクロコントローラは MEC (MicroEngine Compute) とされている。  
ME と MEC はハードウェアブロックとしては GFX/Compute パイプラインのフロントエンドに位置する。  

*GFX9/Vega* 世代までは GFX Ring/ME は通常 1基の構成となっており、*GFX10/RDNA 1* 世代からは優先度が高く設定されたグラフィクスキュー向けに GFX Async Ring が追加された。  
しかし *GFX10/RDNA 1* 世代では GFX Async Ring は無効化されており[^navi1x-pipe1]、これはその後に AMDGPUドライバーへのパッチが初めて公開された *Sienna Cichlid/Navi21*、*RDNA 2/GFX10.3* 世代においても同様だった。[^rdna_2-pipe1]  

そうした状態だったが、*RDNA 2/GFX10.3* 世代の GFX Async Ring が Linux Kernel 5.20/6.0 では有効化される。  
パッチでは *RDNA 2/GFX10.3 APU* も対象に含まれている。  
*GFX10/RDNA 1* 世代の GFX Async Ring を無効化するパッチは 2020/03/02、*Sienna Cichlid/Navi21* へのパッチは 2020/06/01 に投稿されたため、約 2年越しの有効化となる。  

 * [drm/amd/amdgpu: Enable high priority gfx queue (b07d1d73) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/b07d1d73b09ef40e91ace51a2e167391676a8175)
 * [drm/amd/amdgpu: add pipe1 hardware support (4c763180) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/4c7631800e6bf0eced08dd7b4f793fcd972f597d)

 > 		Starting from SIENNA CICHLID asic supports two gfx pipes, enabling
 > 		two graphics queues, 1 on each pipe, pipe0 queue0 would be the normal
 > 		piority queue and pipe1 queue0 would be the high priority queue
 > 		
 > 		Only one queue per pipe is visble to SPI, SPI looks at the priority
 > 		value assigned to CP_GFX_HQD_QUEUE_PRIORITY from each of the queue's
 > 		HQD/MQD.
 > 		
 > 		Create contexts applying AMDGPU_CTX_PRIORITY_HIGH which submits job
 > 		to the high priority queue on GFX pipe1. There would be starvation
 > 		of LP workload if HP workload is always available.
 >
 > {{< quote >}} [drm/amd/amdgpu: Enable high priority gfx queue (b07d1d73) · Commits · Alex Deucher / linux · GitLab](https://gitlab.freedesktop.org/agd5f/linux/-/commit/b07d1d73b09ef40e91ace51a2e167391676a8175) {{< /quote >}}

User Mode Driver 側では *RadeonSI (OpenGL)* ドライバーに `EGL_IMG_context_priority` のサポートを追加し、優先度の高いコンテキストの作成を可能にするパッチ、マージリクエストが公開されているが、これも GFX Async Ring の有効化と関係しているのではないかと思われる。  
パッチはすでに main ブランチに取り込まれている。  

 * [radeonsi: Add support for EGL_IMG_context_priority (!16594) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/16594)

*RadeonSI* へのパッチを投稿した Vlad Zahorodnii 氏は、高い優先度を設定する主な対象として、Wayland コンポジターのようなレンダリングコマンドをできる限り早くに処理する必要のアプリケーションを挙げている。  

[^rdna_2-pipe1]: [[PATCH 162/207] drm/amdgpu: only use one gfx pipe for Sienna_Cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050126.html)
[^navi1x-pipe1]: [[PATCH] drm/amdgpu: disable 3D pipe 1 on Navi1x](https://lists.freedesktop.org/archives/amd-gfx/2020-March/046692.html)

しかし AMD の Michel Dänzer 氏により、GFX Async Ring を有効化した *Sienna Cichlid/Navi21* GPU と AMDGPUドライバー、`EGL_IMG_context_priority` をサポートした *RadeonSI* ドライバーの組み合わせで GPU がハングアップする問題が報告されており、GFX Async Ring サポートの安定化まではまだ時間が必要と見られる。[^navi21-hang]  

[^navi21-hang]: [Navi 21 GFX hangs since "drm/amd/amdgpu: add pipe1 hardware support" (#2117) · Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues/2117)

{{< ref >}}
 * [Core Driver Infrastructure — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/driver-core.html#graphics-and-compute-microcontrollers)
{{< /ref >}}
