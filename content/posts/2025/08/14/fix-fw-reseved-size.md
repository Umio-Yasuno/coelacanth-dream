---
title: "RDNA 4 におけるファームウェア予約領域の問題が修正"
date: 2025-08-14T23:36:40+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "GFX12", "Linux Kernel", "RDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

以前に取り上げた、Linux 環境 + RDNA 4 dGPU において、ファームウェア用の VRAM 予約領域が 516MiB と従来と比べて非常に大きい問題だが、最新の Linux Kernel とファームウェアの組み合わせによる修正が確認された。  

## 詳細
まず問題が報告された [issue](https://gitlab.freedesktop.org/drm/amd/-/issues/4437) だが、AMDGPU ドライバーのメンテナである Alex Deucher 氏により、すでに以下のコミットで修正済みとしてクローズされた。  
RDNA 4 dGPU に使われている PSP (Platform Security Processor) v14 では、特定のコマンドを介して取得できるサイズとアドレスに予約領域を割り当てる方式に変わったらしい。従来の AMD GPU では VBIOS 内にあるサイズの情報を基に割り当てていた。  

 * [drm/amdgpu: reclaim psp fw reservation memory region - kernel/git/torvalds/linux.git - Linux kernel source tree](https://web.git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=a3b7f9c306e19a5b618483c11e8af6404ff69408)

このコミットはまだ Linux Kernel v6.17-rc1 にしか含まれていない。  
安定性や性能に大きく関わるような問題でもないため、v6.15.x と v6.16.x にポートされる可能性は低いだろう。  
また、ファームウェアも最新のものをインストールする必要がある。執筆時点での最新リリースは [20250808](https://gitlab.com/kernel-firmware/linux-firmware/-/commits/20250808?ref_type=tags) だが、自分が試したファームウェアバージョンは含まれていないため、リポジトリをクローンして最新版をインストールする必要があるかもしれない。  

 * <https://gitlab.com/kernel-firmware/linux-firmware/>

VRAM 予約領域のサイズがどれだけ変わったかだが、サイズをログに出力する変更を適用して確認した所、54MiB と 462MiB 減っていた。  

 >         [    5.570972] amdgpu 0000:03:00.0: amdgpu: PSP FW reserve size 50M
 >         [    5.570976] amdgpu 0000:03:00.0: amdgpu: PSP FW reserve size ext 4M
