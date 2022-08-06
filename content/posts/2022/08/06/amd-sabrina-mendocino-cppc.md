---
title: "AMD Sabrina/Mendocino APU は実際には CPPC をサポート"
date: 2022-08-06T01:09:55+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Sabrina", "Mendocino", "GC_10_3_7", "Coreboot" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

CPU に *Zen 2 アーキテクチャ* 4-Core/8-Thread、GPU には *RDNA 2/GFX10.3, GC 10.3.7 (gfx1037)* を採用する *AMD Sabrina/Mendocino APU* 。  
Felix Held 氏により Coreboot での対応が進められた当初、*Sabrina/Mendocino APU* は CPU の電源管理機能 CPPC/2 (Collaborative Processor Performance Control /2) をサポートしないとされていたが、その後に実際にはサポートしていることが判明した。  
*Sabrina/Mendocino APU* において CPPC を有効化するパッチが Felix Held 氏によって投稿されている。  

 * [soc/amd/sabrina: drop CPPC code (I71a1b071) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61096/6/)
 * [soc/amd/sabrina: enable CPPC feature (I1c059653) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/66401/1/)

CPPC2 では従来よりクロックの選択が高速化されており、CPPC2 のサポートは性能、電力効率両方の向上が期待できるため、あえて外す理由が不明だったが、結論としてはドキュメントの抜けから来たものだった。  
あるいは初期リビジョンのシリコンでは本当に CPPC/2 をサポートしていなかったのかもしれない。  

*Sabrina/Mendocino APU* を搭載したノートPC は 2022Q4 より OEMパートナーから発売される予定。  

{{< ref >}}
 * [AMD Reports Second Quarter 2022 Financial Results :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1084/amd-reports-second-quarter-2022-financial-results)
 * [[PATCH v2] x86/cpu/amd: fix the highest perf query for new AMD processors - Perry Yuan](https://lore.kernel.org/linux-pm/20220614085255.256600-1-Perry.Yuan@amd.com/)
 * [AMD Sabrina SoC に向けたさらなるパッチ | Coelacanth's Dream](/posts/2022/01/14/amd-sabrina-soc-more-patch/)
{{< /ref >}}
