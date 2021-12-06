---
title: "【雑記】CPUID の Family と Model、表示する一部ソフトウェアの問題点"
date: 2021-03-01T16:47:01+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Diary", "CPU" ]
noindex: false
# summary: ""
toc: false
---

CPU の判定とかに使われる Family、Model についての話。  

{{< pindex >}}
 * [Base と Extended](#base-and-ext)
 * [一部ソフトウェアの問題点](#oroblem)
{{< /pindex >}}

## Base と Extended {#base-and-ext}

CPU の Family、Model、Stepping 等の情報は CPUID から読み取ることができ、その CPUID のバイナリは以下のフォーマットになっている。  

| CPUID (bit) | 31-28 | 27-20 | 19-16 | 15-14 | 13-12 | 11-8 | 7-4 | 3-0 |
| :--   | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| EAX   | Reserved  | Extended<br>Family   | Extended<br>Model    | Reserved  | Processor<br>Type | Base<br>Family | Base<br>Model | Stepping<br>ID |

主に CPU の判定に使われる Family、Model は、CPUID では Family/Model と Extended.Family/Model に分けられており、それらが組み合わされて表現される。  
AMD の PRR (Processor Programming Reference) では、CPUID のを Base.Family/Model と Extended.Family/Model とし、それらが組み合わせられて Family/Model が表現されると記述しており、そちらのが分かりやすいため、ここではその方式を採用して解説する。  

組み合わせ方は Family と Model で若干異なっており、Family は `(Extended.Family + Base.Family)` で表現されるが、Model は `((Extended.Model << 4) + (Base.Model))` と、Extended.Model を 16進数 1桁分 (4-bit) を拡張、それから加算して表現される。  

以下は Coreboot 内のコードで、CPUID から Processor Type、Family、Model、Stepping を読み取って出力する部分となる。  

 >        	printf("CPU: ID 0x%x, Processor Type 0x%x, Family 0x%x, Model 0x%x, Stepping 0x%x\n",
 >        			id, (id >> 12) & 0x3, ((id >> 8) & 0xf) + ((id >> 20) & 0xff),
 >        			((id >> 12) & 0xf0) + ((id >> 4) & 0xf), (id & 0xf));
 >
 > {{< quote >}} [coreboot/inteltool.c at 1e678169616b959921c38a2f25ca23b7f3e4cc77 · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/1e678169616b959921c38a2f25ca23b7f3e4cc77/util/inteltool/inteltool.c#L554) {{< /quote >}}

処理内容としては、まず `0x90670 (Alder Lake-S)` のような CPUID が渡され、それをビット単位の右シフト演算 (`id >> n`) を行い、次にビット単位の論理積を (`(id >> n) & 0xf`) を求めることで、指定の桁の値を得る、というものになっている。16進数では桁あたり 4-bit で表現されるから、`id >> 4` で右に 1桁ずらすことができる。  
例に出した *Alder Lake-S* の CPUID では、上 2桁、Reserved と Extended.Family の上位 4-bit が省略されている。  

Processor Type を `(id >> 12) & 0x3` としているのは、上の CPUID のフォーマットからも分かるように、Processor Type に定義されているのは 4種 (2-bit) までだからとなる。  
Extended.Family は 8-bit、他は基本 4-bit となっているが、Processor Type の左 2-bit は 予約領域 (Reserved) に割り当てられており、16進数で表現した時にずれることはない。  

Processor Type の各定義は以下のようになっている。[^processor-type]  

| Type | Encoding |
| :-- | :--: |
| Original OEM Processor | 00B |
| Intel OverDrive Processor | 01B |
| Dual processor (not applicable to Intel486 processors) | 10B |
| Intel reserved | 11B |

[^processor-type]: [Intel® 64 and IA-32 Architectures Developer's Manual: Vol. 2A](https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-vol-2a-manual.html) (Page 307)

## 一部ソフトウェアの問題点 {#oroblem}

CPU の Family、Model は各種システム情報を表示するソフトウェアで使われることもあるが、Windows OS で主に使われている [CPU-Z](https://www.cpuid.com/softwares/cpu-z.html) 等では正しく使われていない。  

CPU-Z では CPU の項目に、[Family] と [Ext.Family]、[Model] と [Ext.Model] を表示しているが、それらの中身は [Family] が Base.Family、[Ext.Family] が本当の Family (Base + Extended) となっており、値の意味と一致しない。Model についてもそのようになっている。  
Intel の資料では Base.Family はただ Family と記述されているが、それは実際には組み合わせた値と区別されるべきものである。  
他のソフトウェアでも同じ仕様になっているものがあり、Linux、FreeBSD で動作する、CPU-Z に似たソフトウェア [CPU-X](https://github.com/X0rg/CPU-X) がそうだ。  

{{< figure src="/image/2021/03/01/cpu-x-fam-mod.webp" title="CPU-X CPU Tab" >}}

ただこの問題は CPU-X 自体よりも、CPU-X が用いているライブラリ [libcpuid](https://github.com/anrieff/libcpuid) に起因している。  
`ext_family` に `Base.Family + Extended.Family` を、`ext_model` に `Base.Model + Extended.Model` を代入しており、以下がその該当部分となる。  
Github で確認できる限りでは、v0.2.1 (2015-10-10) からこの仕様となっており、今も続いている。  
libcpuid 最初のリリースは 2008-10-15 と ChangeLog にあるため、もっと前から問題が存在している可能性がある。  

 >        			data->ext_family = data->family + xfamily;
 >        		data->ext_model = data->model + (xmodel << 4);
 >
 > {{< quote >}} [libcpuid/cpuid_main.c at 52c5f505cff57266b32aa8eb95eb7c7fc47db94b · anrieff/libcpuid](https://github.com/anrieff/libcpuid/blob/52c5f505cff57266b32aa8eb95eb7c7fc47db94b/libcpuid/cpuid_main.c#L323) {{< /quote >}}


本来であれば、項目[Family] に (Base + Ext) が使われ、[Ext.Family] に Extended.Family が表示されるべきだと考えている。  
というより、言ってしまえば Family と Model それぞれに項目は 2つも必要ない、本当の Family/Model (Base + Extended) だけで十分だ。  
Linux Kernel では以下のように本当の Family/Model、それと Stepping だけを表示しているし、コンパイラ等も、細かい Base/Extended.Family/Model を使うことはなく、本当の Family/Model で CPU の判定を行っている。  

 >      [    0.170939] smpboot: CPU0: AMD Ryzen 5 2600 Six-Core Processor (family: 0x17, model: 0x8, stepping: 0x2)

わざわざ Base/Extended.Family/Model を個別に表示する必要性はかなり薄いし、ややこしくなるだけだ。  
システム情報を表示する一部の {{< comple >}} それに人気でもある {{< /comple >}} ソフトウェアは、正しく値を用いないために、さも丁寧で親切な顔をしながら問題をもっと厄介なものにしている。  
さらにひどいことに、CPU-Z は 16進数で表示しながら、接頭辞に `0x` を付けていない。CPU-X は付けているから、まだマシではある。  
接頭辞を正しく付けないことの何がひどいかと言えば、まだ正式リリースされていない CPU が登場する場として [Geekbench](https://browser.geekbench.com/) があるが、そこでは CPU の Family、Model、Stepping が 10進数で表示されている。  
だから、CPU-Z と Geekbench の情報を見比べると、同じような表記をしているが、Family は一致しても、Model はどれとも一致しない、ということが普通に起こり得る。  
近年の Intel CPU は `Family: 0x6 (6)` であるため、16進数でも 10進数でもそれは一致する。  

Family/Model の意味と表記方法の違いを正しく理解し、かつ既に OSS へのパッチ等からある CPU の Family/Model が知られていれば、無駄に考えてる振りしてキャッシュ構成からしてどうだの言ったり、あるいは根拠も無く推測を言ったり、または怪しんだりすることなく、  
すぐに判断することができるが、一部の誤った使い方によってそれが難しくされてしまっている。  

自分が知っている限りで、CPUID から得られる情報を最も誠実に表示しているのは、Linux 上で動作する [cpuid](http://www.etallen.com/cpuid.html) コマンドだ。  
以下は実行例の一部だが、Base/Extended をどちらも正しく表示し、本当の Family/Model についても、それが組み合わされたものと分かるようにしてある。  
さらには 16進数と 10進数の両方を表示しているため、他のソフトウェア、ベンチマークで表示される値と比較してもどれかは一致するし、相違点も分かりやすい。  

 >        CPU 0:
 >           vendor_id = "AuthenticAMD"
 >           version information (1/eax):
 >              processor type  = primary processor (0)
 >              family          = 0xf (15)
 >              model           = 0x8 (8)
 >              stepping id     = 0x2 (2)
 >              extended family = 0x8 (8)
 >              extended model  = 0x0 (0)
 >              (family synth)  = 0x17 (23)
 >              (model synth)   = 0x8 (8)
 >              (simple synth)  = AMD Ryzen (Pinnacle Ridge PiR-B2) [Zen+], 12nm

CPUID と Family/Model は 1種の記号であり、確かな意味を持っている。  
しかしそれを正しく認識するには、表記の違いや、そのソフトウェアの表示が信頼できるかどうかを考える必要がある。  

このことに気付いたのは最近ではあるが、まだ libcpuid のレポジトリに issue は送っていない。  
CPU-X の方には issue と Pull Request を送ったが、libcpuid の問題だとしてマージは見送られた。  
以下はそれを適用した場合の表示だが、Family と Model だけとし、Ext. の項目は無駄なので省き、空いたスペースを活用して項目の幅を拡張、10進数表示を加えている。  

{{< figure src="/image/2021/03/01/cpu-x-fix-fam-mod.webp" >}}

libcpuid の方にもさっさと送るべき、最低でも報告はするべきなのだろうが、それをしない理由を端的に言えば気力の問題だ。  
上で述べたようにこれは長年放置されている問題で、それは多くの人が、ユーザー、開発者を含めて関心を持っていないことによるものと考えられる。  
また、そういったソフトウェアでは値をほとんどそのまま表示するため、バグや不具合として検出されないことも原因としてある。この前は、MB でメモリ容量を表示しているが、内部では ( KiB[2^10] / 10^3 ) で計算されていることに気付き {{< comple >}} これは MB [10^6] とも MiB [2^20] とも異なるずれた値になる {{< /comple >}}、単位を MiB に変更、内部処理も ( KiB[2^10] / 2^10 ) に修正したりもした。その部分のコードは 5年近く前からあった。libcpuid は最低でも 5年、それ以上前から存在して、放置され続けてきた可能性がある問題だ。  
1つのソフトウェアの問題を修正するだけならまだ簡単だが、ライブラリとなると影響範囲も大きくなる。  
値を間違った解釈で使ってはいるが、それは長い間受け容れられてきた解釈でありもはや仕様だ。  
だから修正がちゃんと望む形で適用されるか、不安に思っている。  
そして、やり取りは全て英語で行わなければならないということが壁として存在する。  
気力の問題と言ったのはそういう訳だ。  


