---
title: "AMD RDNA 2 のコードネーム良し悪し"
date: 2020-07-27T18:18:25+09:00
draft: false
tags: [ "RDNA_2", "Sienna_Cichlid", "Navy_Flounder" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

*AMD RDNA 2 GPU* 以前のコードネームは、[Vega10](/tags/vega10)、[Vega12](/tags/vega12)、[Vega20](/tags/vega20)、[Navi10](/tags/navi10)、[Navi12](/tags/navi12)、[Navi14](/tags/navi14) と、星と数字を組み合わせた、短く、シンプルでわかりやすいものだった。  
しかし *RDNA 2* からは一転、[Sienna Cichlid](/tags/sienna_cichlid) 、[Navy Flounder](/tags/navy_flounder) のように色と硬骨魚を組み合わせた、長くはあるが、キャラクター性のあるものとなった。  
[Arcturus](/tags/arcturus) なんかは、星ではあるがコードネームとして末尾に数字は使われておらず、それでいて少し長い。中間的なコードネームと言えるかもしれない。  

AMD が *RDNA 2* にこうしたコードネームを付けるようになった理由は想像でき、それによる良い点はあるが、悪い点もある。  

{{< pindex >}}

 * [避けたい名前の混同](#wont-confuse)
 * [襲いかかる Typo](#attack-of-typo)

{{< /pindex >}}

## 避けたい名前の混同 {#wont-confuse}
ソースコード中では、関数名や変数名にコードネームをさらに縮めたものを使うことがある。  
その時、例えば *Vega10* は *vg10* 、*Vega20* は *vg20* 、*Navi10* は *nv10* 、*Navi12* は *nv12* という風になる。  

[^chip-sienna]: [radeonsi: add support for Sienna Cichlid (!5383) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5383/diffs?commit_id=c09cac343eb8dbca0b8dda24941540b20768702b#diff-content-fbbe09bc487101e3f2f96c0b8fe543c915fa99bf)

しかし *vg10* のような表記には問題があり、*Navi1x* 世代で顕在化した。  
ピクセルフォーマットとの混同だ。  
ピクセルフォーマットには **NV12** というものが存在し、これが *Navi12* の略称 *nv12* と被り、混同が発生する。  
ディスプレイコントローラに関連したコードでは、よく見ればわかりはするものの、1つのファイル上にピクセルフォーマットを意味する **nv12** と *Navi12* を意味する *nv12* が同時に存在しており、まあややこしい。[^dcn20-nv12]  

[^dcn20-nv12]: [dcn20_resource.c\dcn20\dc\display\amd\drm\gpu\drivers - ~agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/dcn20/dcn20_resource.c?h=amd-staging-drm-next&id=caaf9e36ffe8847c85aa942caf5c3ddeb4fc92a5#n1023)


そして、ピクセルフォーマットには **NV21** というものも存在する。  
そのため、引き続き *Navi21* を *nv21* のように略しコード中で使うと、 *nv12* 同様の問題が起こってしまう。  
かといって、同パターンのコードネームであるのに略し方を変えてしまうと、それはそれでややこしい。  
そんならと新たに付けたコードネームが、「色+硬骨魚」な *Sienna Cichlid* 、*Navy Flounder* ではないか、というのが想像した理由。  

*nv12* の混同による影響は実際にあり、同ファイル上ではないが、あるパッチに記述された *nv12* を *Navi12* と勘違いし、「*Navi12* は APU」なんて記事を書いた海外サイトもある。  
{{< link >}} [Is the AMD Navi 12 GPU actually powering the Ryzen 4000 APUs after all? | PCGamesN](https://www.pcgamesn.com/amd-navi-12-ryzen-4000-renoir-apu-display-engine) {{< /link >}}
*Sienna Cichlid* のパッチが初めて投稿された時にも思ったが[^sienna-first-patch]、オープンソースコードを読むのって自分含めたオタクだけで、あまり読まれないものなのだろうか。  
自分も誤解釈をしない訳ではないため、あまり人のことを言えないが。  

[^sienna-first-patch]: [AMD の次世代GPU、Sienna Cichlid の Linux Kernelパッチが投稿される | Coelacanth's Dream](/posts/2020/06/02/amd-sienna_cichlid/)

*RDNA 2* のコードネームをどう略しているのかについては、 *Sienna Cichlid* だと自分が見たことあるものでは、頭文字を取って *SC* とは略さず、前の節である色の部分を使っている。[^chip-sienna]  
*Arcturus* は *arct* 等のように略されているが、Architecture を略した *arch* と混同しやすいように思え、適切かどうかは判断しかねる。  

## 襲いかかる Typo {#attack-of-typo}
上記のような混同を避けられるのが *RDNA 2* のコードネームの利点ではあるが、欠点も存在する。  
文字数の長さから来る誤字脱字、Typo だ。他名称混同と比べれば小さいが、出会う頻度は高い。  

自分もそれを予感し、気を付けてはいたが、つい先日に一ヶ月半近く前の誤字を発見した。[^my-typo]まだまだ潜んでいる気がしてならない。  

[^my-typo]: [2020-07-23T11:18:30+0900 · Umio-Yasuno/coelacanth-dream@2f1918d](https://github.com/Umio-Yasuno/coelacanth-dream/commit/2f1918d42d25960b7996b9983a41c4caa5a44801#diff-371619bf2137c3dc1db90e346519bf93)

誤字脱字、Typo は誰にでも襲いかかるものであり、オープンソース Vulkan ドライバー、**RADV** が *Sienna Cichlid* をサポートした時も、マージリクエストのタイトルが `Sienna Sichild -> Sienna Cichild -> Sienna Cichlid` と修正され、移り変わっている。[^radv-fight-of-typo]  

[^radv-fight-of-typo]: [radv: add Sienna Cichlid (!5389) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5389)

コード中に誤字脱字が出ることが多くなるかもしれないが、ほとんどはコンパイラが出力するメッセージとコードレビューの時点で気付けるはずだ。  

よって、混同から来る概念的なズレよりも、単純な誤字脱字の方がマシ、なはず。  
