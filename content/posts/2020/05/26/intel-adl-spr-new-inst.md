---
title: "Intel Alder Lake、Sapphire Rapids にて追加される2つの命令 ――AVX2 VNNI /HFNI"
date: 2020-05-26T20:08:55+09:00
draft: false
tags: [ "Alder_Lake", "Sapphire_Rapids" ]
keywords: [ "", ]
categories: [ "Intel", "Hardware", "CPU" ]
noindex: false
---

{{< ins datetime="2020-06-22T22:52:10" >}}

この記事で情報元として示したリンク先は削除されたため、現在では不確実な情報を取り扱った記事となります。  

{{< /ins >}}

{{< ins datetime="2020-09-26T20:54:19" >}}

メールアーカイブに残ってました。  

 * [[Bug 1879836] Re: [ADL]Add CPUID feature bit for AVX2 VNNI](https://www.mail-archive.com/ubuntu-bugs@lists.ubuntu.com/msg5789589.html)
 * [[Bug 1879865] Re: [ADL] Enable 5G ISA (FP16) / HFNI](https://www.mail-archive.com/ubuntu-bugs@lists.ubuntu.com/msg5786181.html)

{{< /ins >}}

Intel の次世代プロセッサ、[Alder Lake](/tags/alder_lake)、[Sapphire Rapids](/tags/sapphire_rapids)では新たな拡張命令が2つ追加される。  
そのことが新機能のバグ報告プラットフォーム、[intel in Launchpad](https://launchpad.net/intel)からわかった。  

## AVX2 VNNI
VNNI(Vector Neural Network Instructions) というと、*Cascade Lake* 、*Cooper Lake* 、 *Ice Lake (client/server)* 、*Tiger Lake* がサポートする `AVX512 VNNI`命令 が思い浮かぶが、  
`AVX2 VNNI`命令は、既存のそれとは異なる feature bit を追加する(=また別の命令)としている。  
{{< link >}}[Bug #1879836 “[ADL]Add CPUID feature bit for AVX2 VNNI” : Bugs : intel](https://bugs.launchpad.net/intel/+bug/1879836){{< /link >}}
{{< link >}}[Bug #1880162 “[EGS] Add CPUID feature bit for AVX2 VNNI” : Bugs : intel](https://bugs.launchpad.net/intel/+bug/1880162){{< /link >}}

AVX の後の数字から、用いるベクトル演算器のSIMDベクトル幅が異なるのだろう。(AVX2 は 256-bit、AVX512 はまんま 512-bit)  
512-bit から 256-bit となることで、サイクルあたりのピーク処理能力は小さくなるが、ハードウェアへの実装に割かなければならないリソースもまた減ると思われる。(演算器、レジスタファイル、レジスタポート)  

それと、EGS は *Eagle Stream* の略であり、*Eagle Stream* は Intel の次世代サーバ向けプロセッサ *Sapphire Rapids* のプラットフォームとされている。  

## HFNI
 `HFNI` 命令はその概要から、64個の単精度浮動小数点(FP32) のパック実行に対応していた `AVX-512` 命令を、半精度浮動小数点(FP16) のパック実行にも対応させたものと推察される。  
{{< link >}}[Bug #1879865 “[ADL] Enable 5G ISA (FP16) / HFNI” : Bugs : intel](https://bugs.launchpad.net/intel/+bug/1879865){{< /link >}}

想定される用途として、5G基地局のソフトウェア、FSI(Financial Services Industry)[^1]、科学計算、開発等をあげている。  

[^1]: [FSI Multi-Cloud Strategies Webinar](https://www.intel.com/content/www/us/en/financial-services-it/fsi-multi-cloud-webinar.html)

## Alder Lake のSIMD幅はどうなるか
上記2つの命令は、幅の小さいベクトル演算器を意識しているように見える。  
以前、Intel 拡張命令リファレンス資料がアップデートされた際、 *Alder Lake* と *Tiger Lake* で対応する命令が一致していないと書いたが、*Alder Lake* は *Tiger Lake* よりもベクトル演算器の規模を小さく変更するかもしれない。  
{{< link >}}[Intel、拡張命令リファレンスをアップデート (Sapphire Rapids /Alder Lake /ハイブリッドプロセッサ) | Coelacanth's Dream](/posts/2020/04/01/intel-isa-extensiton-update-sapphirerapids-alderlake/){{< /link >}}
また、 *Alder Lake* には2種類のプロセッサが確認されており(Alder Lake-S /Alder Lake-P)、*Skylake* と *Skylake-SP* のように、それぞれでベクトル演算器周りの構成が異なるということも考えられる。  
{{< link >}}[ChromiumOSへのパッチから Alderlake-S / Alderlake-P を確認 | Coelacanth's Dream](/posts/2020/05/01/vboot-code-add-alderlake/){{< /link >}}

AVX-512 の改良が進められていることは興味が引かれるが、またもプロセッサが対応する範囲が混沌としそうな予感がする。  

{{< ref >}}

 * <https://twitter.com/KOMACHI_ENSAKA/status/1263727714210426880>
 * [インテル® アドバンスト・ベクトル・エクステンション 512 (インテル® AVX-512) の概要](https://www.intel.co.jp/content/www/jp/ja/architecture-and-technology/avx-512-overview.html)
* [【後藤弘茂のWeekly海外ニュース】Intelの10nm世代CPUコア「Sunny Cove」のカギとなるAVX-512 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1167662.html)

{{< /ref >}}
