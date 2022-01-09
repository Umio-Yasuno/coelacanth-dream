---
title: "Intel Alder Lake と Rocket Lake に追加される CPUID Model"
date: 2022-01-07T18:12:01+09:00
draft: false
tags: [ "Alder_Lake", "Rocket_Lake" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Intel のソフトウェアエンジニア Cui,Lili 氏より、GCC に *Alder Lake, Rocket Lake* の CPUID Model を追加するパッチが投稿された。  
CPUID (Leaf: 0x1) Model は、CPUアーキテクチャやプロセッサの種類を識別するのに使われる値。  

 * [[PATCH] x86: Update model value for Alderlake and Rocketlake](https://gcc.gnu.org/pipermail/gcc-patches/2022-January/587591.html)

 > 		@@ -415,6 +415,7 @@ get_intel_cpu (struct __processor_model *cpu_model,
 > 		       cpu_model->__cpu_subtype = INTEL_COREI7_SKYLAKE;
 > 		       break;
 > 		     case 0xa7:
 > 		+    case 0xa8:
 > 		       /* Rocket Lake.  */
 > 		       cpu = "rocketlake";
 > 		       CHECK___builtin_cpu_is ("corei7");
 > 		@@ -487,6 +488,7 @@ get_intel_cpu (struct __processor_model *cpu_model,
 > 		       break;
 > 		     case 0x97:
 > 		     case 0x9a:
 > 		+    case 0xbf:
 > 		       /* Alder Lake.  */
 > 		       cpu = "alderlake";
 > 		       CHECK___builtin_cpu_is ("corei7");
 >
 > {{< quote >}} [[PATCH] x86: Update model value for Alderlake and Rocketlake](https://gcc.gnu.org/pipermail/gcc-patches/2022-January/587591.html) {{< /quote >}}

プロセッサに割り当てられている CPUID Model は [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) 「Volume 4: Model-Specific Registers」に記載されている。  
タイムスタンプでは 2021/12/01 に更新されており、GCC へのパッチはそれを反映させたものと思われる。  

## Alder Lake {#adl}
*Alder Lake* ではデスクトップ向け (-S) に 0x97 (151)、モバイル向け (-P/M) に 0x9A (154) が割り当てられている。  
今回追加されたのは 0xBF (191) だが、最近になって coreboot でサポートが進められている *Alder Lake-N* は 0xBE (190) であり、どちらかがタイポ、間違っているのではないかと思われる。  
{{< link >}} [N バリアントも存在する Alder Lake | Coelacanth's Dream](/posts/2021/11/16/coreboot-intel-adl_n/) {{< /link >}}
*Alder Lake-N* については、GPUドライバーにおいてもサポートが進められていることから確かに存在するとされる。それ以外のバリアントが存在して、ドキュメントや coreboot 等で別々にサポート作業が行われているとは考えにくい。  

 > 		enum alderlake_model {
 > 			ADL_MODEL_P_M = 0x9A,
 > 			ADL_MODEL_N = 0xBE,
 > 		};
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/59360/10/src/soc/intel/alderlake/cpu.c#19> {{< /quote >}}

## Rocket Lake {#rkl}
*Rocket Lake* に追加された 0xA8 (168) には謎がある。  
*Rocket Lake* は現在デスクトップ向け (-S)、Model: 0xA7 (167) のみがリリースされており、コンパイラやGPUドライバにおいて、それ以外のバリアントをサポートする動きはこれまで無かった。  
一応 0xA8 は *Rocket Lake* のモバイル向けバリアントに割り当てられていたが、サポートもリリースもされなかったことから、キャンセルされたのではないかという考えが多数だった。  
第11世代 Intel プロセッサにおけるモバイル向けは *Tiger Lake-U* が担当しており、デスクトップ向け *Rocket Lake-S* と棲み分けがなされていた。  
CPUID Model だけではあるが、今になってサポートを追加するのは少し不思議に思える。  
とはいえサポートを追加したから製品としてリリースされるという確証もないが。  
モバイル向けには先日 *Alder Lake-P/M* の SKU が発表されたばかりであるから、単に非公開の内部ドキュメントとのズレを修正したか、手違いということも考えられる。  

それと、以前 *Meteor Lake-M/P, N, S* 、*Raptor Lake-P, S* の CPUID が記載されているとして挙げた [intel/dptf](https://github.com/intel/dptf) には、*Rocket Lake-U/Y, H/S* も記載されている。  
だが `CPUID_FAMILY_MODEL_RKL_H` は、リリース済みであり、明らかな *Rocket Lake-S* の CPUID とは異なり、代わりに *Tiger Lake-H* のものと一致する。  
`CPUID_FAMILY_MODEL_RKL` も *Tiger Lake-U* と一致するため、マクロ名とコメント部についてはミスだと思われる。  

 > 		#define CPUID_FAMILY_MODEL_RKL      0x000806C0		// Rocket Lake U/Y
 > 		#define CPUID_FAMILY_MODEL_RKL_H    0x000806D0		// Rocket Lake H/S
 >
 > {{< quote >}} [dptf/esif_ccb_cpuid.h at e1f10f989223720ccb6b2519f8d96435925407c0 · intel/dptf](https://github.com/intel/dptf/blob/e1f10f989223720ccb6b2519f8d96435925407c0/Common/esif_ccb_cpuid.h#L106-L107) {{< /quote >}}

{{< ref >}}
 * [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
{{< /ref >}}
