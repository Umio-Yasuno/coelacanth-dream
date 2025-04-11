---
title: "AMDGPU ドライバーのデフォルトプロファイルが BOOTUP_DEFAULT に戻り、動的なプロファイル切替が有効に"
date: 2025-04-11T01:26:31+09:00
draft: false
categories: [ "AMD", "GPU", "Software" ]
tags: [ "Linux_Kernel", "RDNA", "RDNA_2", "RDNA_3", "RDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Linux Kernel における AMDGPU ドライバーでは、負荷の低いゲームで報告されていたスタッタリングの問題への対策として、Linux Kernel v6.13 から dGPU では `3D_FULL_SCREEN` プロファイルをデフォルトで有効にしていた。  
それが Linux Kernel v6.15 からは `BOOTUP_DEFAULT` プロファイルがデフォルトに戻り、`3D_FULL_SCREEN` プロファイルを動的に切り替えるようになった。  

 * [Linux 環境では AMD dGPU のデフォルトプロファイルが 3D_FULL_SCREEN に | Coelacanth's Dream](/posts/2024/12/29/amdgpu-power-profile/)

以前から AMDGPU ドライバーでは、メディアエンジンである VCN へのコマンドが送信された時に `VIDEO` プロファイルを有効にし、コマンドが完了した時に無効化する切り替え機能が実装されていた。  
今回の変更はそれを GFX/Compute へのコマンドにも適用し、切り替えるプロファイルを `3D_FULL_SCREEN` にしたものとなる。  
GFX 部を持たない *CDNA 系アーキテクチャ* では代わりに `COMPUTE` プロファイルを切り替えるようになっている。  

 * [[PATCH 1/5] drm/amdgpu/gfx: add ring helpers for setting workload profile](https://lists.freedesktop.org/archives/amd-gfx/2025-January/118546.html)
 * [[PATCH 5/5] drm/amdgpu/swsmu: set workload profile to bootup default](https://lists.freedesktop.org/archives/amd-gfx/2025-January/118549.html)

ただ、デスクトップ環境のコンポジタ等も GPU コンテキストを持つと思われるため、一般的な使用状況で `3D_FULL_SCREEN` プロファイルが無効化されることはないような気がする。  
AMDGPU の電力プロファイルは内部的にビットマスクで管理されており、複数のプロファイルのビットを有効可能となっている。  
そのため今回のパッチが適用されている環境では、通常 `BOOTUP_DEFAULT, 3D_FULL_SCREEN` プロファイルが有効化され、メディアエンジンを使用している状況ではそこに加えて `VIDEO` プロファイルが有効化されるという動作になると思われる。  

自分が見た限り、動的なプロファイル切り替えは無効できないようになっている。  
そのため、従来のように `POWERSAVING` や `COMPUTE` プロファイルのみを有効するといったことは不可能だと思われる。dGPU の映像出力を使用しておらず、D3hot/D3cold ステートに移行可能であればその限りではないかもしれないが。  
AMDGPU ドライバーでは `pp_power_profile_mode` sysfs を経由してユーザーが電力プロファイルを選択可能になっているが、これは 1つのプロファイルしか指定できない。  
例えば `pp_power_profile_mode` で `POWERSAVING` プロファイルを指定した場合、一般的なデスクトップ環境の使用では `POWERSAVING` と `3D_FULL_SCREEN` プロファイルが同時に有効化される。  
各電力プロファイルに応じた動作は GPU によって異なり、そこに複数のプロファイルが有効化されるとなると、動作クロックがどのように変化するか予想するのは難しい。  

また、ユーザーは `pp_power_profile_mode` で指定されているプロファイルのみが確認可能であり、動的なプロファイル切り替えによって実際に有効化されているプロファイルを確認する方法は無い。  
今回の変更は、デフォルトの状態で使用するユーザーにとっては有益な面が多いと思われるが、  
省電力あるいは最適化を目的としてプロファイルを切り替えていたユーザーにとっては複雑化したと感じられるかもしれない。  
