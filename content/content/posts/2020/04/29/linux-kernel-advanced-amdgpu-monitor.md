---
title: "スパコンに向けて強化される Linux Kernel の AMDGPU 関連機能"
date: 2020-04-29T22:16:41+09:00
draft: false
tags: [ "Linux_Kernel" ]
keywords: [ "", ]
categories: [ "Software", "AMD", "GPU"]
noindex: false
---

1.5 Exaflopsの演算性能を持ち、2021年納入予定の *Frontier* 、  
2 Exaflopsの演算性能を持ち、2022年か2023年早期に納入予定の *El Captitan* どちらも AMD CPU + AMD GPU のノード構成になることが明らかにされているが、  
それらスパコンに向けたソフトウェア側の機能が Linux Kernel に組み込まれつつある。  

## インデックス

 * [RAS機能への対応](#ras)
 * [プロセスごとのVRAM使用量が取得可能に](#vram-per-process)
 * [FRUチップ](#fru-chip)
 * [Streaming Performance Monitor](#spm)

## RAS機能への対応 {#ras}
RASは *Reliability, Accessibility, and Serviceability* の略。  
これは *Vega20* の頃から対応が進められており、各ユニット{{< comple >}}UMC、SDMA、GFX{{< /comple >}}へのエラー注入、そこからのGPUリセット機能(BACO)による復帰テストが行なえる。[^5]  
BACO は *Bus Active, Chip Off* の略。  

[^5]: <https://www.kernel.org/doc/html/latest/gpu/amdgpu.html#amdgpu-ras-support>
[^6]: [drm/amdgpu: add supports_baco callback for NV asics. · torvalds/linux@ac74261](https://github.com/torvalds/linux/commit/ac7426169e7bcbbf270fec48301286e5ccae08bc)

## プロセスごとのVRAM使用量が取得可能に {#vram-per-process}
{{< link >}}[[PATCH] drm/amdkfd: Track GPU memory utilization per process](https://lists.freedesktop.org/archives/amd-gfx/2020-April/048643.html){{< /link >}}

AMDは現状、`rocm-smi` というツールを提供しており、それによってVRAMの使用量を確認できるが、VRAM全体であり、どのプロセスがVRAMを多く消費しているか、といったものを知ることはできない。  
だが上記パッチにより、それが可能になるはずだ。  

/proc/\<pid\>/ 下に vram\_\<gpuid\> というファイルを生成するようになり、GPUごとに割り振られるIDをファイル名に使うため複数GPUの環境にも対応されている。  
この機能はアプリケーションの最適化にも役立つだろう。  

この機能を、NVIDIAは以前から `nvidia-smi` [^1]で提供しており、ツール面でようやくAMDが追いついてきたと言える。ROCmの開発、スーパーコンピュータ採用等で必要に迫られたのかもしれない。  
一応、AMDは[umr](https://gitlab.freedesktop.org/tomstdenis/umr/-/tree/master)や[Radeon GPU Analuzer](https://github.com/GPUOpen-Tools/radeon_gpu_analyzer)、[Radeon™ GPU Profiler](https://github.com/GPUOpen-Tools/radeon_gpu_profiler)等、レジスタレベル、パイプラインのモニタツールは豊富に提供している。  

[^1]: [NVIDIA System Management Interface | NVIDIA Developer](http://developer.download.nvidia.com/compute/DCGM/docs/nvidia-smi-367.38.pdf)

また少し前までは、AMDGPUの場合、上記 `rocm-smi` や [radeontop](https://github.com/clbr/radeontop) 等のソフトウェアを用いなければ全体でもVRAM容量、使用量を知ることが難しかったが、  
現在はそれら情報が sysfs へ出力されるようになっているため、VRAM使用量なんかは

    $ cat /sys/class/drm/card0/device/mem_info_vram_used 

を実行するだけで知ることができる。  

ただこの機能、既に組み込まれているがドキュメント類は 2020/04/29 現在、未だにアップデートされていない。[^4]  
自分のような一般ユーザにも恩恵があるため、広く周知されてほしい。  
{{< link >}}[私的conkyrc | Coelacanth's Dream](/posts/2020/03/12/private-conkyrc/){{< /link >}}

[^4]: [linux/amdgpu.rst at v5.7-rc3 · torvalds/linux](https://github.com/torvalds/linux/blob/v5.7-rc3/Documentation/gpu/amdgpu.rst)

## FRUチップ {#fru-chip}
製品名、製品ナンバー、シリアルナンバーを記録した *FRUチップ* を読み取るためのパッチが 2020/03/19 より投稿されている。  
{{< link >}}[Enable reading FRU chip via I2C v3 - Patchwork](https://patchwork.freedesktop.org/patch/358146/){{< /link >}}
この *FRUチップ* によってソフトウェアから問題が発生したハードウェアを特定しやすくなり、大規模なコンピュータにおける障害対応、部品交換を手助けする。  

*FRUチップ* は *Vega20* の一部SKUにも搭載されているがサーバ向けのみとなっており、ゲーミング向けSKUは *FRUチップ* を搭載していない。実際コード中でもゲーミング向け製品 **Radeon VII** の DeviceID:"0x66AF"、Apple専用製品 **Radeon Pro Vega II (Duo)** の DeviceID:"0x66A3" は判定部に含まれていない。[^3]  
*Arcturus* はそういった DeviceIDでの判定はせず、すべてが *FRUチップ* を搭載するとし、また前のパッチリビジョンでは *Arcturus* にゲーミング向けSKUは存在しないとしている。[^2]  
やはり、よく言われているように *Arcturus* はサーバ向けSKUのみとなるだろう。  

[^2]:[drm/amdgpu: Re-enable FRU check for most models v3 - Patchwork](https://patchwork.freedesktop.org/patch/360005/)
[^3]:[drm/amdgpu: Re-enable FRU check for most models v4 - Patchwork](https://patchwork.freedesktop.org/patch/360462/)

## Streaming Performance Monitor {#spm}
先日パッチが投稿されたばかりの機能。  
{{< link >}}[[PATCH] drm/amd: add Streaming Performance Monitor feature](https://lists.freedesktop.org/archives/amd-gfx/2020-April/049049.html){{< /link >}}
中身としては、内部バスの利用状況、GPU内部のデータの流れを監視するためのものと読める。  

しかし、後のパッチレビューにて様々な問題点が指摘されており、仮に実装するとしても時間が掛かりそうだ。[^7]  

[^7]: [[PATCH] drm/amd: add Streaming Performance Monitor feature](https://lists.freedesktop.org/archives/amd-gfx/2020-April/049061.html)

{{< ref >}}

 * [AMD Powering the Exascale Era with El Capitan | AMD](https://www.amd.com/en/products/exascale-era)

{{< /ref >}}
