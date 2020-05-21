---
title: "Intel、DG1 に向けた一連の Linux Kernel パッチを投稿"
date: 2020-05-21T14:01:29+09:00
draft: false
tags: [ "DG1", "Gen12" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
---

Intel は、[Gen12](/tags/gen12)アーキテクチャ採用のディスクリートGPU、[DG1](/tags/dg1) をサポートする一連の Linux Kernel パッチを投稿した。パッチには [Rocket Lake](/tags/rocket_lake) のためのコードも含まれている。  
{{< link >}}[[Intel-gfx] [PATCH 00/37] Introduce DG1](https://lists.freedesktop.org/archives/intel-gfx/2020-May/240205.html){{< /link >}}

今回のパッチからわかる情報としては

 * システムメモリとのコヒーレントは PCIe を通して取られる[^3]
 * SLM (Shared Loacal Memory) の ECCカウント機能に対応[^2]
 * 4基の ピクセルクロックを生成する DPLL (Digital Phase-Locked Loop) を持つ(=最大4画面出力?)[^4]

がある。  

[^4]: [[Intel-gfx] [PATCH 20/37] drm/i915/dg1: Add DPLL macros for DG1](https://lists.freedesktop.org/archives/intel-gfx/2020-May/240236.html)
[^3]: [[Intel-gfx] [PATCH 10/37] drm/i915: add pcie snoop flag](https://lists.freedesktop.org/archives/intel-gfx/2020-May/240212.html)
[^2]: [[Intel-gfx] [PATCH 27/37] drm/i915/dg1: Log counter on SLM ECC error](https://lists.freedesktop.org/archives/intel-gfx/2020-May/240226.html)

*DG1* は先日、オープンソースな UMD (User Mode Driver)、Mesa3D への関連したパッチが投稿されており、Linux 環境におけるサポートは徐々に進んできている。  
{{< link >}}[Intel、オープンソースドライバーに DG1 と Rocket Lake の関するコードを追加 ――DG1 は 96EU、RKL は 16EU または 32EU | Coelacanth's Dream](/posts/2020/05/08/intel-add-dg1-rkl-oss-driver/){{< /link >}}
ただ、PCIeカードタイプの *DG1* はソフトウェア開発用としてのみ供給され、一般向けにはオンボードで実装したノートPCしか出てこないという話が出てきている。[^5]  
Intel Gen12アーキテクチャを採用した、初(?)の dGPU ということで、性能関係なく欲しいと考える人はそれなりにいると思われるため、少し残念ではある。  
ドライバーのサポートは為されているため、入手さえ出来れば、動作させることはそれほど難しくないかもしれないが、その場合、クローズドソースであるファームウェアが公開されるかが問題となる。[^6]  
それさえ解決されれば、一般用における実用性も、これまでの *Xeon Phi* といった製品に比べればずっとあるため、気合で手に入れる人が出てくるかもしれない。  

[^5]: <https://ecpannualmeeting.com/assets/overview/sessions/Aurora-Public-FULL-talk-Feb-4-2020_for_posting_c.pdf#page=7>
[^6]: [[Intel-gfx] [PATCH 35/37] drm/i915/dg1: Load DMC](https://lists.freedesktop.org/archives/intel-gfx/2020-May/240240.html)
