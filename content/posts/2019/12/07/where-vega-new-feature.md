---
title: "Vegaの新機能今いずこ"
date: 2019-12-07T05:13:46+09:00
draft: false
tags: [ "Radeon", "GCN", "GFX9" ]
keywords: [ "Radeon", "Vega" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

色々と新機能をアピールしながら登場したVegaだったが、その新機能はどうなったのか？  
という個人的なまとめ。  

### HBCC（High Bandwidth Cache Controller）
広帯域なHBM2をキャッシュに使い、CPU側のDRAMもVRAMとして効率的に扱う機能。  
スライドでは、HBCCからDRAM以外にNV(Non-volatile) RAMやNetwork Storageにも繋げられるようだったが、一般ではDRAMまでで、Radeon Pro SSGでは2TBのSSDがオンボードに搭載された。  


Vega10/12/20では有効に出来るはずだが、それ以降のArcturus、Naviでは削除されたと思われる。  

まずHBCCのOFF/ONでGFX IDが異なり、Vega10ではOFFがgfx900、ONがgfx901となっているのだが、  
[rocdefs.hpp - RadeonOpenCompute/ROCm-OpenCL-Runtime](https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/blob/master/runtime/device/rocm/rocdefs.hpp#L68)  
そのgfx901でLLVMのレポジトリを検索したところ、無慈悲な言葉が。  
[AMDGPU: Remove remnants of gfx901 (it was deprecated some time ago) - llvm/llvm-project](https://github.com/llvm/llvm-project/commit/1501af4846791c3b52b812c41ec540081343ba38)  

Ravenが***gfx902***、Vega12が***gfx904***、Vega20が***gfx906***であり、HBCC ONのGFX IDがそれに1足した***gxf903/gfx905/gfx907***となる。  
そして、Arcturusが***gfx908***、Raven2/Renoirが***gfx909***となっており、HBCC ON分の空きが存在しない。  
これはNaviも同様で、Navi10が***gfx1010***、Navi12が***gfx1011***、Navi14が***gfx1012***と詰められている。  
GDDR6を搭載するNaviはまだしも、HBM2を使うであろうArcturusでもないのだから、復活は望み薄だ。  

そもそも***gfx905、gfx907***が追加された痕跡がLLVMのリポジトリにないことから、割と早い段階でそれ以上の改良を諦めたのではと勘ぐってしまう。

ゲーム向けの機能としては、使用可能なVRAMを倍以上まで増やしつつ効果的に使えるということで、VRや最新のゲームを高プリセットでやりたいゲーマーから一定の支持はあった、はず。  
Vegaが発売初期はマイニング需要で狩られてゲーマーが買えない状況が続いたり、後継のRadeonVIIが16GBとGPUとしては破格の容量を持っていたため、影は薄くなってしまった。  

コンピュート機能としても、HPC用途においてGPUの比率が高まっていたことから、GPU側でSSD等を扱える機能は、方向が間違っていないはずだったが……
![4GPUs System](/image/2019/11/30/4gpus-system.webp)  

### DSBR（Draw Stream Binning Rasterrizer）
Vega10における改良の目玉であり、レンダリング領域を細かいタイルに分割し、頻繁に使われる部分をL2キャッシュに置くことで効率化するという機能。  
HBM2が広帯域といっても、オンチップなL2キャッシュには速度も電力効率もかなわないため、（Vega64だとメモリ帯域:483.8 GB/s、L2キャッシュ帯域:1.58 TB/s）  
性能向上はもちろん、省電力効果も見込める。  
ゲームによっては、有効時で最大10%のフレームレート向上と33%のメモリ帯域削減が見られたらしい。  
<https://en.wikichip.org/w/images/a/a1/vega-whitepaper.pdf>  

ソフトウェア側では DPBB、DFSM に分けて実装された。  
[[Mesa-dev] [PATCH 2/2] radeonsi: disable primitive binning on Vega10 (v2)](https://lists.freedesktop.org/archives/mesa-dev/2017-October/172054.html)  
DPBBが部分的な、DFSMは全体的なものとなるらしいが、  
決まって速くなる訳ではないため、有効可能ではあるがデフォルトでは無効にされている。  

そしてしばらくしてから、Vega10、Ravenの、指定範囲外の描画を切る scissor 機能にバグが見つかった。  
[ radeonsi/gfx9: add a scissor bug workaround - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/71eca0780a0cd0794545c1fbfdd96fa4f07c2476)  
[ radeonsi/gfx9: limit the scissor bug workaround to Vega10 and Raven only - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/e616743dabe4cdee789c7ad8386fbe9195cbb0ca)  

よく使われる部分を判別しておこなうDSBR機能はこのバグの影響を受け、限定的な実装となった。  
[ radeonsi: limit DPBB context_states_per_bin batches when using gfx9 workaround - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/519bebdb40d9df5926e8b16dedd36b8e0f356f60)  
Vega12/20、Raven2では修正されている。  

<br>
Raven（APU）向けのチューニングがされていることから、メモリ帯域が狭いAPUでは効果があったが、dGPUではそうでもなかったのかもしれない。  
（じゃあ上述したフレームレート最大10%UP、メモリ帯域33%削減は何なんだとなるが、10fpsが11fpsになれば10%の向上だ。下衆の勘繰りであることは否めないが。バグの影響もあったかもしれない）  

それでもGFX10に引き継がれ、改良は進められているらしい。  
[ radeonsi/gfx10: implement primitive binning - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/9f68367d19d9c0394bc935493788dcd189e08f49)  
DPBBのみだが、GFX10ではデフォルトで有効にされる。  
[ radeonsi/gfx10: enable primitive binning by default - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/2b2093961eeb7b9d70573fde33eb8b87d5a6f35f)  

<br>
オープンソースドライバーのRadeonSI、RADVでは確認できるが、Windowsのプロプライエタリなドライバーではどうなのかはわからない。  
#### DPBB（Deferred Primitive Batch Binning）
Raven向け。  
[si_state_binning.c - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/gallium/drivers/radeonsi/si_state_binning.c#L547)  
Ravenではバグがあったが、Raven2、Renoirでは修正されていることから、有効時にそれらで性能や電力効率に差が出ると考えているが、検証した人はいないし、自分も実物を持ってないため不明。  
また、差が出てもそこまで気にするような程ではないかもしれない。  

#### DFSM（Deterministic Finite State Machine）
RADVで2-3%向上するからRavenではデフォルトで有効、と思いきや3%下がるゲームがあったので撤回、とそううまくは行かないようだ。  
[ radv: Enable binning and dfsm by default on Raven. - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/17b5a59b4ee3adb9c99f3d850eb4a561196c69a0)  
[ radv: Disable dfsm by default even on Raven.  - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/0fa2740059a05e47854240ff8a6782d879389525)  

### Rapid Packed Math
INT16/FP16演算において、32レーンのVectorALUでFP16x2とまとめて処理することでスループットを2倍にする機能。  
INT8にも対応しているが、そちらはQSAD/MQSAD命令（動き検出等に使われる）のみ。  
FP16、FP32の混合演算にも対応。  

これはVegaの新機能というよりNCUの機能なため、Vega以降も引き継がれている。  
Vega20ではINT4のPacked、INT4/8、FP16/32/64の混合演算にも対応した。  

それとNaviでは呼び方がNCUではなくCUと戻った。  
もう Next-Generation ではないから？  

### NGG（Next Generation Geometory） / Primitive Shader
解説に関しては、こちらの4Gamer.netの記事を見るのが早い。  
[西川善司の3DGE：新設の「プリミティブシェーダ」を搭載し，Radeon RX Vegaはどこへ行く？](https://www.4gamer.net/games/337/G033714/20170804085/)  

Vega（GFX9）では廃止、諦め、GFX10での実装に注力することとなった。  
[Making a GDS Allocation for NGG](https://lists.freedesktop.org/archives/amd-gfx/2018-August/025320.html)  

DFSMと相性が良いはずで、廃止になってしまったのは残念。  
DFSMの効果があまりなかったのに関係しているかもしれない。  

具体的に何が原因で廃止になったかは不明。  
GFX10ではGFX9から設計に変更があったらしいが、GFX9でのNGGに何らかの問題があって変更したのか、  
GFX10で変更することが決まったからGFX9では廃止したのか。  
[[PATCH] drm/amdgpu: remove gfx9 NGG](https://lists.freedesktop.org/archives/amd-gfx/2019-September/040258.html)  

[LLPC](https://github.com/GPUOpen-Drivers/llpc)（LLVM-Based Pipeline）を見る限り、GFX10でしっかりと進められている様子。  

[radeonsi/gfx10: document NGG shader stages - Mesa GitLab](https://gitlab.freedesktop.org/mesa/mesa/commit/226f650d9222a191130ee673d2cb4405da972c4a)  

### 感想
新機能にバグは付き物……  
Vegaが久しぶりのハイエンドとして登場したために、残念なところが際立ってしまったように思う。  
だがHBCC以外GFX10に引き継がれ、改良されているため決して無駄ではなかった。  

<hr>
参考 :   
[深掘り！「AMD Next Horizon」 - Vega 7nm Deep Dive。7nmのVegaはコンシューマに来ない？ - マイナビニュース](https://news.mynavi.jp/article/20181227-20181227-amd_next_horizon)  
[Hot Chips 29 - AMDの新フラグシップGPU「Vega 10」を読み解く - マイナビニュース](https://news.mynavi.jp/article/20170830-hotchips29_vega/)  
[西川善司の3DGE：AMD，次世代GPU「Vega」における4つの技術ポイントを公開。HBM2はキャッシュで使う!? - 4Gamer.net](https://www.4gamer.net/games/337/G033714/20170101002/)  
[西川善司の3DGE：新設の「プリミティブシェーダ」を搭載し，Radeon RX Vegaはどこへ行く？ - 4Gamer.net](https://www.4gamer.net/games/337/G033714/20170804085/)  
[AMDが次世代GPU「Radeon RX Vega64」を正式発表 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1073276.html)  
