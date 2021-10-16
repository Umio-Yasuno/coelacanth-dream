---
title: "Intel のマルチタイル GPU をサポートするパッチが投稿される"
date: 2021-10-10T23:10:32+09:00
draft: false
tags: [ "Gen12", "Xe-HP" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

intel-gfxメーリングリストに、Linux Kernel に *{{< xe class="hp" >}}* のようなマルチタイル構成を採る GPU の初期的なサポートを追加するパッチが投稿された。  

 * [[Intel-gfx] [PATCH 00/11] i915: Initial multi-tile support](https://lists.freedesktop.org/archives/intel-gfx/2021-October/280126.html)

*{{< xe class="hp" >}}* 含め、今後の Intel GPU プラットフォームではマルチタイル構成が採られ、そこでは複数の GPUタイル (ダイ) とそれぞれに接続されたローカルなメモリが存在する。  
今回のパッチは初期的なサポートを追加するもので、複数のタイルで構成されながら単一の GPUデバイスとして認識される完全なマルチタイルプラットフォームのサポートにはさらなる作業、特にローカルメモリに関する作業が必要だと語られている。  
なお、パッチ中では **GT** という単語が使われているが、*Graphics Technology* の略であり、Intel GPU ドライバーでは単に GPU を指す際に用いられる。[^gt]  

[^gt]: <https://github.com/torvalds/linux/blob/bdb575f872175ed0ecf2638369da1cb7a6e86a14/drivers/gpu/drm/i915/Makefile#L79>

*AMD CDNA 2 アーキテクチャ* を採用すると目される [AMD Aldebran/MI200 GPU](/tags/aldebaran) もまた 2基の GPUダイで構成される。  
{{< link >}} [プライマリーダイとセカンダリーダイで構成される Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/06/09/aldebaran-primary-secondary/) {{< /link >}}
ダイ・チップレット・タイルと用語がバラバラに使われているが、Raja Koduri 氏は以前 Intel 社内ではバンプピッチ 55um以下の高密度実装を必要とするシリコンを「タイル」、標準的なパッケージに実装されるシリコンを「チップレット」と、区別していると語っていた。[^tile-chiplet]  
バンプピッチ 55um以下の高密度実装には EMIB、Foreros が該当し、*{{< xe class="hp" >}}* では各 GPUダイが EMIB で接続されるため「タイル」と呼ぶのが適当となる。  
AMD はまだそうした使い分けに関しては触れていない (と記憶している) が、パッチやオープンソースドライバー中では主に「ダイ」が使われている。  

[^tile-chiplet]: <https://twitter.com/Rajaontheedge/status/1374721309372948492>

## マルチダイ/タイル GPU のソフトウェアサポート {#multi-die-tile-support}

マルチダイ/タイル構成を GPU に導入する上で、ソフトウェア、ドライバーにおけるサポートは Intel, AMD で共通する部分がある。  
今回のパッチで Matt Roper 氏は、複数のタイルで構成されることはユーザースペース、UMD (User Mode Driver) に対して透過的であり、直接意識する必要はないとコメントしている。これはマルチダイ/タイル GPU のプログラミングモデルを簡易にしつつ、性能を引き出す上で重要となる。  
KMD (Kernel Mode Driver) が複数のダイ/タイルを意識し、ユーザースペースには単一の GPU であるように見せる。  
また、CPU と接続されるダイ/タイルはプライマリ/ルートと呼ばれる。  

 > 		Only the primary/root tile is initialized for now; the other tiles will
 > 		be detected and plugged in by future patches once the necessary
 > 		infrastructure is in place to handle them.
 > 
 {{< quote >}} [[Intel-gfx] [PATCH 03/11] drm/i915: Restructure probe to handle multi-tile platforms](https://lists.freedesktop.org/archives/intel-gfx/2021-October/280129.html) {{< /quote >}}

こうした点は *AMD Aldebaran/MI200* においても同じであり、*Aldebaran/MI200* はプライマリダイとセカンダリダイで構成される。そして、AMD のマルチダイ構成における関連特許から、単一の GPU として認識されると考えられる。  
{{< link >}} [プライマリーダイとセカンダリーダイで構成される Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/06/09/aldebaran-primary-secondary/) {{< /link >}}
*Aldebaran/MI200* は CPU とも *XGMI/Infinity Fabric* で接続し、CPU と GPU のメモリ空間を統合する *3rd Gen Infinity Architecture* を想定している関係で、サポートにおいてドライバー部に必要な変更は *{{< xe class="hp" >}}* よりも広い。  

マルチダイ/タイル構成では、単一の GPU に見せつつ、処理をいかに各ダイ/タイルに分散し、メモリアクセスを最適化するかが重要になると考えられる。  
ただ、Intel, AMD 共にまだ分散処理の全容は明かしていない。  
*Aldebaran/MI200* ではプライマリダイが、セカンダリダイと合わせたソケットレベルの電力情報を返すことから、分散処理以外に GPUダイ間のみでそれぞれの温度や電力情報をやり取りしていると推測される。  
{{< link >}} [プライマリーダイがまとめて電力情報を報告する Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/09/04/primary-die-report-total-power/) {{< /link >}}

マルチダイ/タイルの統合を、どこまでハードウェア側で行い、どこまでを KMD で行ってユーザースペースに見せるかは今後も Intel, AMD で実装が分かれていくと思われる。  
