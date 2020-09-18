---
title: "AMD GPU の世代における FMA、MAD 命令の微妙な仕様と違い"
date: 2020-09-18T09:45:41+09:00
draft: false
tags: [ "RadeonSI", "RADV" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

最近 Mesa3D に組み込まれた変更の中に以下のようなコメントがあった。  

 >      /* gfx6-8: use MAD (FMA is 4x slower)
 >       * gfx9-10: either is OK (MAD and FMA have the same performance)
 >       * gfx10.3: use FMA (MAD doesn't exist, separate MUL+ADD are 2x slower)
 >       *
 >       * FMA has no advantage on gfx9-10 and MAD allows more algebraic optimizations.
 >       * Keep FMA enabled on gfx10 to test it, which helps us validate correctness
 >       * for gfx10.3 on gfx10.
 >       */
 >
 > {{< quote >}} [nir,radeonsi: move ffma fusing to late optimizations for better codegen (!6596) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/6596/diffs?commit_id=758ab39d25e10d585929b87a8a2891c5a68b7c55) {{< /quote >}}

AMD GPU の各世代における、FMA と MAD の処理性能の違いらしいが、初期の GCN から *Polaris1x /VegaM* までを含む *GFX6-8* では FMA が MAD の 4倍遅いため MAD を使い、
*Vega* や *Navi* の世代である *GFX9-10* では FMA も MAD も同じであるため、どちらでも良く、  
そして AMD の次世代 GPU *RDNA 2 /GFX10.3* では MAD が存在せず(実装されていない？)、2つに分けて処理するため、FMA を使うとしている。  

こうした話は初耳であり、特に *GFX6-8* の FMA の処理が MAD の 4倍遅いというのは衝撃的だ。  
そこで自分なりに調べ、上記コメントを読み解いてみた。  

## FMA と MAD の違い {#fma-mad-diff}
FMA(Fused Multiply Add) も MAD(Multiply Add) はどちらも 3つのオペランドを必要とする積和演算だが、MAD は乗算の結果を、掛けた値と同じビット数に丸めてから加算の処理を行なうが、  
FMA は乗算の結果を掛けた値の倍のビット長で保持してから加算を行なうため、FMA の方が計算精度が高くなる、という違いがある  
{{< link >}} 参考: <link>[科学技術計算向け演算能力が引き上げられたGPUアーキテクチャ「Fermi」 (2) 科学技術計算向けのさまざまな工夫 | マイナビニュース](https://news.mynavi.jp/article/20091007-nvidia_fermi/2)</link> {{< /link >}}

## 微妙な仕様？と世代ごとの違い
まず、冒頭で引用したコメント部分が追加され、影響を受けるのは、シェーダーを GPU が実行する中間コードに変換するシェーダーコンパイラの部分である。  

そして、*GFX6-8* では FMA が MAD の 4倍遅くなることについて過去のパッチにヒントがあった。  
どうも、Vulkan SPIR-V において FMA は MAD に相当するが、FMA の Opcode(Operation Code, 命令の動作を指定する部分) は AMD GPU *(GFX6-8)* のハードウェア内部? では 2つの Opcode のように発行されるため処理が遅くなるらしい。[^fma-double-opcode]  
これが関係して、FMA の実行が MAD の 4倍遅くなるということと思われるが、これ以上の詳細は不明である。AMD が公開している Shader ISA の資料を漁ってみたが、このことについて言及している部分は見当たらなかった。  
それとして、*GFX6-8* ではこれを回避するため、ドライバー部で FMA を MAD として実行するよう中間コードを生成している。[^fma-to-mad]  
性能低下に対し、シェーダーコンパイラ側でこういった対策を取ったのは少し不思議に思えるが、これは命令の発行を行なうユニットが CU内にあり、ブート時に読み込まれるファームウェアやマイクロコードでは対処できなかったのかもしれない。  

[^fma-double-opcode]: [radv: lower ffma in nir. (2c61594d) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/2c61594d84911f486aa2edb4b8e561e780139d20)
[^fma-to-mad]: [radeonsi: lower ffma in nir to mad. (80bbdb14) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/80bbdb148335c55303960bab841d98f4fbd1feea)

これが修正されたのか、*GFX9-10* では FMA も MAD も同じ性能で処理が可能となっており、FMA としても MAD として実行しても変わらないものとなった。  

*RDNA 2 /GFX10.3* では、恐らく MAD 演算器が省かれ {{< comple >}} あくまでコードのコメントから読み取ったものであり、確かではない {{< /comple >}}、MAD は MUL(乗算) と ADD(加算) の 2つに分けて演算を行なうようになったため遅くなり、FMA の方が明確に速くなった。  
コード中では、まだ *RDNA 2 /GFX10.3* で十分に検証が行なえないため、*RDNA /GFX10.1* から MAD ではなく FMA で処理するようになっている。  
オープンソースで開発されているドライバーは、企業内部でテストに使われているハードウェアよりも多くのユーザーが手にしているハードウェアを対象とした方が開発がスムーズに進むのだろう。  
*RDNA 2 /GFX10.3* で MAD 演算器が省かれた(であろう) 理由については、WGP(2CU) の実装密度を引き上げるためと考えられるが、正しさを保証することはできない。AMD から詳細を解説してくれれば嬉しいが。  


{{< ref >}}

 * [科学技術計算向け演算能力が引き上げられたGPUアーキテクチャ「Fermi」 (2) 科学技術計算向けのさまざまな工夫 | マイナビニュース](https://news.mynavi.jp/article/20091007-nvidia_fermi/2)

{{< /ref >}}
