---
title: "省電力 Zen プロセッサを搭載する新たな Chromebook ボード ――Vilboz"
date: 2020-05-29T00:12:34+09:00
draft: false
tags: [ "Chromebook", "Pollock" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

省電力と低コストを両立する [Raven2](/tags/raven2) APUを搭載し、ソケットもそれを意識した *FT5 BGAソケット* を実装する Chromebook ボード **Dalboz** 。  
それをベースとする派生ボード、**Valiboz** が出てきた。  
{{< link >}}[mb/google/zork: create new variant for Vilboz (I815ff1b3) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2215157){{< /link >}}

コードネームはしっかり慣わしに従い、Zork 由来となっている。[^1]  

[^1]: [Vilboz the Great](https://www.thezorklibrary.com/history/vilboz.html)

**Dalboz** もそうだが、派生ボードは製品に使うことを想定して開発されているとされ、省電力な Ryzen Chromebook の登場が近づいていることが実感できる。  

## Vilboz 仕様
入力インターフェイスには、タッチスクリーンとタッチパッドが用意されている。  

Ryzenラップトップのクロック管理機能 STAPM(Skin Temperature Aware Power Management) の仕様は **Dalboz** と変わらず、  
持続可能なデフォルトの状態で 4.8W、最大ブースト時でも 9W、ブーストからデフォルトの状態に戻るまでが 6Wとなっている。  

## Pollock のベンチマーク結果
**Dalboz** とそれに搭載される [Pollock](/tags/pollock) だが、GeekBench の実行結果が2つアップロードされている。  
アップロードされたのは 2020/05/26、2020/05/28 と新鮮なベンチマーク結果だ。  
{{< link >}}[Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2301114){{< /link >}}
{{< link >}}[Google zork - Geekbench Browser](https://browser.geekbench.com/v5/cpu/2323968){{< /link >}}
結果から確認できる仕様はどちらも一致している。  
CPU名が **Intel Pentium II/III** となっているが、サンプル品を用いたためだろう。  
{{< link >}}<https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/3/src/soc/amd/picasso/northbridge.c#b326>{{< /link >}}

ベースクロックは 1.2GHz となっており、これは先日こっそり仕様が公開された TDP 6W の **AMD 3020e** 、**Athlon Silver 3050e** と同じものだが、  
ブーストクロックは、末尾に `.gb5` を付けることで見られる生の結果データから、2.35(2.4) GHz と思われる。  
**AMD 3020e** の 2.6 GHz からさらに引き下げられている。  
また、クロックが引き下げられた一方で SMT は有効化されており、その分マルチスレッド性能には期待が持てる。  
{{< link >}}[AMD、TDP 6W の AMD 3020e、Athlon Silver 3050e の仕様をひっそりと公開 | Coelacanth's Dream](/posts/2020/05/20/amd-3020e-and-athlon-silver-3050e/){{< /link >}}

{{< ref >}}

 * [Ryzen Controller](https://www.ryzencontroller.com/)
 * [Ryzen 4000 CPUs explained: How AMD optimized Zen 2 for laptops | PCWorld](https://www.pcworld.com/article/3530289/ryzen-4000-cpus-amd-zen-2-laptops.html)

{{< /ref >}}
