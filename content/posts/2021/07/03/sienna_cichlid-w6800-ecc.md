---
title: "ECC に対応する Radeon Pro W6800 と過去のパッチ"
date: 2021-07-03T15:26:42+09:00
draft: false
tags: [ "Sienna_Cichlid", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

こうした個人的なデータベースも兼ねたブログをやっていると、そう言えばあれはどうなった？という気掛かりを抱えることがままある。  
その 1つが *Sienna Cichlid/Navi21* に向けた、ECC機能で検出されたメモリエラーを記録し、レポートする機能を追加するパッチだった。パッチが投稿されたのは 2020/07/27、気付けば 1年近く前だ。  そして、そのパッチにはわずかではあるが HBMメモリに関係する記述があり、下リンク先のようなタイトルを付けた。  

 * [Navi21 /Sienna Cichlid のメモリは GDDR6 か HBM2 か、新たに出てきたパッチを読む | Coelacanth's Dream](/posts/2020/07/27/what-sienna_cichlid-hbm2/)

だが、 *Sienna Cichlid/Navi21* が対応するメモリタイプは GDDR6 で、実際に発表された製品も GDDR6メモリのみを採用している。  

記憶の片隅に置きながらそれ以上調べることはせず、新しい情報が公開されるのを待っていたが、現地時間 2021/06/08 付で *Sienna Cichlid/Navi21* をベースに、GDDR6メモリを採用しながら ECC機能に対応した **Radeon PRO W6800** が発表されたことから再度調べてみようという気になった。  
*RDNA 2 アーキテクチャ* を採用する **Radeon PRO W6000 Series** の発表を、このブログで取り上げなかったのは意図あってのものだが、ここではどうでもいい話なので流す。そのうち雑記に書くかも。  

 * [New AMD Radeon PRO W6000 Series Workstation Graphics with AMD RDNA 2 Architecture and Massive 32GB of Memory to Power Demanding Architectural, Design and Media Workloads :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1008/new-amd-radeon-pro-w6000-series-workstation-graphics-with)

調べ直した結論から言えば、そのパッチから *Sienna Cichlid/Navi21* が HBM(2)メモリに対応するといったようなことは読み取れなかった。  
つまりこの記事は、過去の自分に向かってお前はバカだと言うためのものとなる。  

## 使われていない定数 {#unused}

*Sienna Cichlid/Navi21* が HBM(2)メモリに対応しているのではないかという考えは、以下引用部の `UMC_V8_7_HBM_MEMORY_CHANNEL_WIDTH` から起こったが、端的に書くと、この定数は AMD GPU ドライバーのソースコード中のどこにも使われていない。  
同様の定数は [Vega20](/tags/vega20) 、[Arcturus/MI100](/tags/arcturus) に向けたコードにもあるが、そちらも使われていない。  
恐らく、将来的に使うことを想定して定義したがそのまま使われなかった、あるいは内部で開発中には使っていたのかもしれないが、パッチを公開する段階では使われなくなったものと思われる。  
それをそのまま残し、*Sienna Cichlid/Navi21* に向けたパッチを作成する時にベースにしたため使われない定数が引き継がれたのだろう。  
また、他の定数に関しても、UMC (Unified Memory Controller) あたりのメモリチャネルインスタンス? を求めるためのもので、GPU全体のメモリチャネル・バス幅・メモリコントローラー数はわからない。  

 > 		/* HBM  Memory Channel Width */
 > 		#define UMC_V8_7_HBM_MEMORY_CHANNEL_WIDTH	128
 > 		/* number of umc channel instance with memory map register access */
 > 		#define UMC_V8_7_CHANNEL_INSTANCE_NUM		2
 > 		/* number of umc instance with memory map register access */
 > 		#define UMC_V8_7_UMC_INSTANCE_NUM		8
 > 		/* total channel instances in one umc block */
 > 		#define UMC_V8_7_TOTAL_CHANNEL_NUM	(UMC_V8_7_CHANNEL_INSTANCE_NUM * UMC_V8_7_UMC_INSTANCE_NUM)
 > 		/* UMC regiser per channel offset */
 > 		#define UMC_V8_7_PER_CHANNEL_OFFSET_SIENNA	0x400
 >
 > {{< quote >}} [linux/umc_v8_7.h at 49070c4ea3d97b76c5666466efb35dcc42c6c8fd · torvalds/linux](https://github.com/torvalds/linux/blob/49070c4ea3d97b76c5666466efb35dcc42c6c8fd/drivers/gpu/drm/amd/amdgpu/umc_v8_7.h) {{< /quote >}}

<!--
 > 		/* HBM  Memory Channel Width */
 > 		#define UMC_V6_1_HBM_MEMORY_CHANNEL_WIDTH	128
 > 		/* number of umc channel instance with memory map register access */
 > 		#define UMC_V6_1_CHANNEL_INSTANCE_NUM		4
 > 		/* number of umc instance with memory map register access */
 > 		#define UMC_V6_1_UMC_INSTANCE_NUM		8
 > 		/* total channel instances in one umc block */
 > 		#define UMC_V6_1_TOTAL_CHANNEL_NUM	(UMC_V6_1_CHANNEL_INSTANCE_NUM * UMC_V6_1_UMC_INSTANCE_NUM)
 > 		/* UMC regiser per channel offset */
 > 		#define UMC_V6_1_PER_CHANNEL_OFFSET_VG20	0x800
 > 		#define UMC_V6_1_PER_CHANNEL_OFFSET_ARCT	0x400
 >
 > {{< quote >}} [linux/umc_v6_1.h at 49070c4ea3d97b76c5666466efb35dcc42c6c8fd · torvalds/linux](https://github.com/torvalds/linux/blob/49070c4ea3d97b76c5666466efb35dcc42c6c8fd/drivers/gpu/drm/amd/amdgpu/umc_v6_1.h) {{< /quote >}}
-->

一応、HBM(2)メモリは ECC用のチェックビットを内蔵した仕様の製品があることも、考えをわずかでも後押ししていたが、GDDR6メモリであってもアルゴリズムによって ECC機能に対応できる。  
**Radeon PRO W6800** における ECC機能対応は以下のドキュメントで解説されており、各 GDDR6メモリチップの一部容量をチェックビットの格納用に確保する方法を取っている。  
この方法では、使用可能なメモリ容量と実効メモリ帯域にオーバーヘッドが発生するはずだが、ドキュメント中では触れられていない。  
また、シングルビットエラーの修正とマルチビットエラーの検出に対応するとしており、SECDEDコードを用いていると思われる。  

 * <https://www.amd.com/system/files/documents/ecc-explained-radeon-pro-W6000.pdf>

GPU における ECC機能については、Hisa Ando 氏がマイナビニュースでの連載で詳細に解説しており、大変参考にさせていただいた。  

 * [コンピュータアーキテクチャの話(355) GPUにおけるECCの考え方 | TECH+](https://news.mynavi.jp/article/architecture-355/)
 * [コンピュータアーキテクチャの話(323) GPUで用いられるメモリのエラー検出手法とその訂正手法 | TECH+](https://news.mynavi.jp/article/architecture-323/)

Hisa Ando氏は個人サイトでも、前の週に起こったプロセサ関係の話題を紹介しており、そうした話題に興味がある方にはぜひ薦めたい。  

 * [Ando's Microprocessor Information](https://andosprocinfo.web.fc2.com/)
