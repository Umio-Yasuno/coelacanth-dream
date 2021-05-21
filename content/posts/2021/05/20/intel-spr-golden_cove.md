---
title: "Intel Sapphire Rapids は Golden Coveアーキテクチャ"
date: 2021-05-20T21:44:49+09:00
draft: false
tags: [ "Sapphire_Rapids", ]
# keywords: [ "", ]
categories: [ "Intel", "CPU", Hardware ]
noindex: false
# summary: ""
---

今更とか思われるかもしれないが、 *ようやく* Intel が公式的に公開した情報。  

## Sapphire Rapids / Golden Cove

*Sapphire Rapids* の CPUマイクロアーキテクチャが *Tiger Lake* と同じ *Willow Cove* か、*Alder Lake* の Core (Big) 側と同じ *Golden Cove* かについては色んな人が推測し、*Golden Cove* なんじゃないかと言われてきていたが、Intel がそこに言及したことは今まで無かった。少なくとも自分はそう記憶してる。  
*Alder Lake* は 2020/08/13 に開催された **Intel Architecture Day 2020** で早くに *Golden Cove (Core/big)* 、 *Gracemont (Atom/small)* という共に次世代のマイクロアーキテクチャを採用したハイブリッド構成になることが発表されていた。  
{{< link >}} [Intel Architecture Day 2020 個人的まとめ　―― XeHP は 1-Tile 512EU、XeLPアーキテクチャ詳細 | Coelacanth's Dream](/posts/2020/08/14/intel-architecture-day-2020/#adl) {{< /link >}}

用途に 5G基地局が想定されている `HFNI/AVX512_FP16` 命令を *Sapphire Rapids* がサポートすることと、*Golden Cove* の特徴に Network/5G Perf があったことを合わせれば、公式的に広く公開されている情報を元にある程度の根拠を持って、 *Sapphire Rapids* は *Willow Cove* ではなく *Golden Cove* だと推測することもできた。他にも *Alder Lake* と *Sapphire Rapids* は `SERIALIZE` 、`AVX_VNNI (128/256-bit)` 命令をサポートしている点で共通している。だが、やはり今まで Intel ははっきりとそう明言してくれることは無かった。  
{{< link >}} [Intel Sapphire Rapids は AVX512_FP16 をサポート | Coelacanth's Dream](/posts/2021/01/11/intel-spr-avx512_fp16/) {{< /link >}}

さらには Intel CPU の識別に使われる `Model` 情報をまとめた、Linux Kernel に含まれる [intel-family.h](https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/intel-family.h) ファイルに CPUマイクロアーキテクチャとの関係性を補完するコメントを追加するパッチが投稿された時には、*Sapphire Rapids* が *Willow Cove アーキテクチャ* ということにされており、余計曖昧となっていた。  
{{< link >}} [Intel Kaby Lake 周りの CPU Stepping を整理するパッチ | Coelacanth's Dream](/posts/2021/04/09/intel-kbl-complex/#spr) {{< /link >}}

*Sapphire Rapids* の CPUマイクロアーキテクチャはそうした状況にあったが、今回 Intel の [Andi Kleen](https://github.com/andikleen) 氏より新たに投稿されたパッチにて先のパッチで追加したコメントを修正、 *Sapphire Rapids* は *Golden Cove アーキテクチャ* を採用することが明言された。  

 * [LKML: Andi Kleen: [PATCH] x86/cpu: Fix comment for Sapphire Rapids](https://lkml.org/lkml/2021/5/13/660)
 * [[PATCH] x86/cpu: Fix comment for Sapphire Rapids - Andi Kleen](https://lore.kernel.org/lkml/20210513163904.3083274-1-ak@linux.intel.com/)

 > 		 Sapphire Rapids uses Golden Cove, not Willow Cove.
 > 		
 > 		Cc: peterz@infradead.org
 > 		Fixes: 53375a5a218e ("x86/cpu: Resort and comment Intel models")
 > 		Signed-off-by: Andi Kleen <ak@linux.intel.com>
 > 		---
 > 		 arch/x86/include/asm/intel-family.h | 3 ++-
 > 		 1 file changed, 2 insertions(+), 1 deletion(-)
 > 		
 > 		diff --git a/arch/x86/include/asm/intel-family.h b/arch/x86/include/asm/intel-family.h
 > 		index 955b06d6325a..27158436f322 100644
 > 		--- a/arch/x86/include/asm/intel-family.h
 > 		+++ b/arch/x86/include/asm/intel-family.h
 > 		@@ -102,7 +102,8 @@
 > 		
 > 		 #define INTEL_FAM6_TIGERLAKE_L		0x8C	/* Willow Cove */
 > 		 #define INTEL_FAM6_TIGERLAKE		0x8D	/* Willow Cove */
 > 		-#define INTEL_FAM6_SAPPHIRERAPIDS_X	0x8F	/* Willow Cove */
 > 		+
 > 		+#define INTEL_FAM6_SAPPHIRERAPIDS_X	0x8F	/* Golden Cove */
 >
 > {{< quote >}} [LKML: Andi Kleen: [PATCH] x86/cpu: Fix comment for Sapphire Rapids](https://lkml.org/lkml/2021/5/13/660) {{< /quote >}}

これで *Sapphire Rapids* にまつわる一つの小話はめでたしめでたしで終わったと言えるだろう。  
OSS に対して企業のソフトウェアエンジニアが公式に投稿するパッチは、ネット上で信頼できる情報ではかなり上位に位置すると考えているが、やはりエンジニアも人であり間違うこともある。  
情報を再発信する側は多角的に収集、選別し考えることが必要なのだと改めて実感もした小話の流れでもあった。  
