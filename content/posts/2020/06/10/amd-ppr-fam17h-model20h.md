---
title: "AMD、「PPR for AMD Family 17h Model 20h, Revision A1 Processors」 を公開"
date: 2020-06-10T14:57:15+09:00
draft: false
tags: [ "Raven2", "Dali", "Pollock" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

AMD は 2020/06/05付けで **「Processor Programming Reference (PPR) for AMD Family 17h Model 20h, Revision A1 Processors」** を公開した。  
{{< link >}}[Tech Docs | AMD](https://www.amd.com/en/support/tech-docs?keyword=&page=0){{< /link >}}

`Family 17h Model 20h` は、CPU は Zenアーキテクチャで最大 2-Core/4-Thread、GPU は Vegaアーキテクチャ(gfx909)で最大 3CU、GF 14nmプロセスで製造される *Raven2* ……の別リビジョン、*Dali* と *Pollock* を示す。  
ただ、後述するが *Dali* については Chromebook向けのものが今の所 `Model 20h` とはっきりしている。  

Model 20h-2Fh プロセッサ{{< comple >}}Dali /Pollock{{< /comple >}}は、第8世代APUに基づく、エネルギー効率と性能に優れたラップトップ、タブレットのコンピューティング環境のニーズを満たすために開発されたとしている。  
モバイル向けのソリューションでは TDP 4.5W-15W のプロセッサが利用可能となっている。  

FT5ソケット/パッケージの一部概要も記されており、2-in-1 の小型のフォームファクター向けで、エネルギー効率を重視したクラスのパッケージとしている。  

内部GPUの Device ID に、`0x15D8` 、`0x15DD`、そしてまだ使用が確認されていない `0x15D9` を使うとしており、今後さらなる低電力低コストな Zen APU の登場に期待が持てる。  

## Zen APU 再整理
最初に以前した間違いを訂正すると、*Dali* も x86\_Model は `20h` になる。  
{{< link >}}[AMD Pollock の x86_Model は "0x20" | Coelacanth's Dream](/posts/2020/04/24/amd-fam17h-ft5-extmodel20h/#worry){{< /link >}}

勘違いした原因は単純に自分の知識不足で、以下の **AMD Ryzen 3 3250C** の Geekbench結果にある Identifier の項目を見ると、`Model 32` とあり、これを 16進数に変換すると `20h` になる。  
{{< link >}}[Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/1653443){{< /link >}}
そのため、FP5ソケットで搭載されようと、*Dali* は `20h` であるはずだ。  
ただただ情けなく、申し訳ない。  

しかしもう少しややこしい話があり、すべての *Dali* が `20h` という訳ではない。  
*Raven2* をベースとする、

 * Athlon 300U[^1]
 * Ryzen 3 3200U[^2]
 * Athlon PRO 300GE[^4]

らが x86\_Model `18h` 、つまり *Picasso* と認識されている点は変わりないが、  

[^1]: [Acer Aspire A315-42 - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2441148)
[^2]: [LENOVO 81SS - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2468225)
[^3]: [HP HP Laptop 15s-eq1xxx - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2445614)
[^4]: [LENOVO 11A4CTO1WW - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2396300)

 * Athlon Gold 3150U[^5]
 * Athlon Silver 3050U[^6]
 * Ryzen 3 3250U[^3]

もまた、`18h` となっている。  

[^5]: [HP HP Slim Desktop S01-aF0XXX - Geekbench Browser](https://browser.geekbench.com/v5/cpu/910983)
[^6]: [HP HP All-in-One 22-df0xxx - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2340721)

これは少し不思議な話で、というのもそれらは *Dali* とされていたからだ。  
CES 2020 で正式発表されたとき、脚注にある性能比較の詳細に割り振られた文字列が DAL-1、DAL-3 となっていた。[^8]  
DAL は *Dali* の略称。  

[^8]: [AMD Announces World’s Highest Performance Desktop and Ultrathin Laptop Processors at CES 2020 | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-announces-worlds-highest-performance-desktop-and-ultrathin)

これに関し、Chromium OSへのパッチの中で、一部の Chromebook向けでない *Dali APU* は *Picasso* と同じ CPUIDを使うとコメントされている。[^7]  

[^7]: [soc/amd/picasso: load RV2 VBIOS with rv2 family OPN (I21f317e1) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2033051)

何があった？ GPUの Device ID だけでなく、CPU の判定までもややこしいなんて。  
気にする人しか気にしないかもしれないが、ほんと何があったのか。  

| Family17h SoC | CPU Arch | x86\_Model |
| :-- | :--: | :--: |
| Summit Ridge<br> /Naples | Zen | 1h |
| Pinaccle Ridge<br> /Colfax | Zen+ | 8h |
| Raven Ridge | Zen/+ | 11h |
| Picasso | Zen+ | 18h |
| Raven2<br> /Dali | Zen | 18h |
| Dali(for Chromebook)<br>/Pollock | Zen | 20h |
| Matisse | Zen 2 | 71h |
| Castle Peak<br> /Rome | Zen 2 | 31h |

