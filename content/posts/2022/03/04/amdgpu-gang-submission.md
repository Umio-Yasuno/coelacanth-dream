---
title: "AMDGPUドライバーに「Gang Submit」を実装するパッチ"
date: 2022-03-04T17:53:25+09:00
draft: false
categories: [ "Software", "AMD", "GPU", "APU" ]
tags: [ "Linux_Kernel", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD のソフトウェアエンジニア [Christian König](https://de.linkedin.com/in/christian-k%C3%B6nig-35b7bbaa) 氏より、AMDGPUドライバーの CS (Command Submission) インターフェイスに *Gang submit/submission(s)* と呼ぶ機能を実装するパッチが投稿されている。  
Christian König 氏は DRM (Direct Rendering Manager) やそのメモリ管理機能に多くのパッチを投稿しており、過去には Resizeable PCI BAR の実装も行っている。  

* [Gang submit](https://lists.freedesktop.org/archives/amd-gfx/2022-March/076261.html)

*Gang Submit* は複数の IB (Indirect Buffer) を異なるエンジンで同時に実行可能であることを保証する機能だと、Christian König 氏は説明している。  
IB は特定のエンジンに対するコマンドバッファであり、コマンドはリングバッファで管理される。  
また、ここでのエンジンは、GFX, Compute, SDMA, Multi Media (UVD/VCE/VCN) を指していると思われる。  

 > 		const unsigned int amdgpu_ctx_num_entities[AMDGPU_HW_IP_NUM] = {
 > 			[AMDGPU_HW_IP_GFX]	=	1,
 > 			[AMDGPU_HW_IP_COMPUTE]	=	4,
 > 			[AMDGPU_HW_IP_DMA]	=	2,
 > 			[AMDGPU_HW_IP_UVD]	=	1,
 > 			[AMDGPU_HW_IP_VCE]	=	1,
 > 			[AMDGPU_HW_IP_UVD_ENC]	=	1,
 > 			[AMDGPU_HW_IP_VCN_DEC]	=	1,
 > 			[AMDGPU_HW_IP_VCN_ENC]	=	1,
 > 			[AMDGPU_HW_IP_VCN_JPEG]	=	1,
 > 		};
 >
 > {{< quote >}} [linux/amdgpu_ctx.c at 7d7630fc6b8850ceae5a708bd37dcc7583658316 · torvalds/linux](https://github.com/torvalds/linux/blob/7d7630fc6b8850ceae5a708bd37dcc7583658316/drivers/gpu/drm/amd/amdgpu/amdgpu_ctx.c#L34-L44) {{< /quote >}}

同じ *Gang* に属するメンバー (IB を指していると思われる) は、暗黙的、明示的にも同じ VM の依存関係を取得し、他のメンバーが準備を完了するまでメンバーは実行されない。  
これによりデッドロックを効果的に防止できると説明されている。  

*Gang Submit* を実装する一連のパッチは、Christian König 氏が作成した他のパッチをベースにしているため、どのブランチにも容易に適用できるものではなく、実際に取り込まれるまでには時間が掛かると思われる。  

{{< ref >}}
* [Secure Buffer Support with Trusted Memory Zone - xdc2020_rayhuang_secure_buffer_with_tmz.pdf](https://lpc.events/event/9/contributions/630/attachments/703/1300/xdc2020_rayhuang_secure_buffer_with_tmz.pdf)
* [drm/amdgpu AMDgpu driver — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/)
{{< /ref >}}
