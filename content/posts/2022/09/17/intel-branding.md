---
title: "Intel Pentium, Celeron ブランドが簡素化され \"Intel\" ブランドに ―― Alder Lake-N から導入か"
date: 2022-09-17T11:57:44+09:00
draft: false
categories: [ "Intel", "Hardware", "CPU" ]
tags: [ "Alder_Lake", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は 2022/09/16 付で、*Intel Pentium, Celeron* ブランドが簡素化され、*Intel Processor* ブランドに置き換えられる。  
この新しいブランディングは 2023年のノートPC向け製品から適用される予定。  

 * [Intel Introduces New Intel Processor for Upcoming Essential Segment PCs](https://www.intel.com/content/www/us/en/newsroom/news/welcome-the-new-intel-processor.html)

最近の *Intel Pentium, Celeron* SKU を見ると、世代が同じであっても *i9/i7/i5/i3* ブランドとモデルナンバーの規則は一致せず、また *Pentium* には *Pentium Silver* と *Pentium Gold* の 2種類がある。  
*Jasper Lake* 世代では、*Pentium Silver N6000/N6005* と *Celeron N4500/N4505/N5100/N5105* が SKU にあり、モデルナンバーの上位だけを見て世代を判断するということが難しくなっているように感じた。  
そういったことから、長年続いた *Intel Pentium, Celeron* ブランドではあるが、ユーザーにとってわかりやすくするためには一度ブランドとモデルナンバーをリセットする必要があったのだと思われる。  

まだ正式発表はされていない Intel CPU に *Alder Lake-N* がいるが、intel-gfx-ci の bootlog から **Intel(R) N100 (4-Core/4-Thread?)** と **Intel(R) N200 (4-Core/4-Thread?)** というプロセッサ名が確認できる。  

 > 		<6>[    0.028867] smpboot: Allowing 4 CPUs, 0 hotplug CPUs
 >      ~~~
 > 		<6>[    0.324882] smpboot: CPU0: Intel(R) N100 (family: 0x6, model: 0xbe, stepping: 0x0)
 >
 > {{< quote >}} <https://intel-gfx-ci.01.org/tree/drm-tip/Patchwork_107213v1/bat-adln-1/boot0.txt> {{< /quote >}}
 >
 > 		<6>[    0.029160] smpboot: Allowing 4 CPUs, 0 hotplug CPUs
 >      ~~~
 > 		<6>[    0.319625] smpboot: CPU0: Intel(R) N200 (family: 0x6, model: 0xbe, stepping: 0x0)
 >
 > {{< quote >}} <https://intel-gfx-ci.01.org/tree/drm-tip/IGT_6642/bat-adln-1/boot0.txt> {{< /quote >}}

Atom系コアのみの構成ながら *Intel Pentium, Celeron* ブランドはプロセッサ名に使われておらず、またモデルナンバーが現行 SKU から続かない 3桁という点から、新たなブランディングを導入したプロセッサ名の可能性が考えられる。  
推測を重ねることとなるが、そうであれば *Alder Lake-N* が初の *Intel Processor* ブランドを導入する世代になるかもしれない。一方、*N シリーズ (Nxxx)* は継続することとなる。  
ただ今回の発表内容と合わせると *Alder Lake-N* のモバイル向け SKU 正式リリースが 2023年ということになる。  
*Alder Lake-N* は GPUドライバ等ですでにサポートされているが、公開されているロードマップ上には置かれていないため、大きな問題は無いと言えば無いが。  
しかし、*Alder Lake-N* を搭載する Chromebook のサポートも Coreboot では進められているため、*Intel Processor* ブランドとは別のブランドで *Alder Lake-N* をリリースする可能性もある。  

| Alder Lake | -S | -P | -M | -N |
| :-- | :--: | :--: | :--: | :--: |
| Variant (CPUID Model) | Desktop (0x97) | Mobile (0x9A) | Mobile (0x9A) | Mobile (0xBE) |
| CPU (big + small) | 8 + 8 | 6 + 8 | 2 + 8 | 0 + 8 |
| GPU (Gen12.2) | GT1 32EU | GT2 96EU | GT2 96EU | GT1 32EU |
| Display ver | Gen12 | Gen13 (XE_LPD) | Gen13 (XE_LPD) | Gen13 (XE_LPD) |
| CPU PCIe | Gen5 8L x2,<br>Gen4 4L x2 | Gen5 8L,<br>Gen4 4L x2 | Gen4 4L? | N/A |
| PCH PCIe | ? | ? | ? | 9-Lane |
| Memory Bus width | 128-bit |128-bit |128-bit | 64-bit |

{{< ref >}}
 * [Intel、「Intel Pentium」と「Intel Celeron」ブランドはただの「Intel」に　2023年から - ITmedia NEWS](https://www.itmedia.co.jp/news/articles/2209/17/news052.html)
 * [Alder Lake-N を搭載する Chromebookボード 「Nissa, Nivviks, Nereid」 | Coelacanth's Dream](/posts/2022/01/12/adl_n-chromebook-board/)
 * [Products formerly Alder Lake](https://ark.intel.com/content/www/us/en/ark/products/codename/147470/products-formerly-alder-lake.html)
{{< /ref >}}
