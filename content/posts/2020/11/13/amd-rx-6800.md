---
title: "AMD GPU としては珍しい構成の Radeon RX 6800"
date: 2020-11-13T22:33:49+09:00
draft: false
tags: [ "RDNA_2", "Sienna_Cichlid" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

先日、AMD の [Marek Olšák](https://gitlab.freedesktop.org/mareko) 氏により [Mesa3D](https://gitlab.freedesktop.org/mesa/mesa) に投稿されたマージリクエストの中に、以下のようなものがあった。  

 >       uint32_t max_se;             /* number of shader engines incl. disabled ones */
 >       uint32_t num_se;             /* number of enabled shader engines */
 >
 > {{< quote >}} [ac,radeonsi: many NGG improvements and fixes, etc. (!7542) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7542/diffs?commit_id=1aef37c44a47720f74cf36af5543f884d096606a) {{< /quote>}}
 >
 >     	   /* Guess the number of enabled SEs because the kernel doesn't tell us. */
 >    	   if (info->chip_class >= GFX10_3 && info->max_se > 1) {
 >    	      unsigned num_rbs_per_se = info->num_render_backends / info->max_se;
 >    	      info->num_se = util_bitcount(amdinfo->enabled_rb_pipes_mask) / num_rbs_per_se;
 >    	   } else {
 >    	      info->num_se = info->max_se;
 >    	   }
 >
 > {{< quote >}} [ac,radeonsi: many NGG improvements and fixes, etc. (!7542) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7542/diffs?commit_id=1aef37c44a47720f74cf36af5543f884d096606a) {{< /quote>}}
 >

ドライバーが検出し、表示する GPU の情報一覧に、有効にされている ShaderEngine数が追加された。これまで ShaderEngine数については、その GPU が持つ最大数、`max_se` のみを検出していた。  
そして引用したコード部は、*RDNA 2 / GFX10.3* 世代の GPU で、最大 ShaderEngine 数が 1基より多い場合、ShaderEngineあたりの RB (Render Backend) と有効にされている RB数から、有効な ShaderEngine数を求める処理を行なっている。  
{{< comple >}} それとこれはついでだが、ShaderArray の略称がこれまで SH か SA かで分かれていて、ソースファイル上にどちらも存在することもあったが、ようやく SA で統一されそうな向きを見せている。{{< /comple >}}  

これまでは `max_se` だけで十分だったのに、何故今になって有効 ShaderEngine数を検出し、その処理を *RDNA 2 / GFX10.3* 世代の GPU を対象に絞っているのか。  
それは恐らく {{< comple >}} だがほぼ確実に {{< /comple >}} **Radeon RX 6800** が AMD GPU としては珍しい、いや唯一と言えるかもしれない構成を取っているからだ。  


## ShaderEngine を 1基無効化 {#dis-1se}

*Radeon RX 6000シリーズ* が正式発表された際にも書いたが、**RX 6800** は 30WGP (60CU) という仕様から、ShaderEngine 4基中 1基を無効化しているものと思われる。  
{{< link >}} [AMD RDNA 2 GPU 正式発表　―― 各スペック、Infinity Cache、プロセス | Coelacanth's Dream](/posts/2020/10/29/amd-rdna_2/#spec) {{< /link >}}
AMD GPU はスケジューリング等の観点から、複数の ShaderEngine が対称となるように設定される。  
同じ GPUダイを採用する他の **Radeon RX 6000シリーズ** 、**RX 6900 XT** は ShaderEngine すべてをフルスペックで有効にすることで総 40WGP を、**RX 6800 XT** は各 ShaderEngine 内の WGP を 1基無効化することで総 36WGP というスペックを実現している。  
だが、**RX 6800** の総 30WGP というのは、ShaderEngine 4基の対称性を維持したままでは実現できないため、ShaderEngine を 1基無効化、他 ShaderEngine 3基はフルスペック (各 10WGP) で有効にしていると考えられる。  

この ShaderEngine 1基を無効化というのが、**RX 6800** を唯一たらしめている。  

これまでの AMD GPU は製品化において、WGP/CU といったユニットを一部無効化することで性能を下げ、代わりにその上位モデルより安い価格で提供する下位モデルを用意してきた。WGP/CU の一部無効化は、欠陥のあるダイの救済、歩留まり向上という意味もある。  
*RDNA / GFX10* からは、RB がメモリコントローラーに接続される L2キャッシュではなく、L1キャッシュに接続されるようになったため、そうグラフィクス性能を下げることなく、メモリバス幅を狭めた製品構成も可能になった。  
**RX 5500 XT** 等のベースとなる [Navi14](/tags/navi14) は ShaderEngine 1基の構成であり、ShaderEngine 内の ShaderArray 2基については非対称構成が許容されているため、11WGP (22CU) といった仕様が可能となっている。  

ShaderEngine を丸々 1基無効化することはなく、対称性を維持するため、各 ShaderEngine 内の WGP/CU を 1、2基程度無効化するというのが通常だった。  
だからこれまでは `max_se` の値その情報で事足りていた。  

**RX 6800** はそこから外れ、ShaderEngine 1基を無効化する。  
従来の方法 {{< comple >}} 例えば、ShaderEngine 4基の WGP を 2基ずつ、RB+ を 1基ずつ無効化することで 32WGP (64CU)、12RB+ (96ROP) とするような構成 {{< /comple >}} を取らなかった理由としては、  
ShaderEngine には、ジオメトリプロセッサ、ラスタライザ/プリミティブユニット、RDNA L1キャッシュ 128KB が含まれるため (ジオメトリプロセッサ以外は ShaderArray ごとに存在する)、これらも無効化する方がグラフィクス性能で上位モデルと差別化するのに都合が良かったのではないか、等が考えられる。  

性能という点では他の **Radeon RX 6000シリーズ** より抑えられているが、**RX 6800** は、AMD GPU として初の内部構成/設定というオリジナリティを持つ。  

また、GPU消費電力 {{< comple >}} GPUチップ単体かボード全体かは明言されていないが、USB-C PD の分を引くため、GPUチップ単体の消費電力を記載しているのかもしれない {{< /comple >}} は他モデルより 50W 低い 250W であり、カード厚も 2スロットであるから扱いやすいモデルとも言えるだろう。  

