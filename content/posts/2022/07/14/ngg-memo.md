---
title: "NGG に関するメモ ―― Vega/GFX9 世代の NGG, Mesh Shader"
date: 2022-07-14T16:22:15+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "NGG", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD GPU 向け Vulkan ドライバー **RADV** とそのコンパイラバックエンドに採用されている **ACO** の開発者として知られる Timur Kristóf 氏が、ブログにて *RDNA 1/GFX10* 世代から有効化されている *NGG (Next Generation Geometry) /NGG パイプライン* とドライバー側で行われる *NGG カリング* について解説した記事を公開している。  

 * [What is NGG and shader culling on AMD RDNA GPUs? | Timur’s blog](https://timur.hu/blog/2022/what-is-ngg)

*NGG* に関する情報は **ACO バックエンド** のドキュメント[^aco-readme] や各種ドライバーへのパッチで公開されてきたが、それらは断片的であり、また AMD 公式的には *Vega/GFX9* 世代に *NGG/Primitive Shader* を *大々的* に宣伝していたが、*RDNA 1/GFX10.1* 世代からはあまり情報を公開しなくなった。  
そのため、GPU ドライバー開発者の視点から *NGG* を体系的にまとめた Timur Kristóf 氏の記事は、AMD GPU のグラフィクスパイプラインの沿革を知る上でとても価値のあるものだと言える。  

[^aco-readme]: [src/amd/compiler/README.md · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/amd/compiler/README.md#supported-shader-stages)

この記事自体は Timur Kristóf 氏の記事を紹介することと、あとは個人的なメモが目的のものとなる。  

## NGG on Vega/GFX9 {#ngg-gfx9}
*NGG/Primitive Shader* と呼ばれる機能は *Vega/GFX9* 世代にも実装されていたが、Kernel Mode Driver (KMD) である AMDGPU ドライバーでは一部対応が進められていたものの、User Mode Driver (UMD) がそれを活用することはなく、実質封印された機能となった。  
AMDGPU ドライバー側の実装も 2019-09 に関連コードを削除するパッチが AMD の Marek Olšák 氏によって投稿されている。  
*Vega/GFX9* 世代での *NGG* にそれ以上対応しないことはそれよりも早く、2018-08 の時点で AMD の Jin, Jian-Rong 氏より語られていた。  

 * [Making a GDS Allocation for NGG](https://lists.freedesktop.org/archives/amd-gfx/2018-August/025320.html)
 * [[PATCH] drm/amdgpu: remove gfx9 NGG](https://lists.freedesktop.org/archives/amd-gfx/2019-September/040254.html)

*RDNA 1/GFX10.1* 世代からの *NGG* は新たに設計されたもので、*Vega/GFX9* 世代とは互換性がないものとされている。  
また、Marek Olšák 氏は [Phoronix](https://www.phoronix.com) のフォーラムにて、*「Vega/GFX9 で NGG が (利点が得られつつ) 機能するチャンスはまだありますか？」* というコメントに対して、はっきりと *「ない、まったく異なり (筆者補足: RDNA 1/GFX10 世代とは?) 、むしろ遅くなる。」* と返答している。[^phoronix-forum]  

[^phoronix-forum]: <https://www.phoronix.com/forums/forum/linux-graphics-x-org-drivers/open-source-amd-linux/1328443-amd-lands-a-number-of-radeonsi-rdna-ngg-fixes-ahead-of-rdna3-enabling?p=1328449#post1328449>

現在は削除されている AMDGPU ドライバー側の実装から、*Vega/GFX9* 世代の *NGG* は Primitive/Position/Control Sideband Buffer を VRAM側に、GDS (Global Data Share) も一部を *NGG* 用に確保する仕様となっていた。  
推測だが、こうした仕様が性能上のボトルネックになっていた可能性はある。  
*RDNA 1/GFX10.1* 世代からの *NGG* にそうした KMD側のバッファ確保等は必要なく、UMD側ですべて制御可能となっている。  

### NGG 有効時の性能 {#perf}
*RDNA 1/GFX10.1* 世代からの *NGG* を用いた場合の性能について、Timur Kristóf 氏の言ではただ従来のパイプラインから *NGG* に切り替えるだけでは性能は特に変わらない。  
先に書いたように *Vega/GFX9* 世代において *NGG* は大々的に宣伝されていたため、Timur Kristóf 氏はこの結果に非常に驚いたと語っている。  
しかしこれは *NGG* でも従来と同じようにシェーダーをコンパイルしていたため、*NGG* の新機能を一切用いていなかった。後に *NGG カリング* を実装したことで、*NGG* 有効時の性能は向上した。  

カリング機能は無駄を省くことで、実性能を向上させる手法であるため、性能の向上幅はアプリケーションに依存する。アプリケーション側にすでにそうした機能が実装されている場合、*NGG カリング* は効果がないか、かえって性能を低下させることも考えられる。  

こうしたことから Timur Kristóf 氏は、*NGG* はゲーミング性能を向上させる銀の弾丸ではなく、ドライバーへの新機能や新しいプログラミングモデルの実装を可能にするイネーブラだと語っている。  

しかし、**RadeonSI (OpenGL)** も **RADV (Vulkan)** も環境変数から *NGG カリング* の有効無効を切り替えられるため、実際に *NGG カリング* の有効で性能が顕著に低下するアプリケーションがあっても `drirc` の設定によって簡単に対処できると思われる。  

現状 *NGG カリング* がデフォルトで有効化される条件は、**RadeonSI (OpenGL)** では *RDNA 1/GFX10.1* 世代以上かつ RB (RenderBackend) 2基以上の場合に、**RADV (Vulkan)** では RB数は **RadeonSI (OpenGL)** と同じ条件だが、世代は *RDNA 2/GFX10.3* 世代以上の場合となっている。[^radeonsi-nggc][^radv-nggc]  
また、*RDNA 1/GFX10.1* 世代であっても *Navi14 (gfx1012)* はハードウェア側にバグがあるため、*NGG* を有効化することができない。  
*Cyan Skilfish/Skillfish APU (gfx1013)* も *RDNA 1/GFX10.1* 世代だが、UMD側では対応が行われていないため、詳細は不明。  

[^radeonsi-nggc]: [src/gallium/drivers/radeonsi/si_pipe.c · a1763ad4b362c9f3a1fd12b6d06009b17fac3d24 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/a1763ad4b362c9f3a1fd12b6d06009b17fac3d24/src/gallium/drivers/radeonsi/si_pipe.c#L1301-1309)
[^radv-nggc]: [src/amd/vulkan/radv_device.c · 9035408d62f411aabc4df017a77969fca85ad9b9 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/9035408d62f411aabc4df017a77969fca85ad9b9/src/amd/vulkan/radv_device.c#L793-797)

*VanGogh APU* は GPU部が *RDNA 2/GFX10.3* 世代であり、*NGG* と *NGG カリング* が有効可能だが、*NGG カリング* による性能向上幅は dGPU よりも低くなっているという。  
これについては、Timur Kristóf 氏が以前にパッチ内で語った、*NGG カリング* の効率は Pixel Shader のスループットに依存するということと、APU では PS へのオンチップバッファ Parameter Cache (PC) の規模が dGPU より小さいことで説明できる。  
{{< link >}} [RADV + RDNA 2 で NGGカリングがデフォルトで有効に | Coelacanth's Dream](/posts/2021/09/29/radv-enable-nggc-default-on-rdna_2/#nggc) {{< /link >}}
他の *RDNA 2/GFX10.3 APU*、*Yellow Carp/Rembrandt APU* や *GC 10.3.7/Sabrina APU* でも同様の傾向が見られるのではないかと思われる。  

## NGG と Mesh Shader {#ms}
Timur Kristóf 氏は XDC2020 にて **ACO** の今後の計画について発表した際、*NGG* で Mesh Shader を実行できる *可能性* について触れていたが、今回の記事内でプリミティブごとの出力機能などが *RDNA 1/GFX10* 世代の *NGG* には実装されていないため、*NGG* が有効であっても Mesh Shader への対応は *RDNA 2/GFX10.3* 世代からに限定されると語っている。  
{{< link >}} [【XDC2020】 ACOバックエンドの今後の計画 ―― RDNA 2, RT, Mesh Shader | Coelacanth's Dream](/posts/2020/09/19/xdc2020-aco/#ms-on-ngg) {{< /link >}}
*NGG パイプライン* として基本的な部分は共通しつつも、一部の機能は世代によって異なることとなる。  
*RDNA 3?/GFX11* 世代ではついに従来のパイプラインは廃され、*NGG* のみの実装となるが、こうしたことから新機能や最適化に関わる部分が変更されている可能性がある。  
{{< link >}} [RadeonSI ドライバーで GFX11 のサポートが進められる | Coelacanth's Dream](/posts/2022/05/05/radeonsi-gfx11/#ngg) {{< /link >}}
