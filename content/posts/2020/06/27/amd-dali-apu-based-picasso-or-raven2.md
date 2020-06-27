---
title: "AMD Picassoベースの Dali APU が存在するらしい？"
date: 2020-06-27T10:41:30+09:00
draft: false
tags: [ "Dali", "Picasso", "Raven2" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

Zen APUにまつわるややこしい仕様、特に *Picasso* と *Raven2* にはこれまでに何回か触れてきたけど、自分がそれを把握しているという自信が揺らいできたし、そもそも正しく把握できている人がいるのか不安になってきた。  
それでも、*Dali* については一応以下を参照してほしい。  
{{< link >}}[AMD Dali APU Database | Coelacanth's Dream](/posts/2020/06/24/amd-dali-apu-database/){{< /link >}}

きっかけは Coreboot へのあるパッチとそれに付随するメッセージだ。  
{{< link >}}[soc/amd/picasso/soc_util: rework reduced I/O chip detection (I9eb57595) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/42833){{< /link >}}
{{< link >}}[soc/amd/picasso/soc_util: add comment on the silicon and soc types (I71704ab2) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/42838){{< /link >}}

{{< pindex >}}

 * [Picasso シリコンをベースにした Dali ?](#dali-based-pco)
 * [疑問と考察](#question)
   * [Dali (RV2/PCO) ≠ Dali (PCO/PCO) ](#rv2-pco-ne-pco-pco)
   * [OPN で異なる？](#opn-diff)
   * [問題](#problem)

{{< /pindex >}}

## Picasso シリコンをベースにした Dali ? {#dali-based-pco}
パッチの中身としては、上が、*RV2 (Raven2)* シリコンをベースとする *Dali* と *Pollock* は *Raven /Picasso* から外部 I/O の規模が減らされているため、それを検出するコードとなり、  
下がそのパッチの補足メッセージをさらに補足して、ソースコード内にコメントとして記述するというもの。  

<!--

 > Both Dali and Pollock chips have less PCIe, USB3 and Displayport
 > connectivity. While Dali can either be fused-down PCO or RV2 silicon,
 > Pollock is always RV2 silicon.
 >
 > 引用元: [soc/amd/picasso/soc_util: rework reduced I/O chip detection (I9eb57595) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/42833)

-->

で、問題がメッセージの内容で、  
まず *RV2 (Raven2)* シリコンは *RV (Raven) / PCO (Picasso)* から PCIE、USB3、Displayport が減らされている、と述べており、これは過去これまでの情報と一致する。  

 >       /*
 >       * The Zen/Zen+ based APUs can be RV (sometimes called RV1), PCO or RV2 silicon. RV2 has less
 >       * PCIe, USB3 and Displayport connectivity than RV(1) or PCO. A Picasso SoC is always PCO
 >       * silicon, a Dali SoC can either be RV2 or fused-down PCO silicon that has the same
 >       * connectivity as the RV2 one and Pollock is always RV2 silicon. Picasso and Dali are in a FP5
 >       * package while Pollock is in the smaller FT5 package.
 >       */
 >
 > 引用元: [soc/amd/picasso/soc_util: add comment on the silicon and soc types (I71704ab2) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/42838)

そして *Picasso* は常に *PCO* シリコンをベースとする。ここに疑問は特に無い。  
だがその後が一番問題で、*Dali* は *RV2* シリコンか、外部 I/O の規模を *RV2* 同等までに減らした *PCO* シリコンになるとしている。  

問題と考えるのは、自分はこれまで、*Dali* はすべて *RV2* シリコンをベースとするものの、一部は CPUID を *Picasso* と共用し、`x86_Model` も *Picasso* と同じ `18h` になるため、ソフトウェアからは *Picasso* と認識される、と考えていたが、  
それだけでなく、ベースとするシリコン自体が *PCO* シリコンであるものもある、と述べられているからだ。  
この場合 *Dali* には最大 3種類あることとなり、`シリコン/CPUID` の形で記述すると、`RV2/RV2` 、`RV2/PCO` 、`PCO/PCO` になる。  

*Pollock* は `RV2` シリコンのみをベースとし、パッケージには FT5パッケージを使うこと、  
*Picasso* と *Dali* は FP5パッケージを使う、ということに問題、疑問は特に浮かばない。  

## 疑問と考察 {#question}

まず、*Picasso* と *Raven2* の違いには、CPU、GPU、I/O(PCIe/USB/Display) の規模があり。*Raven2* の方が小さい。  
次に製造プロセスの違いがあり、*Picasso* は 12nmプロセス、*Raven2* は 14nmプロセスとなる。  
そして、GPUのアーキテクチャが微妙に異なり、*Raven2* GPU の *gfx909* は、*Raven /Picasso* GPU の *gfx902* に存在したバグを修正したものとなる。  

| | Picasso | Raven2 |
| :-- | :--: | :--: |
| **CPU** | |
| &emsp;Max CPU Core/Thread | 4/8 | 2/4 |
| &emsp;Max CPU L3cache | 4MB | 4MB |
| **GPU** | *gfx902* | *gfx909* |
| &emsp;Max GPU CU | 11CU | 3CU |
| &emsp;Max GPU RB | 2<br>(8ROP) | 1<br>(4ROP) |
| &emsp;Max GPU L2cache | 1024KB | 512KB |
| Process | 12nm | 14nm |

### Dali (RV2/PCO) ≠ Dali (PCO/PCO) ？ {#rv2-pco-ne-pco-pco}
表面上 *Dali* とされるが、内部で CPUID と x86\_Model は *Picasso* となるプロセッサには、**Athlon Silver 3050U** 、**Athlon Gold 3150U** 、**Ryzen 3 3250U** が確認されているが、  
それらは AMD公式の仕様ページに `CMOS: 14nm` とあるため[^1]、*RV2* シリコンであると思われる。  

[^1]: [AMD Athlon™ Gold 3150U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u)

そのため、CPUID、x86\_Model が *Picasso* だからといって、シリコンも *PCO* になるとは限らないと言える。  

### OPN で異なる？ {#opn-diff}
考えられるのは OPN(Ordering Part Number) でベースとなるシリコンが異なる可能性だ。  

最近そのコスパから話題となった **Ryzen 5 1600 (AF)** は、Ryzen 1000シリーズ(Summit Ridge) でありながら、シリコンは Ryzen 2000シリーズ(Pinnacle Ridge) と同一のものだった。  
それと同様に、今後 *Dali* APU製品を一時的に *PCO* シリコンをベースとして提供する予定がある、もしくは既に一部で行なっているのかもしれない。  

### 問題 {#problem}
そうした場合に考えられる問題がいくつかある。  

**Ryzen 5 1600 (AF)** は、*Pinnacle Ridge* の CPU、I/O 規模が *Summit Ridge* と同じであったから特に問題は発生しなかった。  
対応するマイクロコードは異なるが、*Pinnacle Ridge* のみをピンポイントでサポートしていない UEFI(BIOS) というのは考えにくいし、実際あったという話も聞かない。  
しかし、*PCO* シリコンと *RV2* シリコンでは CPU、GPU、I/O の規模が違う。  
結果、判定部分のコードがさらに複雑となっている。  
コードを読むと、`is_mystery_silicon` 関数を呼び、その先でシリコンのタイプを取得して判定を行なっている。  
関数名からどこか投げやりなものを感じるのは自分だけだろうか。  

GPUアーキテクチャの違いに関しては、バグ修正くらいであるため、問題ないと考えているのかもしれない。  
*gfx909* も LLVM のバージョンが古い場合は、 *gfx902* として判定されるようになっていた。  

*PCO* シリコンは *RV2* シリコンよりシリコンダイのサイズが大きいため、同様のコア、クロック設定で動作させた場合、*PCO* シリコンの方が冷却が容易となる、  
といった利点のようなものが存在しなくもない。  

ただ、最も懸念されるのは、OPNで違うとして、その情報が早くに公開されるかどうかだ。  
わずかでも、実質的に同じでも、違いはやはり違いであり、区別できた方が良い。  

コードネームの元ネタである サルバドール・ダリ は数々のエピソード(奇行)を残しているが、もしかしたらそれが由来になったんじゃないか、なんて考えに及んでいたりする。  
さすがにこれ以上複雑にならないことを願う。  
*Dali* というコードネームから簡潔に説明できないのでは、コードネームの意義が揺らいでしまう。  
