---
title: "RadeonSIドライバーから DFSM 機能のサポートが削除"
date: 2021-05-21T01:28:25+09:00
draft: false
tags: [ "RadeonSI", "GFX9" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

オープンソースで開発される AMD GPU の OpenGLドライバー **RadeonSI** に向けて、AMD の GPUドライバー開発者である [Marek Olšák](https://github.com/marekolsak) 氏より DFSM 機能のサポートを削除するパッチが投稿された。  
パッチのマージリクエストは、元は *Navi1x/GFX10.1* 世代の GPU に存在するハードウェアのバグに対する回避策や、 [NGG (Next Generation Geometry)](/tags/ngg) カリング機能の最適化等、細やかな修正、最適化を目的としたもので、DFSM 機能を削除する部分は後から追加されている。  

 * [radeonsi: remove DFSM after we discovered how bad it is (2e4ca54a) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/2e4ca54a02b60ca39699f00d3089c38b5a25fc7c?merge_request_iid=10813)
 * [radeonsi: disable DFSM on gfx9 by default because it decreases performance a lot (cc92c727) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/cc92c72798842958c58441cff08de6fe8324c4b1?merge_request_iid=10813)

## DFSM

DFSM 機能は *Vega/GFX9* 世代の AMD GPU からハードウェアに搭載された機能であり、その世代における改良の目玉とされた *DSBR (Draw Stream Binning Rasterizer)* を全体的に適用するのが DFSM とされる。[^pb]  
*DSBR* ではレンダリング領域を細かいタイルに分割し、頻繁に使われる部分を L2キャッシュに置くことで性能向上とメモリアクセスを減らすことで電力効率の向上を狙った機能。  
*Vega/GFX9* 発表時の資料では、有効時で最大 10%のフレームレート向上と 33% の使用メモリ帯域削減が確認できたとしている。[^vega-whitepaper]  
{{< link >}} [Vegaの新機能今いずこ | Coelacanth's Dream](/posts/2019/12/07/where-vega-new-feature/) {{< /link >}}

[^vega-whitepaper]: <https://en.wikichip.org/w/images/a/a1/vega-whitepaper.pdf>

DFSM が何の略かは正直不明で、1年と半年前の自分は *DFSM （Deterministic Finite State Machine）* の略だとしていたが、AMD の公式資料あるいは AMD のソフトウェアエンジニア方がそう呼ぶところは見付からなかった。唯一 [Phoronix](https://www.phoronix.com/) の [Michael Larabel](https://www.phoronix.com/scan.php?page=michaellarabel) 氏が記事内でそう書いており、恐らく過去の自分はそれを元にしたと思われるが、記事内にあるリンクからその略称とする根拠になるようなものは無い。[^phoronix]  
その記事のコメントに 「*Deterministic Finite State Machine* の意じゃないか」というものがあり、[Michael Larabel](https://www.phoronix.com/scan.php?page=michaellarabel) 氏はそれをそのまま追記した可能性がある。  
要は DFSM が何の略かは本当に不明で、 Phoronix や自分は一度二度 *Deterministic Finite State Machine* の略と説明したがそれは不確かな情報ということだ。  

[^pb]: [[Mesa-dev] [PATCH 2/2] radeonsi: disable primitive binning on Vega10 (v2)](https://lists.freedesktop.org/archives/mesa-dev/2017-October/172054.html)
[^phoronix]: [Performance-Boosting DFSM Support Flipped On & Off For RADV Vulkan Driver - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=DFSM-RADV-September-2019)

*DSBR* の部分的な適用が *DPBB* とされるが、こっちは *Raven Ridge APU* の資料で言及され、特許資料もあり、 *Deferred Primitive Batch Binning* の略であることが確認できる。[^rv-apu]  

[^rv-apu]: [presentation title - 1.05_AMD_APU_AMD_Raven_HotChips30_Final.pdf](https://old.hotchips.org/hc30/1conf/1.05_AMD_APU_AMD_Raven_HotChips30_Final.pdf)

### DFSM の効果

ドライバーからサポートが削除されることになった DFSM の性能への効果は実際どうだったかと言うと、まあ微妙だった。  
Vulkanドライバー **RADV** においては、*Raven APU* だといくつかのグラフィクスデモとゲームで 2-3% の性能向上が確認できたことからデフォルトで有効化――された後に一部のゲームでは逆に 3% 性能低下したためデフォルトでは無効状態に戻されている。  
**RADV** 開発者の一人である [Bas Nieuwenhuizen](https://www.basnieuwenhuizen.nl/about/) 氏が検証した限りではベンチマーク (dual_quad_bench) で最大フィルレートが 16 pixels/cycles から 32 pixel/cycles に倍の性能向上があったとコメントしている。[^dfsm-fillrate] シンプルなベンチマークでは DFSM の効果が最大限発揮されるが、ゲームといった現実的なアプリケーションではそう単純には行かなかったということだろう。  


[^default-dfsm]: [radv: Enable binning and dfsm by default on Raven. (17b5a59b) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/17b5a59b4ee3adb9c99f3d850eb4a561196c69a0)
[^disable-dfsm]: [radv: Disable dfsm by default even on Raven. (0fa27400) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/0fa2740059a05e47854240ff8a6782d879389525)
[^dfsm-fillrate]: [radv: Add DFSM support. (4b7e7956) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/4b7e7956f0f161c958f570f1201517d50e5d3ed4)

DFSM 自体は *Vega/GFX9* の次世代、*Navi1x/RDNA/GFX10.1* でも引き継がれたが、有益な効果は得られないとコード中のコメントに記述され、さらにその次世代 *Navi2x/RDNA 2/GFX10.3* ではサポートがとうとう削除された。  
それでも *Vega/GFX9* 世代の GPU を搭載する APU は今も多くが出回っており (*Raven2/Renoir/Cezanne*)、DFSM は今までドライバーでのサポートが残され、APU の場合はデフォルトで有効にする処置が取られていたが、*Raven2 APU* だとプリミティブ描画のレートを最大 43%も低下することが判明し、今回サポートが削除されることになった。  

 > 		      /* DFSM is not supported on GFX 10.3 and not beneficial on Navi1x. */
 > 		   } else if (sscreen->info.chip_class == GFX9) {
 > 		      sscreen->dpbb_allowed = !sscreen->info.has_dedicated_vram;
 > 		      /* DFSM reduces the Raven2 draw prim rate by ~43%. Disable it. */
 > 		      sscreen->dfsm_allowed = false;
 > 		    }
 >
 > {{< quote >}} [radeonsi: disable DFSM on gfx9 by default because it decreases performance a lot (cc92c727) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/cc92c72798842958c58441cff08de6fe8324c4b1?merge_request_iid=10813) {{< /quote >}}

*Raven2 APU* の GPU L2キャッシュは小さく、*Raven/Picasso/Renoir/Cezanne APU* が 1MiB 持つ中、その 1/8 の 128KiB しか持たない。  
{{< link >}} [Raven2 の GPU L2キャッシュは 128KB | Coelacanth's Dream](/posts/2021/02/11/raven2-gpu-l2c-size/) {{< /link >}}
DFSM に使用メモリ帯域を削減する効果があり、それはメモリ帯域がディスクリートGPUより狭い APU では性能への良い影響を与える可能性があったが、L2キャッシュを多く活用するため、L2キャッシュが小さい *Raven2 APU* では逆に働いてしまったのではないかと考えられる。  

*DSBR* を部分的に適用する *DPBB (Deferred Primitive Batch Binning)* は対照的に優秀とも言え、性能向上はおとなしいが低下されることはなく、*Vega/GFX9* 世代の以降の GPU では APU、dGPU 問わずデフォルトで有効化され、*Navi2x/RDNA 2/GFX10.3* 世代でもハードウェアに機能が搭載されている。  
ただ、ソースコードを追うと RenderBackend が 4基 (ROP 16基相当、RB+ は 32基相当[^rbplus]) 搭載されている GPU のような場合では (他の条件もあるが) 非効率なためか無効化されるようだ。  
GPU の規模が小さく、メモリ帯域も狭い APU 向けという所は *DPBB* も同じだと言える。  

 > 		   /* Disable DPBB when it's believed to be inefficient. */
 > 		   if (sscreen->info.max_render_backends > 4 && ps_can_kill && db_can_reject_z_trivially &&
 > 		       sctx->framebuffer.state.zsbuf && dsa->db_can_write) {
 > 		      si_emit_dpbb_disable(sctx);
 > 		      return;
 > 		   }
 >
 > {{< quote >}} [src/gallium/drivers/radeonsi/si_state_binning.c · cc92c72798842958c58441cff08de6fe8324c4b1 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/cc92c72798842958c58441cff08de6fe8324c4b1/src/gallium/drivers/radeonsi/si_state_binning.c) {{< /quote >}}

[^rbplus]: [一部の AMD GPU で実装、有効化されている RB+ とは何か | Coelacanth's Dream](/posts/2020/11/10/what-is-rbplus/)

ほとんどの機能は性能向上、電力効率の向上を目的に開発、実装されるが、その全てがうまく働いてくれる訳ではないという現実を DFSM は見せてくれたのかもしれない。  
実装されても一切サポートされなかった機能がある可能性を思えば、DFSM は一時期でも注目、活用されたことはまだ幸いだったとも考える。  
