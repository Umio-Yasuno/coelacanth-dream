---
title: "AMD Pollock の x86_Model は \"0x20\""
date: 2020-04-24T01:02:04+09:00
draft: false
tags: [ "Raven", "Picasso", "Raven2", "Dali", "Pollock", "Renoir" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

以前、ややこしいややこしいと書いた Zen APU に関して、[Raven2 (Pollock)](/tags/pollock)のx86_Model が `20h` となることがわかった。  

{{< link >}}参考: <cite>[soc/amd: Update macro name for IOMMU on AMD Family 17h (I17c78200) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2153715)</cite>{{< /link >}}

## インデックス

 * [x86_Model](#x86_model)
 * [Pollock の x86_Model は "0x20"](#pollock)

## x86_Model {#x86_model}
現在主流の x86-64プロセッサはその判別に Family ID と Model を用いられる。  
基本的に、Family ID は ベースとなるアーキテクチャを、Model はより詳細なプロセッサの種類を表す。  
例で言えば、**Ryzen 5 2600** は Family ID: 0x17、Model: 0x8 となっており、この値は Linux Kernelの温度センサーに関するコードでは *Zen+* と判別され、[^1]GCCでは `znver1` と判別される。[^2]  

[^1]: <https://github.com/torvalds/linux/blob/b02c6857389da66b09e447103bdb247ccd182456/drivers/hwmon/k10temp.c#L589>
[^2]: <https://github.com/gcc-mirror/gcc/blob/master/libgcc/config/i386/cpuinfo.c#L109>

そして、Linux Kernelの温度センサーに関するコードを読むと、以下のように記述されている。

 >		case 0x1:	/* Zen */
 >		case 0x8:	/* Zen+ */
 >		case 0x11:	/* Zen APU */
 >		case 0x18:	/* Zen+ APU */

 > 引用元: <cite><https://github.com/torvalds/linux/blob/b02c6857389da66b09e447103bdb247ccd182456/drivers/hwmon/k10temp.c#L587></cite>

これに従えば、`0x18` は *Zen+ APU* 、[Picasso](/tags/picasso)となるが、  
*Zen APU* である[Raven2](/tags/raven2)までも x86\_Model が `0x18` となっており、それによって *Zen+ APU /Picasso* と判別されてしまう――というのを前回記事にした。  
{{< link >}}[いかに Zen APU は複雑か | Coelacanth's Dream](/posts/2020/02/16/raven-family-complex/#cpu){{< /link >}}

## Pollock の x86_Model は "0x20" {#pollock}
そういうことで *Raven2* の影が薄いことを勝手に憂いていたが、先日投稿された ChromiumOSのリポジトリにおける Coreboot へのパッチで、[Pollock](/tags/pollock)の x86_Model が `0x20` であることがわかった。  

{{< link >}}参考: <cite>[soc/amd: Update macro name for IOMMU on AMD Family 17h (I17c78200) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2153715)</cite>{{< /link >}}

コミットメッセージでは *FP5ソケット* が `0x18` としているが、これはChromebookに採用される *FP5ソケット* 搭載のAPUが *Raven* 以降の *Picasso* 、*Raven2 (Dali)* のみであるからと思われる。  
そして、*FT5ソケット* を `0x20` としているが、それに搭載するAPUは *Raven2 (Pollock)* だけとしている。[^3]  
これで 12nm *Zen+ APU /Picasso* ではなく、14nm *Zen APU /Raven2 (Pollock)* としっかり判別されるはずだ。  

[^3]: [soc/amd/picasso: Add Kconfig option for chip footprint (Ia4663d38) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051509)

### 気がかり
気がかりなのは、CPU-Z[^4]といったハードウェアの情報を表示するソフトウェアで*FT5ソケット* *Raven2 (Pollock)* を見た時、コードネームの項目に表示されるのは *Raven2* となるか *Pollock* となるかだ。  
個人的には、*Pollock* は *Raven2* のリビジョンの1つというものであるため、*Raven2* と表示してほしい。  

[^4]: [CPU-Z | Softwares | CPUID](https://www.cpuid.com/softwares/cpu-z.html)

それと、[Raven2 (Dali)](/tags/dali)は恐らく *Picasso* と判別されるままだろう。  
Chromebookには *Raven2 (Dali)* ベースの **Ryzen 3 3250C** 、**Athlon Silver 3050C** 、 **Athlon Gold 3150C** も採用されるが、これらは数字セグメント的に *FP5ソケット* に搭載される可能性が高い。[^5]  
そしてコミットメッセージから x86_Model は `0x18` となるはずだ。  

[^5]: [AMD Ryzen™ 3 3250U | AMD](https://www.amd.com/en/products/apu/amd-ryzen-3-3250u#product-specs)&emsp;/[AMD Athlon™ Silver 3050U | AMD](https://www.amd.com/en/products/apu/amd-athlon-silver-3050u#product-specs)&emsp;/[AMD Athlon™ Gold 3150U | AMD](https://www.amd.com/en/products/apu/amd-athlon-gold-3150u#product-specs)

つまり、*AM4ソケット* 、*FP5ソケット* に搭載される *Raven2* 、*Raven2 (Dali)* は *Picasso* と判別され続け、  
*FT5ソケット* に搭載される *Raven2 (Pollock)* は正しくそのままに判別される。  

やっぱりややこしいままな気も。  
