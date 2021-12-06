---
title: "RadeonSIドライバー + RDNA 2 では NGGカリング/プリミティブシェーダー がデフォルトで有効に"
date: 2020-10-17T11:34:57+09:00
draft: false
tags: [ "RDNA_2", "RadeonSI", "GFX10", "NGG" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

AMD GPU のオープンソースドライバー **RadeonSI (OpenGL)** に、*RDNA 2 /GFX10.3* 世代の dGPU にて NGGカリングをデフォルトで有効にするパッチが投稿、メインラインに組み込まれた。  
{{< link >}} [radeonsi: enable NGG culling by default on gfx10.3 dGPUs (7648060d) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/7648060dc03775979e3fa8904c4948c084e82b6a) {{< /link >}}
他にも性能最適化のため、LDS のバンクコンフリクト (同一バンクにアクセスが集中してしまうこと) を解消するパッチも組み込まれている。  

また、*Navi1x* ベースの **Radeon Pro** カードでもデフォルトで NGGカリングが有効化されるようになった。[^navi1x-pro-nggc]  
Proカードのみの理由は恐らく、非同期コンピュートでのカリング処理が実装された時もそうだったが[^pd-cs]、ゲーム等ではまだ効果が小さく、**Radeon Pro** がターゲットとするワークステーション用途におけるソフトウェアでは効果が大きいからと思われる。  

[^navi1x-pro-nggc]: [radeonsi: enable NGG culling by default on Navi1x PRO cards (57d31786) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/57d317865e7bee02a17efcde8beeb6a220f900f1)
[^pd-cs]: [[Mesa-dev] [PATCH 00/26] RadeonSI: Primitive culling with async compute](https://lists.freedesktop.org/archives/mesa-dev/2019-February/215085.html) <br> [RadeonSI Picks Up Primitive Culling With Async Compute For Performance Wins - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=RadeonSI-Prim-Culling-Async-Com)

## NGGカリング / プリミティブシェーダー {#nggc}

[以前](/posts/2020/10/04/aco-ngg-gfx10/) NGG(Next Genaration Geometry) の機能を紹介した時に、ハードウェア側のシェーダーステージ統合とそれによる効率化が NGG の全てではないと断ったが、その時触れなかった NGG の目玉機能がこの NGGカリングである。  

NGGカリングは、{{< comple >}} 結局その世代で有効化されることは無かったが {{< /comple>}} *Vega /GFX9* の新機能として実装され、PS5 でも採用が発表された プリミティブシェーダー/Primitive Shader の名でも呼べるだろう。  
[GPUOpen Drivers](https://github.com/GPUOpen-Drivers) を構成するソフトウェアにはそのように記述されている。  

 >       "Description": "Enable NGG mode, use an implicit primitive shader on a per-pipeline type basis. Use this instead of PAL setting, NggEnableMode.",
 >
 > {{< quote >}} [xgl/settings_xgl.json at c6c90450fb1abf5f414acf1e38a0f51a72c426c1 · GPUOpen-Drivers/xgl](https://github.com/GPUOpen-Drivers/xgl/blob/c6c90450fb1abf5f414acf1e38a0f51a72c426c1/icd/settings/settings_xgl.json#L1152) {{< /quote >}}
 >
 >       // Represents configuration of static registers relevant to hardware primitive shader (NGG).
 >
 > {{< quote >}} [llpc/Gfx9Chip.h at 93e40124f5067c8e932398204077843fb8445594 · GPUOpen-Drivers/llpc](https://github.com/GPUOpen-Drivers/llpc/blob/93e40124f5067c8e932398204077843fb8445594/lgc/patch/Gfx9Chip.h#L316) {{< /quote >}}

NGGカリング/プリミティブシェーダーは、レンダリングパイプラインにおいて、後に描画されずに破棄される対象を、計算する前に破棄する早期カリング機能の 1つ。従来のレンダリングパイプラインでは、頂点処理を行なった後に破棄するため、演算リソースが無駄となりやすい。  

今回のパッチでは *「for better performance」* とあるだけで具体的な性能向上の度合いには触れられていないが、  
*Vega /GFX9* の資料には、従来のパイプラインと比較して、カリング処理の速度をサイクルあたり 4倍以上に引き上げられたとある。[^vega-whitepaper]  

[^vega-whitepaper]: <https://en.wikichip.org/w/images/a/a1/vega-whitepaper.pdf> (Page7)

最終的な性能への影響、ゲームにおける効果等はまだ不明だが、*Vega /GFX9* から約 3年越しで *RDNA /GFX10* 、*RDNA 2 /GFX10.3* 世代でついにデフォルトで有効化されるというのは感慨深いものがある。  

{{< ref >}}

 * [西川善司の3DGE：新設の「プリミティブシェーダ」を搭載し，Radeon RX Vegaはどこへ行く？](https://www.4gamer.net/games/337/G033714/20170804085/)

{{< /ref >}}
