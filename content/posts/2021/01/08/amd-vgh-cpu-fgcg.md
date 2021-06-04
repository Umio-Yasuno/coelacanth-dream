---
title: "VanGogh APU がサポートする 「Fine Grain Clock Gating」 は CPU にも適用　―― 最大 4コアか"
date: 2021-01-08T19:48:47+09:00
draft: false
tags: [ "VanGogh", ]
# keywords: [ "", ]
categories: [ "AMD", "APU", "Hardware" ]
noindex: false
# summary: ""
toc: false
---

以前、Linux Kernel (amd-gfx) に *VanGogh APU* の機能として `FGCG (Fine Grain Clock Gating)` をサポートするパッチが投稿されたが、  
{{< link >}} [Linux Kernel に "Fine Grain Clock Gating" のサポートが追加される | Coelacanth's Dream](http://localhost:1313/posts/2020/11/04/amd-linux-kernel-fgcg-support/) {{< /link >}}
それを CPU にも適用するパッチが新たに投稿された。  
省電力に関係する部分とはいえ、GPU ドライバーに CPU のサポートも追加される、興味深いものとなっている。  

 * [[PATCH 0/7] implement the processor fine grain feature for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-January/057994.html)

`Fine Grain Clock Gating` の詳細はまだ語られていないが、名前から、単純により強化されたクロックゲーティングと捉えている。  
プロセッサには細かい時間単位で見ると使われていないユニットが多くあり、それらにもクロックを入力しているとダイナミック電力を消費してしまう。その時は処理に関係していないユニットであるから、それは無駄な消費電力と言える。  
使われていないユニットへのクロック供給を止め、無駄な消費電力を減らす技術がクロックゲーティングとなる。  
APU である *VanGogh* は SoC らしく、CPU、GPU、マルチメディアエンジン、ディスプレイエンジン、各種 I/O……と多くのユニットを搭載しており、クロックゲーティングの強化は効果的と思われる。  

## VanGogh は最大 4コア? {#vgh-4core}

GPUクロックのように、CPUクロックを設定された範囲内で調節する機能もサポートされている。  
デフォルトの設定では、最小 CPUクロック 1.4GHz (1400MHz)、最大 CPUクロック 3.5GHz (3500MHz) となっており、あくまでもコード内に直接記述された (ハードコード) 設定で、UEFI やファームウェアから上書きされるだろうが、*VanGogh* の特徴を知る上で参考にはなるだろう。  

APU のクロック調整は、ドライバーからは *Vega20* 以降最小/最高クロックの 2つを設定するのみとなっている。  
ユーザーがクロックのポイントを細かく調整することは難しくなったが、それだけハードウェア側の電力管理機能が優秀になったとも考えられる。  

また、GPUクロックと同じ要領で自分好みの CPUクロックを設定できるのは嬉しい話だ。  

 >        @@ -1351,6 +1422,11 @@ static int vangogh_set_fine_grain_gfx_freq_parameters(struct smu_context *smu)
 >         	smu->gfx_actual_hard_min_freq = 0;
 >         	smu->gfx_actual_soft_max_freq = 0;
 >         
 >        +	smu->cpu_default_hard_min_freq = 1400;
 >        +	smu->cpu_default_soft_max_freq = 3500;
 >        +	smu->cpu_actual_hard_min_freq = 0;
 >        +	smu->cpu_actual_soft_max_freq = 0;
 >        +
 >         	return 0;
 >         }
 >
 > {{< quote >}} [[PATCH 7/7] drm/amd/pm: implement processor fine grain feature for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058001.html) {{< /quote >}}

そして、*VanGogh* は最大 4コアであることが窺える記述が、コードとパッチに添えられたコメントにあった。  

 >        echo "p core_id level value" > pp_od_clk_voltage
 >        
 >        1. "p" - set the cclk (processor) frequency
 >        2. "core_id" - 0/1/2/3, represents which cpu core you want to select
 >        2. "level" - 0 or 1, "0" represents the min value,  "1" represents the
 >           max value
 >        3. "value" - the target value of cclk frequency, it should be limited in
 >           the safe range
 >
 > {{< quote >}} [[PATCH 7/7] drm/amd/pm: implement processor fine grain feature for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058001.html) {{< /quote >}}

`boot_cpu_data.x86_max_cores` がコア数のカウントに使われていることから、スレッド数ではなく物理コア数を指していると思われる。  
SMT の分のスレッドは、CPUコアのように見せてもクロックドメイン等は 2スレッドで物理コアものを共用するというのもある。  

現行の *Renoir APU* 、次世代 *Cezanne APU* の CPU コア/スレッド数は 8-Core/16-Thread であり、*VanGogh* はその半分となる 4-Core の規模となる。  
このことから *VanGogh* は、省電力低コストを特徴とする [Raven2 APU](/tags/raven2) ([Dali](/tags/dali), [Pollock](/tags/pollock)) の後継として開発されているのではないかと考えられる。  

*VanGogh* では LPDDR5メモリがサポートされ、GPU部は *RDNA 2 アーキテクチャ* となる。  
省電力を目的としながら、多くの新機能、新アーキテクチャを搭載した APU となり、性能向上以外にも魅力的な部分が見れる。  
パッチから正式発表日は探れないが、その分 *VanGogh* の登場は個人的にとても楽しみにしている。  
{{< link >}} [VanGogh APU でも NGG/プリミティブシェーダーがデフォルトで有効に | Coelacanth's Dream](/posts/2020/11/26/vgh-apu-ngg-default/) {{< /link >}}

{{< ref >}}

 * [コンピュータアーキテクチャの話(219) ダイナミック電力の低減手法(2) | マイナビニュース](https://news.mynavi.jp/article/architecture-219/)

{{< /ref >}}
