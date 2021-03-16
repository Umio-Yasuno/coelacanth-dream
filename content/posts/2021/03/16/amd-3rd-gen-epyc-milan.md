---
title: "AMD、Zen 3 アーキテクチャを採用した EPYC 7003シリーズ を正式発表　―― INT8 のスループット 2倍"
date: 2021-03-16T01:26:20+09:00
draft: false
tags: [ "Zen_3", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

現地時間 2021/03/15 付で、*Zen 3 アーキテクチャ* を採用する **EPYC 7003シリーズ** (*Milan*) が正式発表された。  

 * [Introducing 3rd Gen AMD EPYC Processors for the Modern Data Center - Youtube](https://www.youtube-nocookie.com/embed/xtrhHH0kQI0)
 * [AMD EPYC™ 7003 Series CPUs Set New Standard as Highest Performance Server Processor :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/993/amd-epyc-7003-series-cpus-set-new-standard-as-highest)

*Zen 3 アーキテクチャ* を採用する **Ryzen 5000シリーズ** は 2020/10/08 に発表され、*Zen 3 アーキテクチャ* の詳細も既に、*Software Optimization Guide for the AMD Family 19h Processors* が 2020/11/07 には公開されていることもあって[^fam19h-doc]、  
**EPYC 7003シリーズ** の発表は概ね *Zen系アーキテクチャ* の振り返りと、対抗となる Intel Xeonプロセッサとの比較、そして採用事例というものだった。  
 
[^fam19h-doc]: [AMD、Zen 3 アーキテクチャの詳細を公開 | Coelacanth's Dream](/posts/2020/11/07/amd-zen_3-arch-detail/)

しかし、その中で際立っていたのが、*「INT8精度の演算スループットが 2倍になった」* という記述だ。  
上述したように、*Zen 3 アーキテクチャ* 自体は既に発表済みであり、採用したプロセッサが {{< comple >}} 十分かはともかく {{< /comple >}} 世に出回っているが、新機能としてアピールされた中に INT8 のスループットに関するものは無かったと記憶している。  
メディア向けに配布された資料ではなく、AMD公式から誰でも入手できる範囲で資料を探したが、やはり同様の記述は無かった。  

{{< figure src="/image/2021/03/16/amd-epyc-milan-feature.webp" title="" caption="画像出典: [Introducing 3rd Gen AMD EPYC Processors for the Modern Data Center - Youtube](https://www.youtube-nocookie.com/embed/xtrhHH0kQI0?start=567)" >}}

{{< figure src="/image/2021/03/16/amd-ryzen-5000-feature.webp" title="" caption="**Ryzen 5000シリーズ** 発表時のもの。INT8 演算に関しては触れられていない。<br>画像出典: [Where Gaming Begins | AMD Ryzen™ Desktop Processors - Youtube](https://www.youtube-nocookie.com/embed/iuiO6rqYV4o?start=506)" >}}

## INT8 のスループット 2倍 {#2x-int8}

結論から言えば、直接的ではないが、確かに *Zen 3 アーキテクチャ* で前世代から強化された部分に INT8精度演算に関するものがあり、スループットは 2倍になっていた。  

*「Software Optimization Guide」* に同梱している、ある命令を処理するユニット、処理に掛かるレイテンシ、スループット等が記載された表によると、  
*Zen 2 アーキテクチャ (Family 17h Models 30h and Greater Processors)* では `VPMADDUBSW` 命令、`VPMADDWD` 命令を処理できるのは FP0ユニットのみで、命令スループット: 1 となっていたが、  
*Zen 3 アーキテクチャ (Family 19h)* では、FP0ユニットに加え、FP3ユニットでも処理が可能となり、命令スループットは倍の 2 になった。  
`VPADDD` 命令も *Zen 2 アーキテクチャ* は FP0/1/3ユニットで処理可能で、命令スループット: 3 だったのが、*Zen 3 アーキテクチャ* では FP0/1/2/3ユニット へと対応が拡張され、命令スループットは 4 に高速化されている。  

*Zen系アーキテクチャ* では、各 FPユニット (パイプ) は対称的となるようそれぞれ 1基ずつ配置されているが、一部の複雑な命令は片方だけの実装とされており、それを新世代のアーキテクチャで他のユニット (パイプ) にも実装することで前世代よりも高速化できる。  

`VPMADDUBSW/VPMADDWD/VPADDD` 命令は INT8 での低精度で行う推論処理における積和演算に使われている命令で、  
`VPMADDUBSW`命令は、*符号無し 8-bit 整数/unsigned int8 (u8)* と *符号付き 8-bit 整数/signed int8 (s8)* 2組の値を乗算し、結果を *符号付き 16-bit 整数/signed int16 (s16)* のフォーマットに出力、  
`VDMADDWD` 命令はそれで得られた *s16* の値をもう 1つ、ペアで *符号付き 32-bit 整数/signed int32 (s32)* に合計して結果を出力する。  
`VPADDD` 命令はその *s32* の値を、ソースとなるもう 1つの *s32* と加算する。  
こうして INT8 の乗算から INT32 に結果を累算する、畳み込み処理を行える。  
ただ、自分では非常にざっくりとした説明しかできないため、より正確な内容は参考リンク先を参照して頂きたい。  

従来の *Zen/2 アーキテクチャ* では `VPMADDUBSW/VPMADDWD` 命令のスループットが 1 で、上記の処理を行う時ボトルネックとなっていたのが、*Zen 3 アーキテクチャ* では軽減され、倍のスループットを得ている。  
それと、`VPMADDUBSW/VPMADDWD/VPADDD` 命令 3個と処理フローをまとめた命令が `AVX-VNNI/VPDPBUSD` 命令であり、Intel プロセッサでは *Cascade Lake* から 512-bit幅 (`AVX512`) で対応している。[^avx512] マーケティング的には **Intel Deep Learning Boost** という名もある。[^dl-boost]  
*Alder Lake* に搭載される *Golden Cove (Core/Big)* 、*Gracemont (Atom/Small)* からは 128-bit幅、256-bit幅の `AVX-VNNI/VPDPBUSD` 命令に対応している。  
`AVX-VNNI/VPDPBUSD` 命令に対応していない CPU では `VPMADDUBSW/VPMADDWD/VPADDD` 3個の命令が必要だが、対応している CPU では 1個だけでいいため、理論上のピーク性能は 3倍となる。  

[^avx512]: [Intel Sapphire Rapids は AVX512_FP16 をサポート | Coelacanth's Dream](/posts/2021/01/11/intel-spr-avx512_fp16/)
[^dl-boost]: [Intel® Deep Learning Boost - Intel® AI](https://www.intel.com/content/www/us/en/artificial-intelligence/deep-learning-boost.html)

まとめると、*Zen 3 アーキテクチャ* では FPユニットの実装と対応が拡張されたことで INT8 精度の演算を行う一部命令が高速化されており、これが INT8 スループット 2倍の根拠と考えられる。  
**Ryzen 5000シリーズ** 発表時にはアピールされなかった点が今回 **EPYC 7003シリーズ** で強調されたのは、Intel Xeonプロセッサを意識したからだろう。  
*Zen 3 アーキテクチャ* ではまだ `AVX-VNNI/VPDPBUSD` 命令は実装されていないが、AMD も CPU による推論処理の高速化を念頭に置いており、そこでもある程度 Intel Xeon と対抗は可能ということと、今後の CPUアーキテクチャで強化していく方向性を持っていることを示したかったのではないかと思われる。  


{{< ref >}}

 * [MKL-DNNで学ぶIntel CPUの最適化手法 - Cybozu Inside Out | サイボウズエンジニアのブログ](https://blog.cybozu.io/entry/2019/04/15/170000)
 * [算術演算の組込み関数](https://jp.xlsoft.com/documents/intel/compiler/19/cpp_19_win_lin/GUID-E70D11BD-217B-4E52-9C3F-1A9177658929.html)
 * [Lowering Numerical Precision to Increase Deep Learning Performance](https://www.intel.com/content/www/us/en/artificial-intelligence/posts/lowering-numerical-precision-increase-deep-learning-performance.html)
 * [oneDNN: Nuances of int8 Computations](https://docs.oneapi.com/versions/latest/onednn/dev_guide_int8_computations.html)
 * [56665, Software Optimization Guide for AMD Family 19h Processors (PUB)](https://www.amd.com/en/support/tech-docs?keyword=Software+Optimization+19h)
 * [Software Optimization Guide for AMD Family 17h Models 30h and Greater Processors](https://www.amd.com/en/support/tech-docs?keyword=Software+Optimization+17h+30h)

{{< /ref >}}
