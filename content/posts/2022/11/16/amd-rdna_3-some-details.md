---
title: "RDNA 3 アーキテクチャのさらなる詳細"
date: 2022-11-16T09:24:27+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GFX11", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

2022/11/14 23:00 付で *RDNA 3 アーキテクチャ* のさらなる詳細が解禁された (らしく)、各種メディアが解説記事を公開している。  
AMD がメディア向け資料を一般向けに公開していないため、それら各種メディアの記事に掲載されているスライド資料を参考に *RDNA 3 アーキテクチャ* で疑問だった部分をいくつか個人的に整理してみる。  

PC Watch の記事は、*GCN アーキテクチャ* を VLIW (Very long instruction word) としていたり、  
*GCN* における L1キャッシュが CU あたり 64KB になっていたり (正確には 16KB)、  
*Radeon RX Vega (Vega10)* の ShaderArray (SA) 数を 8基としていたり (正確には 4基、ShaderEngine (SE) 4基、SE あたり SA 1基という構成)、*Navi21* の SA 数を 4基としていたり (正確には 8基、SE 4基、SE あたり SA 2基)、[^se-sa]  
CU 内の L0キャッシュを内部レジスタのキャッシュと表現していたり (レジスタファイルとキャッシュメモリの関係を説明するのに適切ではないように思う)、  
Wave を、スレッドの塊といったよく説明に使われる表現ではなく、Wave32 を 32-bits 単位の ALU 定義、Wave64 を 64-bits 単位の ALU 定義としていたり、  
発行 (issue) ではなく実行 (execution) となっていたり……怪しいと思う記述がいくつか見られ、質問をメールで送ったのだが、執筆時点で返信は来ていない。  
そうしたことを気にするのは自分くらいしかいないのかもしれない。  

