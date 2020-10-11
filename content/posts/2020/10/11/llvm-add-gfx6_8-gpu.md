---
title: "LLVM に旧世代の GPUID を整理するパッチが投稿される"
date: 2020-10-11T17:34:21+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU", "APU" ]
noindex: false
# summary: ""
toc: false
---

AMD GPU のコンパイラバックエンドとして用いられる LLVM に、現行よりも二世代から四世代前の GPUID を一部整理するためのパッチが投稿された。製品が出た年で言えば、4〜7年前の AMD GPU が対象となる。(R7 250 は1スロットLPカードとして需要があるのか未だに出てきたりするが。)  

これまではある GPU の GPUID を、細かな違いを無視して他の GPUID とまとめて認識、コンパイラバックエンドとして最適化処理を行なっていたが、無視することをやめ、細かな違いを区別するため GPUID を追加、GPU の割り当てを行なったのが今回のパッチとなる。  
{{< link >}} [[AMDGPU] Add gfx602, gfx705, gfx805 targets · llvm/llvm-project@666ef0d](https://github.com/llvm/llvm-project/commit/666ef0db208bb3880115bdc133e72e954ed55300) {{< /link >}}
{{< link >}} [Add new targets for a few cases in old hardware by trenouf · Pull Request #994 · GPUOpen-Drivers/llpc](https://github.com/GPUOpen-Drivers/llpc/pull/994) {{< /link >}}

GPUID については以下を参照。  
{{< link >}} [AMD GPU の GPU ID は何を意味するか | Coelacanth's Dream](/posts/2020/06/22/amdgpu-gpuid-mean/) {{< /link >}}

*gfx602 (Oland, Hainan)* を *gfx601 (Pitcairn, Verde)* と区別した理由は、これまでシェーダーコンパイラのフロントエンド部で *GFX6* 世代に存在する `shaderZExport` 機能の問題に対処していたが、*gfx602 (Oland, Hainan)* ではそれが必要ないからとしている。  

*gfx805* は、*Tonga (gfx802)* のワークステーション向けバリアント *TongaPro* の GPUID となり、倍精度シフト演算が *Tonga* よりも速く処理できるとしている。  
ただ、今回のパッチは *gfx805* の区別とターゲット追加のみであり、その倍精度シフト演算の処理フローは実装されていない。  

そして謎なのが *gfx705* であり、*gfx703* と区別した理由は *gfx602* のように、`shaderSpiCsRegAllocFragmentation` 機能に関する回避策が必要ないからとしているが、その *gfx705* が何の GPU に採用されているかは不明である。記述から APU とされてはいる。[^gfx705-unknown]  
前世代のアーキテクチャを採りながらその正体が不明な APU には [Cato APU](/tags/cato) がいるが、それだったりするのだろうか。  

それにしても発端であり、*Cato APU* をベースとする **A9-9820** を搭載するミニPC、AeroBox は 2020/03 に発表されて以降、発売開始の音沙汰が特に無いが、今どこにいるのだろう。  

[^gfx705-unknown]: <https://github.com/llvm/llvm-project/commit/666ef0db208bb3880115bdc133e72e954ed55300#diff-fc410247cc52875daa3b6a3f8a49e32d>
