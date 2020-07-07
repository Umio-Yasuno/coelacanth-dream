---
title: "AMD、TDP 4.8-6W の AMD 3015e APU の仕様をひっそりと公開"
date: 2020-07-07T20:34:52+09:00
draft: false
tags: [ "Raven2", "Dali", "Pollock" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

AMDの新たな省電力モバイル向けAPU、**AMD 3015e** の仕様が公式サイトより公開されていた。  
TDP は 4.8-6W の範囲にあり、Zen系APUとしては最も低いTDPとなっている。  
{{< link >}}[AMD 3015e | AMD](https://www.amd.com/en/product/10161){{< /link >}}
{{< link >}}[Processor Specifications | AMD](https://www.amd.com/en/products/specifications/processors/){{< /link >}}

## AMD 3015e 仕様
CPU は 2-Core/4-Thread、Base 1.0GHz、Boost 2.3GHz、GPU は 3CU、1.1GHz(1100MHz)、  
メモリは DDR4 2667MHz、デュアルチャネルに対応している。  
似た製品名である **AMD 3020e** と比較すると、製品のセグメントナンバーこそ小さいが、SMTが有効、GPUクロックも 0.1GHz(100MHz) と勝り、ただCPUクロックのみが **AMD 3020e** より抑えられている。  
恐らく TDP 6W時の仕様が表記していると思われ、CPUクロックが抑えられている分GPUクロックを上げる余地があり、それが **AMD 3020e** より 0.1GHz 高いGPUクロックに繋がっているのだろう。  

また、今回初めて気付いたが、製品ラインに **AMD 3000 Series Mobile Processors with Radeon™ Graphics** が追加されており、  
**AMD 3020e** 、**AMD 3015e** はそこに含まれるらしい。  

**AMD 3000 Series Mobile Processors with Radeon™ Graphics** の SKU が大々的に発表されないのは、Intel の 10nm Tremontコア採用のモバイル向けプロセッサ *Jasper Lake* への対抗とするため、まだ今は隠し玉、カウンターとして持っているのではないかという気がする。  
*Jasper Lake* も TDP(PL1) が同じ 6W に対応することがわかっており、搭載製品はぶつかり合うことが予想される[^jsl-pl1]。  
*Jasper Lake* はある程度のプロセッサの情報は出ており、登場が待ち遠しいが、実際の製品情報に関しては中々に焦らされている。  

[^jsl-pl1]: [jasperlake: enable DPTF functionality for dedede (I17b6e4e9) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/41668)

## Dali か Pollock か {#dali-or-pollock}
んで、気になるのは **AMD 3015e** が *Dali* と *Pollock* のどちらに当たるかだが、  
メモリがデュアルチャネルであることから、パッケージは *FP5 BGA* 、よって *Dali* ではないかと思う。  
*Pollock* とそれに使われる *FT5 BGAパッケージ* はシングルチャネル仕様であることが判明している。[^ft5-sc]  

[^ft5-sc]: [soc/amd/picasso: Add Kconfig option for chip footprint (Ia4663d38) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/39867/4)

このあたりはなおも複雑であり、個人的に TDP 4.8W の領域は *Pollock* が担当するものと思っていたため[^plk-4_8W]、**AMD 3015e** の登場には驚かされた。  
*FT5 BGAパッケージ* は省電力というよりは低コストに傾けられているのだろうか？  

[^plk-4_8W]: [mb/google/zork: update power parameters to 4.8w for dalboz (I711d1109) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2135098)

 * [AMD Pollock APU Database | Coelacanth's Dream](/posts/2020/06/14/amd-pollock-apu-database/)  
 * [AMD Dali APU Database | Coelacanth's Dream](/posts/2020/06/24/amd-dali-apu-database/)  

| Product (Fam17h Model20h) | Core/Thread | CPU Base/Boost | GPU CU | GPU Clock | TDP |
| :-- | :--: | :--: | :--: | :--: | :--: |
| Athlon Silver 3050U [^t3050u] | 2/2 | 2.3GHz/3.2GHz | 2 | 1.1GHz | 15(12-25)W |
| Athlon Gold 3150U [^t3150u] | 2/4 | 2.4GHz/3.3GHz | 3 | 1.0GHz | 15(12-25)W |
| Ryzen 3 3250U [^t3250u] | 2/4 | 2.6GHz/3.5GHz | 3 | 1.2GHz | 15(12-25)W |
| AMD 3020e [^t3020e] | 2/2 | 1.2GHz/2.6GHz | 3 | 1.0GHz | 6W |
| *AMD 3015e* | *2/4* | *1.0GHz/2.3GHz* | *3* | *1.1GHz* | *4.8-6W* |
| Athlon Silver 3050e [^t3050e] | 2/4 | (1.4GHz?)/2.8GHz | 3 | 1.0GHz | 6W |
| Athlon Silver 3050C | |
| Athlon Gold 3150C | |
| Ryzen 3 3250C | |
| Ryzen Embedded R1102G [^t1102g] | 2/2 | 1.2GHz/2.6GHz | 3 | 1.0GHz | 6W |
| Ryzen Embedded R1305G [^t1305g] | 2/4 | 1.5GHz/2.8GHz | 3 | 1.0GHz | 8-10W |

[^t3050u]: [AMD Athlon™ Silver 3050U | AMD](https://www.amd.com/en/products/apu/amd-athlon-silver-3050u#product-specs)
[^t3150u]: [AMD Athlon™ Gold 3150U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u#product-specs)
[^t3250u]: [AMD Ryzen™ 3 3250U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u#product-specs)
[^t3050e]: [AMD Athlon™ Silver 3050e | AMD](https://www.amd.com/en/product/9896)
[^t3020e]: [AMD 3020e | AMD](https://www.amd.com/en/products/apu/amd-3020e#product-specs)
[^t1102g]: [AMD Ryzen™ Embedded R1102G with Radeon™ Vega 3 Graphics | AMD](https://www.amd.com/en/product/9226)
[^t1305g]: [AMD Ryzen™ Embedded R1305G with Radeon™ Vega 3 Graphics | AMD](https://www.amd.com/en/product/9221)