{{< ins >}}
その後 PC Watch より返信が来て、メールで最初に質問した部分に関しては一部修正された。  
上でぐだぐだと書いた記事への指摘には、メール送信後に気付いた部分も含まれている。  
{{< /ins >}}

 * [AMD Reveals More Details Around The Radeon RX 7900 Series / RDNA3 - Phoronix](https://www.phoronix.com/review/amd-radeon-rx7900)
 * [【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 - PC Watch](https://pc.watch.impress.co.jp/docs/column/ubiq/1455417.html)

[^se-sa]: [pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp)

## CU, SIMD Unit {#cu-simd}
最初の発表では曖昧だったが、今回の情報解禁でより詳細な Compute Unit のブロック図が示された。  
WGP (Workgroup Processor) という言葉は使われなくなったが、CU 2基で LDS (Local Data Share)、命令キャッシュ、スカラデータキャッシュを共有する構成は継続している。  
LDS のサイズはスライド中にないが、従来と同じ 128KB (2x64KB) と思われる。[^lds]  

[^lds]: [RDNA 3 アーキテクチャの CU の若干の考察 | Coelacanth's Dream](/posts/2022/11/09/rdna_3-cu-consider/)

SIMD Unit については、1xSIMD64 と 2xSIMD32 の 2つのモードがあり、Dual issue Wave32 時は SIMD32 (Float or Int) と SIMD32 (Float) の組み合わせで動作し、それぞれで別の命令を発行することができる。  
この説明は `VOPD (Dual issue wave32)` 命令が基本 FP32 フォーマットにのみ対応することと一致する。  
1xSIMD64 では Wave64 を 1サイクルで発行することができる。  
おそらく `VOPD` 命令以外を Wave32 で発行すると SIMD Unit の使用率は半分に留まると思われるが、コンパイラは `VOPD` 命令以外は Wave64 を選択すれば問題ないと思われる。  

LLVM へのパッチでは *RDNA 3 アーキテクチャ* の FP64 (DP) : FP32 (FP) の演算性能レートは 32:1 になるとしていたが、スライド中の図では SIMD64 (2xSIMD32) に対して **DPFP(1)** となっており、64:1 となっている可能性が出てきた。[^dpfp]  
また、超越関数、三角関数や平方根の逆数等を計算するための Transcendental Unit は SIMD8 のまま維持されている。  

[^dpfp]: [GFX11 では FP64 演算性能が FP32 の 1/32 に | Coelacanth's Dream](/posts/2022/06/18/gfx11-dpfp-rate/)

 * [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (4/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/004_o.jpg.html)
 * [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (5/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/005_o.jpg.html)
 * [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (9/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/009_o.jpg.html)

### WMMA (Wave Matrix Multiply-accumulate) {#wmma}
行列演算命令、`WMMA (Wave Matrix Multiply-accumulate)` 命令については、1個の `WMMA` 命令から内部ではドット積命令用の 64x Dot2 ALU (FP16, BF16, Int8), 64x Dot4 ALU (Int4) に対して分解した命令を連続で発行する形式となっている。  
BF16 フォーマットを用いた計算処理において *RDNA 3 アーキテクチャ* はピークスループット性能が *RDNA 2* の 2.7倍、という FP32 フォーマットと同じ比率の性能向上だったことはこの `WMMA` 命令の形式が関係していると思われる。  
`WMMA` 命令によってピークスループット性能が飛躍的に向上したりはしないが、命令数を減らすことができ命令キャッシュを圧迫せず、またコンパイラによるレジスタ割り当てもやりやすくなると思われ、実性能では向上が見込める。  

*RDNA 3 アーキテクチャ* では Int4, unsigned Int4 の Dot8 命令にも対応しているはずだが、スライドでは触れられていない。内部的には 64x Dot4 ユニットを活用する形式とかなのだろうか。[^dot8]  

[^dot8]: [GFX11 のサポートに向けた LLVM へのさらなるパッチ ―― True16Bit, Dot8, Dual issue wave32 | Coelacanth's Dream](/posts/2022/05/10/llvm-gfx11-dual-issue/#dot8)

 * [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (9/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/009_o.jpg.html)
 * [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (10/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/010_o.jpg.html)

## Infinity Cache {#ic}
*RDNA 3 アーキテクチャ* のキャッシュ階層を示したスライドに帯域も記載されており、L2-L3 (Infinity Cache) 間が 2304 Byte/Clock となっている。  
スライドは **RX 7900XTX** を例に取っており、メモリ構成は GDDR6 384-bits (24ch x 16-bits) であることから、*RDNA 3 アーキテクチャ* の L3キャッシュ (Infinity Cache) のキャッシュラインサイズは 96Byte と考えられる。  
*RDNA 2 アーキテクチャ* では 64Byte であったから、*RDNA 3 アーキテクチャ* ではクロックあたりの L3キャッシュ帯域が 1.5倍に増加している。  

他キャッシュ階層に合わせて 128Byte とせずに 96Byte への拡大に留めたのは、キャッシュの利用効率やチップレットデザインが関係しているのだろうか。  
スライドには無いが、*RDNA 3 アーキテクチャ* では命令キャッシュのキャッシュラインサイズも 64Byte から 128Byte に拡大されている。[^cacheline]  

[^cacheline]: [RadeonSI ドライバーで GFX11 のサポートが進められる | Coelacanth's Dream](/posts/2022/05/05/radeonsi-gfx11/#inst-cache-line-size)

| Cache Line Size               | L1i  | L1k   | L1d   | GL1   | GL2  | L3  |
| :--------------               | :-:  | :-:   | :-:   | :-:   | :-:  | :-: |
| GFX9/Vega (without Aldebaran) | 32B  | 32B   | 64B   | -     | 64B  | -   |
| GFX10/RDNA 1, GFX10.3/RDNA 2  | 64B  | 64B   | 128B  | 128B  | 128B | 64B (RDNA 2) |
| GFX11                         | 128B | 128B? | 128B? | 128B? | 128B | 96B |

 * [[画像] 【笠原一輝のユビキタス情報局】新演算器とチップレットで電力効率と性能を引き上げたRadeon RX 7000の詳細 (18/31) - PC Watch](https://pc.watch.impress.co.jp/img/pcw/docs/1455/417/html/018_o.jpg.html)

{{< ref >}}
 * <https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf>
 * <https://hc33.hotchips.org/>
 * [コンピュータアーキテクチャの話(345) Kepler GPUの構成 | TECH+（テックプラス）](https://news.mynavi.jp/techplus/article/architecture-345/)
{{< /ref >}}
