---
title: "Linux 環境では AMD dGPU のデフォルトプロファイルが 3D_FULL_SCREEN に"
date: 2024-12-29T23:42:08+09:00
draft: false
categories: [ "Software", "AMD", "GPU" ]
tags: [ "Linux_Kernel", "RDNA_2", "RDNA_3" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD GPU は複数の電力プロファイル (Power Profile) をサポートしており、プロファイルによって負荷に応じて変動するクロックのバイアス (ドキュメント等ではヒューリスティック) が変わってくる。  
これまで AMDGPU ドライバーではデフォルトのプロファイルを `BOOTUP_DEFAULT` としていたのだが、Linux Kernel v6.11.7/v6.12 から取り込まれた変更によって、一部を除き dGPU では `3D_FULL_SCREEN` がデフォルトとなった。  

 * <https://www.kernel.org/doc/html/latest/gpu/amdgpu/thermal.html#pp-power-profile-mode>
 * [[PATCH 3/4] drm/amdgpu/swsmu: default to fullscreen 3D profile for dGPUs](https://lists.freedesktop.org/archives/amd-gfx/2024-October/115121.html)

`BOOTUP_DEFAULT` と `3D_FULL_SCREEN` の違いを説明すると、厳密には GPU によって設定は異なるが、概ね `3D_FULL_SCREEN` の方が低い負荷でも高クロックを維持するような設定になっている。  

`3D_FULL_SCREEN` のデフォルト化によって、低負荷、中程度の負荷が掛かるケースでの消費電力が若干増える可能性はある。ただ自分で試した限り、ブラウジング等では `BOOTUP_DEFAULT` の時と特に変わらない。  
また、プロファイルに関するビットマスクの変更によって `3D_FULL_SCREEN` がデフォルトになっているため、従来のように `BOOTUP_DEFAULT` を選択することはできなくなっている。  

SMU v13.0.7、GPU コードネームで言えば *Navi33/GC 11.0.1* では `3D_FULL_SCREEN` よりも `BOOTUP_DEFAULT` の方が動作クロックが高くなりやすい設定になっているらしく、*Navi33/GC 11.0.1* に限って `BOOTUP_DEFAULT` がデフォルトになっている。  

 * [[PATCH] drm/amd/pm: set the default workload type to bootup type on smu v13.0.7](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117552.html)
 * [[PATCH v2] drm/amd/pm: Set SMU v13.0.7 default workload type](https://lists.freedesktop.org/archives/amd-gfx/2024-December/117562.html)

## 3D_FULL_SCREEN がデフォルトになった経緯 {#issue}

この変更が取り込まれた経緯としては、まずフレームレート制限機能のあるゲームにおけるスタッタリングの発生の報告がある。  
スタッタリングは主に *RDNA 2 アーキテクチャ* とそれ以降の世代の dGPU で発生する。  
AMD の Alex Deucher 氏によれば、GPU の処理パイプラインを見た時、フレームレート制限機能はパイプラインの途中から GPU の負荷が低くなる期間を作り出す。一種のパイプラインバブル、パイプラインストールを作るとも言える。  
描画を完了するまでの時間が短いほど、負荷が低い期間は長くなる。  
プロファイルに応じて、GPU 内部の管理ユニット (SMU, System Management Unit) はその間の動作クロックを下げることで消費電力を最適化するが、次に描画更新する際のクロックが低いため処理時間が長くなり、場合によっては FPS が瞬間的に低下する。  
描画更新に必要な処理時間が長くなったことで、SMU は動作クロックを引き上げ、そして次の FPS は安定する。処理時間が短くなったことでパイプラインバブルが発生し、SMU は動作クロックを下げる……この繰り返しによってスタッタリングが発生する。  
この問題の軽減策として `3D_FULL_SCREEN` が有用だったため、dGPU では `3D_FULL_SCREEN` をデフォルトとすることになった。  

 * <https://gitlab.freedesktop.org/drm/amd/-/issues/1500#note_1487319>

*RDNA 2 アーキテクチャ* でこの問題が顕在化した理由としては、最大動作クロックの向上が考えられる。  
*RDNA 1 アーキテクチャ* 世代の dGPU では、ブーストクロックは大体 1600-2000MHz の範囲、ゲームクロックは 1400-1850MHz となっていたが、*RDNA 2* 世代の dGPU ではブーストクロックで 2100-2800MHz、ゲームクロックでは 1800-2650MHz となっている。  
世代が進むごとに最大動作クロックは大きく向上し、消費電力の最適化機能によるクロックの変動範囲も大きくなった。  

あくまでも軽減策であり、プロファイルに `3D_FULL_SCREEN` を設定していても引き続きこの問題は発生し得る。  
ゲーム側のグラフィック設定を引き上げる、高リフレッシュレートのモニタを使用するといった環境側でも軽減策を模索する必要はあるだろう。  
しかし、古いゲームだと最高設定でも最近の GPU からしたら負荷が低いことや、最大フレームレート設定が 30fps か 60fps しか選べないことも考えられる。  
その場合は GPU の動作クロックの固定、消費電力の最適化の無効、VSync (垂直同期) の無効といった方法を取る必要があるかもしれない。  

