---
title: "RadeonSI ドライバーから非同期コンピュートによるプリミティブカリング機能が削除"
# date: 2021-09-21T18:42:34+09:00
date: 2021-09-21T23:42:34+09:00
draft: false
tags: [ "RadeonSI", "NGG" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

先日、AMD のソフトウェアエンジニア [Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏より、非同期コンピュートを用いたプリミティブカリング機能を削除するパッチ (マージリクエスト) が投稿され、メインラインにも組み込まれた。  

 * [radeonsi: reviewed commits from !12343 (part 2) (!12812) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/12812/commits)

なお、今回削除されたのは非同期コンピュートベース、コンピュートシェーダーによるプリミティングカリング実装であり、*RDNA/GFX10* 世代とそれ以降でサポートされている *NGGカリング/プリミティブカリング* とは異なる。  

## Primitive culling with async compute {#prim-discard}

非同期コンピュートによるプリミティブカリングが **RadeonSI** に組み込まれたのは約 2年半前 (2019/02)。パッチは、今回削除するパッチを投稿したのと同じ [Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏によって投稿された。  
機能としては頂点シェーダーの前段で非同期コンピュートを活用したカリング処理を行うことで、トライアングル等のプリミティブがサンプルポイントと交差しないため、多数のジオメトリ処理を行うアプリケーションでのパフォーマンスを大幅に向上させる。  
GDC 2016 で発表された [Optimizing the Graphics Pipeline with Compute](https://archive.org/details/GDC2016Wihlidal) を元に実装されているため、より詳細を知りたい方はそちらを参照して頂きたく。  

 * [[Mesa-dev] [PATCH 00/26] RadeonSI: Primitive culling with async compute](https://lists.freedesktop.org/archives/mesa-dev/2019-February/215085.html)

実装時のパッチのコメントによれば、Paraview での性能比較において約 2-4倍の性能向上を得られている。多数のプリミティブを扱うためメモリ性能の影響が大きいのか、HBM系メモリを採用する **Radeon Fury X, Radeon Vega 56, Radeon VII** では約 3-4倍の性能向上となっている。  
パッチではゲームのベンチマークをする時間が無かったため、まずはすべての Pro カードで有効化するとコメントされている。  
後日、Linux をメインに OSS情報の発信、ハードウェアのレビューを行っている [Phoronix](https://www.phoronix.com/) の Michael Larabel 氏が検証したところ、ゲームでは性能向上の効果は無く、むしろ悪化しているケースが多かったため、Proカードのみデフォルトで有効化する点は最後まで変わらなかった。[^prim-discard-game]  


[^prim-discard-game]: [RadeonSI Primitive Culling Yields Mixed Benchmark Results - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=RadeonSI-Prim-Culling-Tests)

## 削除された理由 {#removed-prim-discard}

ゲームでは効果は無くとも、アプリケーションごとに有効無効の設定は可能であり、削除せずに残しても問題ないと思いもするが、同時に今回削除した理由として複数の理由が挙げられる。  

プリミティブカリング機能を削除する該当コミットで Marek Olšák 氏は、*「常に機能するものではなく、GFX9 (Vega) とそれ以前のみで効果的であり、そして (コードが) とても複雑だ。」* とコメントしている。  

 > 		It doesn't always work, it's only useful on gfx9 and older, and it's too
 > 		complicated.
 >
 > {{< quote >}} [radeonsi: reviewed commits from !12343 (part 2) (!12812) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/12812/diffs?commit_id=576f8394db652feffd6f57eaaf5fad4daa0ea409) {{< /quote >}}

常に機能するものではない、という点については、プリミティブカリングは常に有効化するオプションが存在はするが、デフォルトでは巨大なドローコールが呼ばれた時のみに用いられるようになっていた。  
毎回プリミティブカリング機能を使おうとすると、性能への悪影響や視覚的な影響が発生するのだと思われる。  

次に *GFX9 (Vega)* とそれ以前の AMD GPU のみで効果的というのは、*GFX10/RDNA, GFX10.3/RDNA 2* 世代では [NGG (Next Generation Geometry)](/tags/ngg) というハードウェアシェーダーステージが追加されており、*NGG* ではプリミティブの変更が可能となっている。  
そのため、*NGG* ではコンピュートシェーダーから頂点シェーダーに受け渡す必要なしにカリング処理を行うことができる。  
*GFX9 (Vega)* とそれ以前の世代のみで非同期コンピュートによるプリミティブカリングが役立つ、というよりも、*GFX9* 以降の *GFX10, GFX10.3* 世代ではもっと有用なカリング機能を実装できるから、と言った方が正確かもしれない。  
非同期コンピュート版は OpenGLドライバーである **RadeonSI** のみに実装されていたが、*NGG* と *NGG カリング* は Vulkanドライバー **RADV** にも実装されているのも大きい。  
[Phoronix](https://www.phoronix.com/) の Michael Larabel 氏が行った検証では、*NGG カリング* の有効化でいくつか性能が低下しているものもあるが、確かな性能向上を得られているゲームもあり、ここも非同期コンピュート版との大きな違いと言える。[^nggc-perf]  

[^nggc-perf]: [[Linux Gaming Performance With Radeon Vulkan NGG Culling - Phoronix](https://www.phoronix.com/scan.php?page=article&item=radeon-radv-nggc&num=4)]

*NGG* は **RadeonSI, RADV** 共に、*Navi14* 以外の *GFX10* とそれ以降の世代でデフォルトで有効化されている。[^ngg-default]
{{< link >}} [RADVドライバーに NGGカリングが実装 | Coelacanth's Dream](/posts/2021/07/26/radv-nggc/) {{< /link >}}
*Navi14* で *NGG* がデフォルトで有効化されない理由はハードウェア的にバグによるものとされている。[^navi14-hw-bug]  
*NGG カリング* は、**RadeonSI** では RenderBackend (RB = 4 ROPs, RB+ = 8 ROPs) が 2基以上という条件が付いているが、現時点で存在する *GFX10/RDNA, GFX10.3/RDNA 2* 世代の dGPU は条件をクリアしているため、デフォルトで有効化される。[^radeonsi-nggc]  
**RADV** では、安定したパフォーマンスの向上が確認できたなら *GFX10.3/RDNA 2* 世代でデフォルトで有効化するつもりであることをコメントで触れている。[^radv-nggc]  
{{< link >}} [RadeonSIドライバー + RDNA 2 では NGGカリング/プリミティブシェーダー がデフォルトで有効に | Coelacanth's Dream](/posts/2020/10/17/gfx103-default-ngg-culling/#fn:navi1x-pro-nggc) {{< /link >}}

[^ngg-default]: <https://gitlab.freedesktop.org/mesa/mesa/-/blob/99c5e03986294e3aa90be6dc656080d8304d3313/src/gallium/drivers/radeonsi/si_pipe.c#L1242>,<br> <https://gitlab.freedesktop.org/mesa/mesa/-/blob/f6d5cb233974ab32b8b62e5d1aff686122298150/src/amd/vulkan/radv_device.c#L686>
[^radeonsi-nggc]: <https://gitlab.freedesktop.org/mesa/mesa/-/blob/99c5e03986294e3aa90be6dc656080d8304d3313/src/gallium/drivers/radeonsi/si_pipe.c#L1246>
[^radv-nggc]: <https://gitlab.freedesktop.org/mesa/mesa/-/blob/f6d5cb233974ab32b8b62e5d1aff686122298150/src/amd/vulkan/radv_shader.c#L897>
[^navi14-hw-bug]: <https://gitlab.freedesktop.org/mesa/mesa/-/blob/395c0c52c72ce11c52130fecb98ed98cec79eeae/src/amd/common/ac_shader_util.c#L466>

そして、(コードが) とても複雑という点だが、該当コミットではある Issue をクローズしており、そこでは非同期コンピュートによるプリミティブカリングを有効化して UNIGINE Heaven を実行すると、GPU がハングすることが報告されている。  
その問題は後のパッチで修正されたが、他にもアーティファクト (視覚的なエラー?) が発生する問題もあった。  

まとめれば、*GFX10/RDNA* とそれ以降の世代ではもっと有用な *NGG*, *NGG カリング* が実装、有効化されている中で、ワークステーション向けのいくつかのアプリケーションのみで効果的に動作する、複雑なコードで実装された非同期コンピュート版を今後時間を掛けてメンテナンスする意味は薄い、と判断されたのだろう。  


