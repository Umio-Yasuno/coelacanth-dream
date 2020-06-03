---
title: "Navi14のちょっとしたまとめ"
date: 2019-12-06T07:08:54+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi14", "GFX10", "gfx1012" ]
keywords: [ "Radeon", "Navi14" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

別タイトル「時間稼ぎ」  
Navi14のあまり大したことのない、新鮮でもない情報まとめ。  

### Navi14 スペック
AMDGPUのハードウェア抽象化に使われるライブラリ、PAL（Platform Abstraction Library）にNavi14の情報が追加された。  
<https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L950>  

PALにはハードウェアの詳細なスペックが記述されており、私的にはとてもありがたい。  
具体的には、ShaderEngine（以後SE）数、SEあたりのShaderArray（以後SA）数、SAあたりのRenderBackend数、SAあたりのCU数、そしてL2キャッシュのブロック数（ブロックあたり256KB）がある。  

そこにNavi14が追加されたが、特に目新しい情報はなく、以前画像から推測したスペックが確定したくらい。  
[Navi14 Matome](/posts/2019/11/04/navi14-matome/)  
（見直したところ、今頃表内のSP数が間違っていることに気付いたため修正した。申し訳ない……）  

### ROPの犠牲なくメモリバス幅削減、L2キャッシュ一部無効
少し前の[The amd-gfx Archives](https://lists.freedesktop.org/archives/amd-gfx/)を漁ったところ、GFX10（Navi1x）向けのあるパッチを見つけた。  
[[PATCH] drm/amdgpu: return tcc_disabled_mask to userspace](https://lists.freedesktop.org/archives/amd-gfx/2019-September/040619.html)  
UMD（User Mode Driver）にてL2キャッシュを一部を無効可能にするためのものと思われる。  
パッチレビューでも突っ込まれているが、L2キャッシュにはメモリコントローラーが接続され、メモリアドレスを保存するため、  
普通だったら一部でも無効にしたりはしない。  

だが普通ではないパターンがあり、それはメモリバス幅を削減した下位モデルだ。  

#### 解説
NVIDIAは以前、こっそりとGTX1050の3GBモデルをリリースした。  
GTX1050に使われるGP107はメモリバス幅を128-bitまでしか持たないため、3GBという仕様を満たすには96-bitに減らし、  
GDDR5 8Gbit x3という構成を取る他ない。  
そしてメモリバス幅削減はメモリコントローラーの一部無効化を意味し、繋がれたL2キャッシュも自然と無効化される。  
これだけで済めばいいのだが、L2キャッシュにはROP（Render Output Unit）が繋がれているため、ROPも一部無効にされてしまう。  
ROPが少なくなるというのは、グラフィック処理におけるスループットが下がることを意味するため、GPUとしては中々に痛い。  

それを見越してか、NVIDIAはGTX1050 3GBのCUDAコア数をGTX1050Tiと同数にし、クロックはGTX1050Tiよりも引き上げたが、  
それでも実際の性能ではGTX1050Ti 4GBに劣る結果となった。  
[Nvidia GeForce GTX 1050 3GB Review: (Mostly) Faster Than 1050 2GB - Tom's Hardware](https://www.tomshardware.com/reviews/geforce-gtx-1050-3gb-benchmarks,5654-3.html)  

こうした問題があるからNVIDIAはこっそりとリリースしたのだろうし、NVIDIAもAMDも基本CUDAコア、CUを削減する方向で同一ダイにおけるモデルの差別化をおこなってきたと思われる。  

（追記:2020/01/02）RX 640、RX 630ではメモリバス幅が64-bitでありながら16ROP構成だが、これは恐らくそれぞれのメモリチップを16-bitで接続していると思われる。  
その証拠に64-bit幅なのに最大メモリサイズが4GBとなっている。GDDR5では8Gbit（1GB）チップまでで、16Gbit（2GB）チップの製品はないはず。  
GDDR5では、16-bitでのメモリコントローラー、メモリチップ間接続がサポートされており、主にメモリバス幅そのままでメモリ容量を倍にしたい時にこの機能が使われる。  
メモリ容量を増やすことなく16-bit接続することのメリットは、製品としてそこまでのメモリ帯域は必要はないが容量は欲しい際、消費電力（熱）を減らすことができることだと思われる。  
（追記終了）  

<br>
だがRDNAではROPが増設されたL1キャッシュに接続されるようになり、メモリバス幅を削減（= L2キャッシュの一部無効）してもROPを犠牲にする必要がなくなった。  
これならグラフィック性能の低下を抑えられる。  
上記パッチはそれをドライバに伝えるためのものだろう。  

実際、AMDはNavi14のバス幅を128-bitから96-bitに削減したモデル、RX 5300Mのスペックを公表しているが、ROPは32と他のNavi14を使ったモデルと変わりない。  
[AMD Radeon™ RX 5300M](https://www.amd.com/en/product/8976)  

これはNavi10でもメモリバス幅を削減したモデルの可能性を示唆する。  
***もし***、やるとするならば128-bitと256-bitの中間、192-bitあたりが妥当だろうか。  
