---
title: "GDS と GWS を持たない AMD RDNA 4 GPU"
date: 2025-01-26T06:41:13+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "RDNA_4", "GFX12" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD GCN アーキテクチャから存在していた、すべての CU (Compute Unit) からアクセス可能なスクラッチメモリ GDS (Global Data Share) と、異なる CU や SE (Shader Engine) で実行されているスレッドグループ、Wave と同期するための機能 GWS (Global Wave Sync) だが、  
RDNA 4 アーキテクチャ ではその両方が削除される。  

 * [[AMDGPU] Remove GDS and GWS for GFX12 by jayfoad · Pull Request #76148 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/76148)

GDS の変更自体は最近の世代で実施されており、CDNA 系アーキテクチャでは、従来のアーキテクチャでは 64KiB だった GDS を *CDNA 1/Arcturus/MI100/gfx908* を 4KiB に、そして *CDNA 2/Aldebaran/MI200/gfx90a* では削除している。  
RDNA 系アーキテクチャでは、*RDNA 1/2* は 64KiB だが、*RDNA 3/3.5* で 4KiB にサイズを減少させている。  
GDS の削除は CDNA 系アーキテクチャの方が先行していたが、GWS の削除は RDNA 4 アーキテクチャが初となる。  

GDS の用途としては、計算カーネルやリダクション操作に関わる重要な制御データを格納するためのソフトウェアキャッシュが想定されている。  
GDS/GWS を使うための API は ROCm/HIP では公開されておらず、計算カーネルで使いたい場合はインラインアセンブリを書く必要があった。  
ただ、GDS/GWS は AMD GPU 固有の機能であり、計算カーネルで使うと移植性が下がるという問題がある。  

 * <https://rocm.docs.amd.com/projects/HIP/en/latest/tutorial/reduction.html#global-data-share>

RADV ドライバーでは一部のクエリで GDS を使用していたが、Samuel Pitoiset 氏により対応が行われている。  
また、一部のアトミック操作に関しては `GLOBAL_ATOMIC_*` 命令に置き換えられている。  

 * [radv: implement emulated queries with global atomics on GFX12 (!33041) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33041)

GDS を削除した理由については触れられていないが、*CDNA 3/MI300* のような単体でも GPU として動作するチップレットを複数組み合わせた場合に、GDS のサイズが変わることや、ある GPU チップレットの CU から見て近い GDS と遠い GDS が存在してしまうこと等、複雑化や実装のコストが増えることを避けた結果かもしれない。  

{{< ref >}}
 * [AMD GPU architecture programming documentation - AMD GPUOpen](https://gpuopen.com/amd-gpu-architecture-programming-documentation/)
{{< /ref >}}
