---
title: "ROCm + RDNA 4 GPU で発生していたメモリアクセスエラーが修正される"
date: 2025-12-13T05:32:49+09:00
draft: false
categories: [ "Hardware", "Software", "AMD", "GPU" ]
tags: [ "ROCm", "RDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

ROCm + RDNA 4 GPU という環境で、数ヶ月に渡って発生していたメモリアクセスエラーがようやく修正されるかもしれない。  

## ランダムに発生するメモリアクセスエラー {#memory-error}
まず ROCm + RDNA 4 GPU では以前からランダムにメモリアクセスエラーが発生しており、安定性に問題を抱えていた。  
主に PyTorch を使用している際に発生するが、発生する頻度はほとんどランダムであり、発生するタイミングもカーネルを実行中、唐突に発生するということしか分からなかった。  
ログを取り、仔細に読み込んでも、割り当てていないメモリアドレスにアクセスしている以外には不明だった。  

 * [[Issue]: Memory access fault by GPU node-1 (Agent handle: 0x5de975b3cc80) on address 0x799bfc063000 · Issue #5051 · ROCm/ROCm](https://github.com/ROCm/ROCm/issues/5051)

自分が RX 9060 XT 16GB (gfx1200) で試した限りでは ROCm 6.4.1 では発生せず、ROCm 7.0rc から発生するようになったが、  
また、Vulkan API を使用したゲームをプレイ中に発生することは一度もなかった。  

## 回避策と修正 {#wa-fix}
問題が報告されてから、しばらくの間は特に進展が無かったが、最近になって Linux Kernel のブートパラメータによる回避策が見つかった。  
`amdgpu.cwsr_enable=0` をブートパラメータに指定するというものであり、自分が探した限り、回避策を最初に見つけたのは Github ユーザーの CSFFlame 氏である。  

 * [[Issue]: Comfyui Ksampler error(HIP error: an illegal memory access was encountered) · Issue #1795 · ROCm/TheRock](https://github.com/ROCm/TheRock/issues/1795#issuecomment-3519877539)

CWSR (compute wave store and resume) は AMD GPU が実行する Wave、シェーダーの切り替えを可能にするための機能であり、先のブートパラメータはそれを無効化するためのものとなる。  

その後、AMD の Filip Jankovic 氏により、ROCm ライブラリの 1つ、hipBLASLt にメモリアクセスエラーを修正するため "expert scheduling mode" を無効化するパッチが適用された。  

 * [[hipblaslt] Disable expert scheduling mode for gfx12 (#3164) · ROCm/rocm-libraries@29b7468](https://github.com/ROCm/rocm-libraries/commit/29b74681771a7da911faee1e38c1d6d90826023c)
 * [[Issue]: [hipblaslt][gfx12] Conditionally enable Expert Scheduling Mode · Issue #3211 · ROCm/rocm-libraries](https://github.com/ROCm/rocm-libraries/issues/3211)

"expert scheduling mode" について "RDNA4" Instruction Set Architecture: Reference Guide に詳細は書いていないが、hipBlasLt のコード中のコメントから、ハードウェアによるデータ依存性のチェックを無効化し、ソフトウェアによる管理を信頼するモードであることが推察される。  
この "expert scheduling mode" は 2025/04 に有効化されるパッチが公開されており、ROCm 7.0 からリリースに含まれている。  

 >         +    # 0: Normal mode. Hardware applies all of the normal data dependency checks
 >         +    # 1: Full expert mode (not suppoeted yet). Disable hardware checks against: VA_VDST, VA_SDST, VA_SSRC, VA_VCC, VM_VSRC and SA_SDST.
 >         +    # 2: Disable only VA_VDST and VM_VSRC checks.
 >
 > {{< quote >}} [Add ExpertSchedulingMode (#1716) · ROCm/hipBLASLt@b92d043](https://github.com/ROCm/hipBLASLt/commit/b92d043dd5bb9e881967f0ccc5052c84b3dd66ad) {{< /quote >}}

そして hipBLASLt のパッチに関連付けられている、Linux Kernel の amdkfd ドライバーへのパッチを見ると、CWSR の GPU が実行するコードに "expert scheduling mode" のサポートが追加されている。  

 * [[PATCH v3] drm/amdkfd: Trap handler support for expert scheduling mode](https://lists.freedesktop.org/archives/amd-gfx/2025-December/134755.html)

つまり今までの情報を要約すると、ROCm 7.0 から hipBLASLt は "expert scheduling mode" を有効化しているが、amdkfd の CWSR における、Wave を中断し、保存、そして再開する処理においては "expert scheduling mode" をサポートしていなかった、という状況の不一致がメモリアクセスエラーを引き起こしていた可能性が高い。  
PyTorch は内部的に hipBLASLt を使用しているため、主に PyTorch を実行中に発生していたこととも符号する。  
この問題は TheRock での rc リリースを含めるなら 2025/08 から、正式なリリースでは 2025/09 から発生していたことになる。  

今後は amdkfd から取得可能な情報に "expert scheduling mode" のサポートについて追加され、hipBLASLt はそれを用いてサポートを切り替える方式となるらしい。  

 * [Add HasExpertSchedMode device prop by fjankovi · Pull Request #2241 · ROCm/rocm-systems](https://github.com/ROCm/rocm-systems/pull/2241)

"expert scheduling mode" のサポートを amdkfd の CWSR に追加するパッチはまだリリースには取り込まれておらず、執筆時点では手動で適用し、ビルドする必要がある。  
また、執筆時点での最新リリースは 7.1.1、プレビューリリースは 7.10.0 だが hipBLASLt へのパッチはどちらにも含まれていないため、これも自分でビルドするか、TheRock の rc リリース、あるいは次回の正式リリースを待つ必要がある。  

CWSR を無効化した環境 (`amdgpu.cwsr_enable=0`) と、amdkfd にパッチを適用し CWSR を有効化した環境で RX 9060 XT 16GB を用いて検証しているが、今のところメモリアクセスエラーは一度も発生していない。  
まあ偶然発生していないだけで、完全に修正されことを証明し、確信することはできないが。  

ROCm + RDNA 4 GPU を使用する上でしばらくの間悩まされていた問題が解決した (と思われる) ことは喜ばしい。  
しかし、ドキュメント化が充分にされていないハードウェアの機能が強制的に有効化され、環境変数といった有効無効を切り替える手段が存在していなかったことは不便と言わざるを得ない。  
もしもそのような手段存在していれば、早くに問題の原因が判明していたのではないだろうか。  

