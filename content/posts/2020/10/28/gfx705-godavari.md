---
title: "gfx705 には Godavari が該当"
date: 2020-10-28T17:50:25+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

以前、[LLVM に旧世代の AMD GPU に向けた GPUID が追加された](/posts/2020/10/11/llvm-add-gfx6_8-gpu/) 際、*gfx703* から分化し、ある APU の GPUコア、ISA とされるが、具体的な採用製品は謎の *gfx705* があった。  
自分はそれを最近出てきた、**Xbox One SoC** と同一のダイとなる異質の APU **A9-9820 )
Cato APU** が当たるのではないかと考えていたが、実際には 2015年頃に登場した *Godavari APU* が該当するようだ。  
{{< link >}} [pal/ndDevice.cpp at 007816f4bd2fd3ace30f8af9629cff7ec1dcb998 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/007816f4bd2fd3ace30f8af9629cff7ec1dcb998/src/core/os/nullDevice/ndDevice.cpp#L123) {{< /link >}}

**A9-9820** の GPU部に付けられた **R7 350** の謎はそのままだが、  
**A9-9820** のある Geekbench 5 実行結果によると、どうやら **A9-9820** でも Linux環境を動かすことが可能らしい。[^cato-gb5]  

[^cato-gb5]: [AMD X64 - Geekbench Browser](https://browser.geekbench.com/v5/cpu/4318888)

また、**A9-9820** を搭載するボードには複数の種類が確認でき、  
AliExpress で販売されているボード[^cato-ali]、CHUWI AeroBox に組み込まれているボード[^cato-aerobox]、Geekbench 5 の実行結果にある AMD_BL2 を検索すると出てくる PDFファイルに掲載されているボード[^cato-bl2]の 3種類となる。  

[^cato-ali]: <https://aliexpress.com/item/1005001340237782.html>
[^cato-aerobox]: [CHUWI AeroBox 実機レビュー － CHUWIのスリムなデスクトップPC。「謎のAMD製CPUと外部GPU」はホーム・オフィス向けの性能](https://win-tab.net/imported/chuwi_areobox_review_2010142/)
[^cato-bl2]: [AMD_BL2_V2.0.pdf](http://www.dj-product.com:8811/AMD_BL2%20V2.0/AMD_BL2_V2.0.pdf)

それぞれの違いは、CPU補助電源が 4-pin か 8-pin か、PCIe x1スロットの有無または数、mSATA、M.2スロットの数等、同じ **A9-9820** を搭載するため仕方ないかもしれないが、微妙なものとなる。  
全く同じボードを使っていないというのは少し意外で、**Xbox One SoC / A9-9820** のストックは結構あるのかもしれない。  

Linux が動作可能ということもあり、その内さらなる **A9-9820** の詳細が明らかになるのではないかと期待している。  

