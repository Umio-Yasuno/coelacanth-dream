---
title: "プライマリーダイとセカンダリーダイで構成される Aldebaran"
date: 2021-06-09T17:12:30+09:00
draft: false
tags: [ "Aldebaran", "MI200", "gfx90a", "Linux_Kernel" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

*CDNA 2 アーキテクチャ* を採用し、2基の GPUダイによる MCM 構成を採ると目される *Aldebaran* GPU だが、Linux Kernel (amd-gfx) メーリングリストにて AMD の [Lazar, Lijo](https://in.linkedin.com/in/lijo-lazar-4488073) 氏より公開されたパッチから、GPUダイがプライマリーダイ (Primary Die) とセカンダリーダイ (Secondary Die) で分かれて *Aldebaran* を構成することが分かった。  
{{< link >}} [Aldebaran GPU は 2-Die 構成か | Coelacanth's Dream](/posts/2021/03/05/amd-aldebaran-2die-mcm/) {{< /link >}}

 * [[PATCH] drm/amd/pm: Only primary die supports power data](https://lists.freedesktop.org/archives/amd-gfx/2021-June/065161.html)

 > 		[Public]
 > 		
 > 		On aldebaran, only primary die fetches valid power data. Show
 > 		power/energy values as 0 on secondary die. Also, power limit should not
 > 		be set through secondary die.
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: Only primary die supports power data](https://lists.freedesktop.org/archives/amd-gfx/2021-June/065161.html) {{< /quote >}}

パッチの目的としては、消費電力の取得が可能なのがプライマリーダイのみであるため、取得とパワーリミットの設定をプライマリーダイのみに行うようにするというもの。  

## Primary, Secondary

プライマリー、セカンダリーといった単語は、AMD の GPUチップレット技術に関する特許資料中においても用いられている。  

 * [ACTIVE BRIDGE CHIPLET WITH INTEGRATED CACHE - ADVANCED MICRO DEVICES, INC.](https://www.freepatentsonline.com/y2021/0097013.html)
 * [GPU CHIPLETS USING HIGH BANDWIDTH CROSSLINKS - ADVANCED MICRO DEVICES, INC.](https://www.freepatentsonline.com/y2020/0409859.html)

プライマリーダイ/チップレット (以下資料に合わせてチップレットの語を用いる) の役割はまず CPU からのメモリリクエストを受け取ることにあり、セカンダリーチップレット等その他チップレットは直接受け取ることが無い。  
プライマリーチップレットはメモリリクエストに対し、まずプライマリーが持つキャッシュ (ロカール) に対応したものかを確認する。対応したものであった場合は、GPUメモリにアクセスしキャッシュ、そして CPU にデータを送信する。  
対応したものでなかった場合は、プライマリーに接続された他のチップレットにメモリリクエストを送信し、対応したキャッシュ、メモリを持つチップレットはプライマリーを経由して CPU にデータを送信することとなる。  

{{< figure src="/image/2021/03/05/amdgpu-fabric.webp" title="GPU Memory, I/O, and Connectivity" caption="画像出典: [ORNL_Application_Readiness_Workshop-AMD_GPU_Basics.pdf](https://www.olcf.ornl.gov/wp-content/uploads/2019/10/ORNL_Application_Readiness_Workshop-AMD_GPU_Basics.pdf)" >}}

また特許資料中では、同一のパッケージに搭載された複数のチップレットは、プログラムからは単一の GPU として認識され、GPU へのコンピュート処理は分散されるとしている。  
分散処理がどう行われるかはまだ読み取れていないが、KMD (Kernel Mode Driver) 部で行われるか、あるいは CPU からのメモリリクエストをプライマリーチップレットのみが受け取ることを踏まえればプライマリーチップレットが担うことが考えられる。  

*Aldebaran (gfx90a)* はフロントエンド部における新機能として *TgSplit (Thread group Split)* をサポートしている。  
通常 1つの Workgroup は同じ CU に割り振られ、CU内の異なる SIMDユニットで実行されるが、*TgSplit* では Workgroup を分割し、異なる CU に割り振ることを可能とする。  

Workgroup は CUDA における Thread Block に相当し、GPU 上で同時に存在する Waveflont のグループ。同 Workgroup に属するスレッドは同期とローカルメモリを介した通信が可能。また CU内の LDS (Local Data Share) は 1つの Workgroup 内のスレッド間で共有される。  
だが *TgSplit* では Workgroup が分割され、異なる CU で実行されるため、LDS が割り当てられない場合がある。  

LLVM に *TgSplit* のサポートが追加された当初自分は CU の稼働率を引き上げる機能ではないかと推測したが、*Aldebaran (gfx90a)* が MCM/チップレット構成を採り、それに関連した特許資料に単一の GPU として認識される旨が記述されていたことから、チップレットに処理を分散するための機能だと推測される。  
異なる Workgroup をそれぞれチップレットに割り振るのでは、特定のチップレットに集中しないようにしながら稼働率を引き上げることが難しく、またプログラム側で複数のチップレットを認識し、意識したマルチGPUプログラミングが必要になってしまうと思われる。  
{{< link >}} [LLVM に GFX90A のサポートが追加される　―― CDNA 2/MI200 か | Coelacanth's Dream](/posts/2021/02/19/llvm-gfx90a/#tgsplit) {{< /link >}}

他にも *Aldebaran* では、命令のプリフェッチ機能を大幅に強化する、L2キャッシュラインサイズを *RDNA/2 アーキテクチャ* 同様に 128 Bytes にする、GDS を削除する等、多数の変更/改良が施されていることが OSS へのパッチから確認できる。  
{{< link >}} [RadeonSI に Aldebaran GPU のサポートを追加するパッチが投稿される | Coelacanth's Dream](/posts/2021/03/04/aldebaran-umd/) {{< /link >}}
{{< link >}} [Linux Kernel に 「Aldebaran」 GPU をサポートするパッチが投稿される　―― CDNA 2/MI200? | Coelacanth's Dream](/posts/2021/02/25/amd-aldebaran-gpu/#removed-gds) {{< /link >}}
