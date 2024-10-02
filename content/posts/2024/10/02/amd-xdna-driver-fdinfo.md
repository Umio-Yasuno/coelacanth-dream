---
title: "AMD XDNA ドライバーからプロセスごとの使用量が取得可能に"
date: 2024-10-02T16:20:37+09:00
draft: false
categories: [ "AMD", "APU", "NPU", ]
tags: [ "Linux_Kernel", "Phoenix" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD がオープンソースで開発、公開している [AMD XDNA Driver for Linux](https://github.com/amd/xdna-driver) が `fdinfo` へのメモリ使用量と NPU 使用量の出力に対応した。  
`fdinfo` への使用量の出力には `amdgpu, i915, xe` 等の GPU ドライバーが対応しており、それを利用するモニタリングソフトウェアとしては [Syllo/nvtop](https://github.com/Syllo/nvtop) が存在する。  

 * [support drm usage stats by amd-xiaomren · Pull Request #262 · amd/xdna-driver](https://github.com/amd/xdna-driver/pull/262/files)
 * [DRM client usage stats — The Linux Kernel documentation](https://docs.kernel.org/gpu/drm-usage-stats.html)

Windows では Ryzen 8040 Series 向けに NPU モニタリングのサポートがタスクマネージャーに追加されることが 2024/02/08 に発表されていた。[^for-windows]  
今回 AMD XDNA Driver が `fdinfo` への出力に対応したことで、Linux 環境においても同様の機能が追加可能になったと言える。  

[^for-windows]: [Upcoming Windows Task Manager Update Will Add NPU Monitoring For Ryzen 8040 Series Processors](https://community.amd.com/t5/ai/upcoming-windows-task-manager-update-will-add-npu-monitoring-for/ba-p/664382)

自作の AMD GPU 向けモニタリングソフトウェア `amdgpu_top` にも最低限の対応を追加したが、XDNA NPU を搭載した環境を持っていないため、ドライバーのソースコードやドキュメントを参照しながらの作業となった。  

Linux 環境における XDNA NPU のモニタリング機能に関しては、Ryzen AI 300 Series (Strix Point) 限定ではあるが他にも方法はある。  
`amdgpu` ドライバーの機能として、SMU (System Management Unit) から内部的に取得可能な温度やクロック、電力量、使用率、帯域等を構造体にまとめ、バイナリ形式で出力する `gpu_metrics` というものがある。  
Ryzen AI 300 Series では `gpu_metrics` に `average_ipu_activity, average_ipu_reads, average_ipu_writes, average_ipu_power, average_ipuclk_frequency, average_mpipu_frequency` というフィールドが存在し、それらは恐らく XDNA NPU の状態に関する情報と思われる。  

