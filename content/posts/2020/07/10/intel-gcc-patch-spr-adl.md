---
title: "Intel、GCC に Sapphire Rapids と Alder Lake をサポートするパッチを投稿"
date: 2020-07-10T21:09:56+09:00
draft: false
tags: [ "Sapphire_Rapids", "Alder_Lake" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
---

Intel は次世代プロセッサ、*Sapphire Rapids* と *Alder Lake* をサポートするパッチを GCC に投稿した。まずは拡張命令の対応となっている。  
{{< link >}}[Initial Sapphire Rapids and Alder Lake support from ISA r40](https://gcc.gnu.org/pipermail/gcc-patches/2020-July/549699.html){{< /link >}}
{{< link >}}[Initial Sapphire Rapids and Alder Lake support from ISA r40 · gcc-mirror/gcc@ba9c87d](https://github.com/gcc-mirror/gcc/commit/ba9c87d3255f168db811dd1fa69e5011d4e8194f){{< /link >}}

パッチを投稿した Cui, Lili 氏は、*Sapphire Rapids* の拡張命令の対応範囲を、*Cooper Lake* の対応範囲に `MOVDIRI /MOVDIR64B /AVX512VP2INTERSECT /ENQCMD /CLDEMOTE /PTWRITE /WAITPKG /SERIALIZE /TSXLDTRK` に追加したもの、  
*Alder Lake* は *Skylake* の対応範囲に `CLDEMOTE /PTWRITE /WAITPK /SERIALIZE` を追加したものと説明している。  

{{< comple >}}現在はページが消されているが{{< /comple >}}以前判明した `AVX2 VNNI` 、`HFNI` には触れられていないが、 **Intel®ArchitectureInstruction Set Extensions and Future FeaturesProgramming Reference** にもまだ出てきていないためだろう。  
{{< link >}}[Intel Alder Lake、Sapphire Rapids にて追加される2つの命令 ――AVX2 VNNI /HFNI | Coelacanth's Dream](/posts/2020/05/26/intel-adl-spr-new-inst/){{< /link >}}

そしてやはり *Alder Lake* は AVX512 には対応しないような動きを見せている。  
また今回は *Cooper Lake* 、*Skylake* の対応範囲に加える形にあるためか、 *Cannon Lake* 、*Goldmont* から対応した `SHA` 命令に、 *Sapphire Rapids* と *Alder Lake* は対応していない。  
わざわざ外すというのも考えにくいため、今後のパッチで追加されるものと思われる、たぶん。  

そして、*Alder Lake* はハイブリッドプロセッサと言われることが多いが、今回のパッチで *Alder Lake* は高性能コア (Core) 側とされている。  
また、*Lakefield* もハイブリッドプロセッサであり、拡張命令の対応範囲は *Ice Lake* とも *Tremont* とも異なるが、コンパイラに `-march=lakefield` といったものはこれまで追加されていない。  
そうなると、*Alder Lake* が先行して追加されているということにはズレを感じる。  

{{< ref >}}

 * [GCC 11 Compiler Lands Intel Sapphire Rapids + Alder Lake Support - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=GCC11-SapphireRapids-AlderLake)

{{< /ref >}}
