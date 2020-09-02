---
title: "AMD Navi21 /Sienna Cichlid 情報近況　―― GDDR6と省電力とサーバ向けと 【2020/07/21】"
date: 2020-07-21T21:14:59+09:00
draft: false
tags: [ "Sienna_Cichlid", "Navi21", "RDNA_2" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

{{< pindex >}}

 * [VRAM には GDDR6 を採用か](#sienna-gddr6)
 * [AVFSモジュール と Graphics Power Optimizer](#sienna-avfs-gpo)
   * [Navi10 から倍近い数の AVFSモジュール](#avfs-module-2x-navi10)
   * [GPO](#sienna-gpo)
 * [RAS機能と EEPROM をサポート](#sienna-ras-eeprom)

{{< /pindex >}}

## VRAM には GDDR6 を採用か {#sienna-gddr6}
*Sienna Cichlid* は GPUメモリに、*Navi10/14* 同様 GDDR6メモリを採用すると見られる。  

根拠となるのは以下のパッチであり、  
{{< link >}} [[PATCH 095/207] drm/amdgpu: add vram_info v2_5 in atomfirmware header](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050059.html) {{< /link >}}
VBIOS から情報を読み取るコード[atomfirmware.h](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/include/atomfirmware.h&h=amd-staging-drm-next) に、*Sienna Cichlid* をサポートするため追加された部分には GDDR6メモリ関連のパラメータが記述されている。  

先日、*Sienna Cichlid* は **XGMI** をサポートするという記事を書いたが、コストは掛かるが実装面積は小さく収められる HBM2 ではなく、{{< comple >}} バス幅は不明なものの {{< /comple >}}GDDR6メモリを使うのであれば、1カード上に 2GPUを搭載、マルチGPUとして接続する形よりも、専用ブリッジを用いて 2カードを接続する形のが現実的かもしれない。  
{{< link >}} [Navi21 /Sienna Cichlid は高速なGPU間通信 XGMI をサポート | Coelacanth's Dream](/posts/2020/07/17/navi21-sienna_cichlid-support-xgmi/) {{< /link >}}

## AVFSモジュール と Graphics Power Optimizer {#sienna-avfs-gpo}
### Navi10 から倍近い数の AVFSモジュール {#avfs-module-2x-navi10}
**AVFS** とは、dGPU では *Polaris* 世代から導入された技術であり、シリコン製造における微妙にばらつきが存在する個々の GPU に対して、動作を測定し、それぞれにより優れた電圧と周波数を選択する。  
これにより GPUのある周波数における電圧をより低いものに設定し、消費電力を削減することができる。  

その AVFSモジュールが *Navi10* で 36モジュール[^navi10-avfs]、*Navi14* で 28モジュール[^navi14-avfs] だったのが、*Sienna Cichlid* では 67モジュールとなる[^sienna-avfs]。  

[^navi10-avfs]: [smu11_driver_if_navi10.h\inc\powerplay\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/inc/smu11_driver_if_navi10.h?h=amd-staging-drm-next&id=54c96f8679520b9e15a4960fce03e3b757ef08e3#n979)
[^navi14-avfs]: [smu11_driver_if_navi10.h\inc\powerplay\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/inc/smu11_driver_if_navi10.h?h=amd-staging-drm-next&id=54c96f8679520b9e15a4960fce03e3b757ef08e3#n914)
[^sienna-avfs]: [smu11_driver_if_sienna_cichlid.h\inc\powerplay\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/inc/smu11_driver_if_sienna_cichlid.h?h=amd-staging-drm-next&id=5c23cc5c1c5f436270d301c3081541f1364b240c#n1119)

*Arcturus* は 75モジュールと読める。[^arcturus-avfs]  
モジュール数が GPUの規模を表しているように見えてしまうが、*Navi10* と *Navi14* の数があまり変わらないことから、ある世代の GPU でどれだけ省電力機能に力を入れているか次第なようにも思う。  
同世代であっても、[Navy Flounder](/tags/navy_flounder)用の値が設定されていないため、規模に関しては何とも言えない。  

[^arcturus-avfs]: [smu11_driver_if_arcturus.h\inc\powerplay\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/inc/smu11_driver_if_arcturus.h?h=amd-staging-drm-next&id=f75c8c018d7c3d2e6300d3762ba2e2f8e77eba99#n787)


### GPO {#sienna-gpo}
*Sienna Cichlid* では GPO(Graphics Power Optimizer) と称す新機能が追加されており、機能の中身としては SMU(System Management Unit) が CU数やメモリ状況に応じて、GPUクロックに 16個の V/F(Voltage /Frequency?) ポイントを作成、RLC がデータ転送に要求されるメモリ速度に応じて 16個のポイントから 1個を選択する。[^sienna-gpo]  

[^sienna-gpo]: [[PATCH 135/207] drm/amd/powerplay: enable GPO](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050099.html)

*Navi10* から GPUクロックの DPM(Dynamic Power Management) は 16レベルとなっていたが、それよりも積極的に最適な V/F を計算、適用する機能なのだろうか。[^navi10-gfx-dpm-level]  

[^navi10-gfx-dpm-level]: [smu11_driver_if_navi10.h\inc\powerplay\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/inc/smu11_driver_if_navi10.h?h=amd-staging-drm-next-sienna_cichlid&id=43349491dae7284a195abfa8ff9044f7a20222a2#n33)

上記 AVFSモジュール数と合わせて、*Sienna Cichlid* ではより最適な電圧/クロックを追求し、消費電力の削減に繋げていると思われる。  


## RAS機能と EEPROM をサポート {#sienna-ras-eeprom}
*Sienna Cichlid* は [Vega20](/tags/vega20) と [Arcturus](/tags/tags/arcturus) がサポートする RAS(Reliability, Accessibility, and Serviceability)機能に対応し、不揮発性メモリ EEPROM もサポートする。[^sienna-eeprom]  

[^sienna-eeprom]: [[PATCH] drm/amdgpu: add RAS EEPROM support for sienna chichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051693.html)

RAS機能の中で EEPROM は VRAM の ECCエラー(2-bitエラー?) が発生した不良ページを記録するのに使われる。  
記録された不良ページは R/P/F 3つのフラグで分けられ、F のフラグが立ったものは予約しないように設定される。[^amdgpu-ras]  
これにより、将来的なさらなるメモリエラーを減らすことができる。  
不揮発性メモリである EEPROM を使うため、再起動しても記録された値は残り続ける。

同じような機能に、NVIDIA の [Dynamic Page Retirement](https://docs.nvidia.com/deploy/dynamic-page-retirement/index.html)がある。  

[^amdgpu-ras]: [drm/amdgpu AMDgpu driver — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu.html#amdgpu-ras-support)

それとして、RAS機能と EEPROM を使った不良ページの記録は信頼性を確保したい、GPU を使うサーバ向けの機能であり、それを *Sienna Cichlid* がサポートするというのは、*Sienna Cichlid* をベースとしたサーバ向け製品の登場予測に繋がる。  
*Sienna Cichlid* は MxGPU といった VF(Virtual Function) 機能もサポートしており、記述からそのための Device ID も用意されているものと思われる。[^sienna-vf]  

[^sienna-vf]: [drm/amdkfd: sienna_cichlid virtual function support](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdkfd/kfd_device.c?h=amd-staging-drm-next&id=9110b6c1cbcd98a39134273f321691d7729dd72a)

*Sienna Cichlid* は [RDNA 2](/tags/rdna_2) GPU としてグラフィクス向け機能が強化されるだけでなく、XGMI によるマルチGPUサポート、サーバ向け機能もサポートと、厚い構成のコアを持って広くに活躍する GPU となる *かもしれない* 。あくまで個人的な推測。  

しかし、RDNA系 GPU がサーバ向けもカバーするのであれば、今後 CDNA系 GPU は数値計算等の HPC向けといった感じになるのだろうか。  
自分の推測であるし、クラウドゲーミングやサーバ向けとして出るんじゃないかと推測した *Navi12* も現状 Apple向けの **Radeon Pro 5600M** しか出ていないため、普通に外している可能性もある。  
基本、根拠とソースはすべて示しているので、自分の考えはあくまで一個人のものであり、後は個々人で考えて頂ければ、というのが当サイトのスタンス。  

{{< ref >}}

 * [The Polaris Architecture | - polaris-whitepaper.pdf](https://www.amd.com/system/files/documents/polaris-whitepaper.pdf)
 * [GTC 2015 - GPUはどの程度エラーするのか? | マイナビニュース](https://news.mynavi.jp/article/20150410-gtc2015_gpu_error/)

{{< /ref >}}
