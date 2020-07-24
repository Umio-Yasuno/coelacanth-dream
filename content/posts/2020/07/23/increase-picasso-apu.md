---
title: "増える AMD Picasso APU"
date: 2020-07-23T21:00:18+09:00
draft: false
tags: [ "Picasso", "Dali", "Raven2" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

めでたくデスクトップ向け [Renoir](/tags/renoir) が発表され、同時に **Athlon** の名を冠した *Picasso* が増えた。  
{{< link >}} [AMD Ryzen 4000 Series Desktop Processors with AMD Radeon Graphics Set to Deliver Breakthrough Performance for Commercial and Consumer Desktop PCs | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-ryzen-4000-series-desktop-processors-amd-radeon-graphics-set) {{< /link >}}

話題としてはデスクトップ向け *Renoir* の方が大きいけれど、今回自分が語りたいのは *Picasso* の方だ。  

## 例の Picassoシリコンベースな Dali?

まず、今回のプレスリリースで追加された *12nm Picassoシリコン* ベースの SKU が以下。  

 * [Athlon Gold 3150G](https://www.amd.com/en/products/apu/amd-athlon-gold-3150g#product-specs)
 * [Athlon Gold 3150GE](https://www.amd.com/en/products/apu/amd-athlon-gold-3150ge#product-specs)
 * [Athlon Gold PRO 3150G](https://www.amd.com/en/products/apu/amd-athlon-gold-pro-3150g#product-specs)
 * [Athlon Gold PRO 3150GE](https://www.amd.com/en/products/apu/amd-athlon-gold-pro-3150ge#product-specs)
 * [Athlon Silver PRO 3125GE](https://www.amd.com/en/products/apu/amd-athlon-silver-pro-3125ge#product-specs)

そしてこっそり追加されていたのが以下の 2-SKU。  

 * [Ryzen 5 PRO 3350G](https://www.amd.com/en/products/apu/amd-ryzen-5-pro-3350g#product-specs)
 * [Ryzen 5 PRO 3350GE](https://www.amd.com/en/products/apu/amd-ryzen-5-pro-3350ge#product-specs)


**Athlon Silver 3050GE** が省かれているが、それだけはプロセスが 14nm となっており、*Raven2シリコン* ベースであると思われる。  

 * [Athlon Silver 3050GE](https://www.amd.com/en/products/apu/amd-athlon-silver-3050ge#product-specs)

**Athlon Silver PRO 3125GE** は、クロック等のスペックは **3050GE** とほとんど同じであり、クロックが変更可能(アンロック)かどうか、PRO向けの機能が有効かどうかの違いしかないが、**PRO 3125GE** には *Picassoシリコン* が使われているようだ。  

自分が思うに、*Picassoシリコン* をベースとし、 **Athlon** の名を冠したそれらは、以前記事にした、外部 I/O を *Raven2シリコン* 同等までに抑えた *Dali APU* ではないだろうか。  
{{< link >}} [AMD Picassoベースの Dali APU が存在するらしい？ | Coelacanth's Dream](/posts/2020/06/27/amd-dali-apu-based-picasso-or-raven2/) {{< /link >}}

その時に取り上げた Coreboot へのコミットでは *FP5 パッケージ* のみに触れていたが、また別のコミットでは *AM4 パッケージ* でもそういった SKU の可能性があるとしている。[^pco-dali-usb]  
そして、*FP5 パッケージ* における *Picasso* と *Dali* の違いが一部明かされていた。  
*Picasso* は XHCI(eXtensible Host Controller Interface) を `XHCI0` と `XHCI1` の 2基持ち、`XHCI0` は 4 USB2 + 4 USB3 ポートを、`XHCI1` は 2 USB2 + 1 USB3 ポートを担当する。  
*Dali* は `XHCI0` のみを持ち、それが 6 USB2 + 4 USB3 ポートを担当する。  

[^pco-dali-usb]: [mb/google/zork: Disable unsupported SOC usb ports (Ie37e2ac6) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/42800)

*Dali* の仕様は *Raven2シリコン* をそのまま反映しているのではないかと思う。  
また、外部 I/O の規模を抑え、*Raven2シリコン* に合わせるというのは USB だけでなく、PCIe、ディスプレイ出力にも適応される。  

まだ今回追加された **3050GE** を除く **Athlon 3000シリーズ** が *Picassoシリコン* ベースの *Dali APU* と確定した訳ではないが、そうであったらわかりやすいな、というのが本音。  
というより、中身としては *Picasso* であるから CPU/GPU の情報から *Dali* と知ることはできず、何かしら外部 I/O を制限されていることが通知されるのを待つしかなかったりする。  
