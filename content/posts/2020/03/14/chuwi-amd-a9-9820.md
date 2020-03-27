---
title: "CHUWIよりAMD A9-9820搭載MiniPC発売予定 & 推測"
date: 2020-03-14T23:50:38+09:00
draft: false
# tags: [ "", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

更新休止前、最後の記事。昼頃からはほんとに休止。  

CHUWIはMiniPC [AeroBox](https://www.chuwi.com/product/items/Chuwi-AeroBox.html)が近日Amazonにて発売予定であることを発表した。  
CPUにAMD A9-9820、GPUにRadeon R7 350を搭載し、RAMにはDDR3 8GB、ストレージにはSSD 256GBを標準で搭載し、またDDR3スロットを4基備えるため増設も可能となっている。  
A9-9820は8-Core/8-Threadを最大2.35GHzで駆動し、日常的な軽作業はもちろん、重い作業もスムーズに処理できるとする。  
[8コアCPU搭載！CHUWIオフィス向けミニPC「AeroBox」近日発売 - Chuwi](https://www.chuwi.com/jp/news/items/36.html)  

## A9-9820推測
気になるのは A9-9820 の正体。その推測がこの記事のメインとなる。  
R7 350は既存のGPUであるため、特に謎はない。[^1][^2]  

[^1]: <https://gitlab.freedesktop.org/mesa/drm/blob/master/data/amdgpu.ids#L22>
[^2]: <https://github.com/GPUOpen-Tools/common-src-DeviceInfo/blob/master/DeviceInfoUtils.cpp#L73>

まずA9-9820の名は以前1度出たことがあった。  
[AMD 'Cato' RX-8125, RX-8120 and A9-9820 Reveal Themselves in 3DMark database - Guru3D](https://www.guru3d.com/news-story/amd-cato-rx-8125rx-8120-and-a9-9820-reveal-themselves-in-3dmark-database.html)  
同時にRX-8120、RX-8125の存在も確認されており、中身としては3つとも同じだろう。  
AeroBoxでは外部GPUにRadeon R7 350を搭載しているが、組み込み向けのR-Series (RX8120, RX8125) があることから実際はiGPUも搭載されているはずだ。  
上記URL先のスクリーンショットを見ても、GPU名が *Cato SoC* と認識されている。  

そしてDDR3メモリということから、CPUは少なくともZenアーキテクチャではなく、Jaguar系/Bulldozer系アーキテクチャだと考えられる。  
しかしコンシューマ向けの製品で、CPUにJaguar/Bulldozer系を8コアを搭載したAPUは無い。  
どこかのカスタムチップの転用か、新たに起こしたチップということになる。  

### ゲーム機向けのチップ転用である可能性
ネット上では少しそういった声を聞く。  
しかし比較的最近の *PS4 /Pro* 、*Xbox One X* のメインチップはJaguar系のCPUを8コア搭載してはいるが、単純にDDR3のメモリコントローラが無い。  
*Xbox One* であれば備えているが、TSMC 28nm HPプロセスでJaguar系CPUが2.35GHzが可能なのか、ESRAM 32MBも搭載しているチップを組み込み向けに転用できるのか、といった疑問が生じる。  
前者はゲーム機ではチップを多く確保するためクロックや一部ユニットにマージン、冗長性を設けることから、実際には可能かもしれない。  
しかし後者に関しては、待機時の消費電力が大きいSRAMが多いと、消費電力的に組み込み向けには必然的に向かなくなる。  

そのためゲーム機向けチップの転用ではないと考える。  

### Excavatorアーキテクチャの新規チップか
AMD Embedded R-Series は G-Series よりも高性能という立ち位置にあり、CPUにBulldozer系を採用してきた。  
[Embedded R Series Processors | High Performance CPU | AMD](https://www.amd.com/en/products/embedded-r-series)  
そも Ax-9xxx のモデルナンバーは *Bristol Ridge (CPU Excavator + GPU Stoney)* に使われている。  

そういうことで、あまり面白みのない結論となったが、個人的にはExcavator系の新APUだと推測する。  
違う可能性も当然あるため、誰かが実際に購入し、検証するのを期待している。  
