---
title: "Alder Lake-P を搭載する Chromebookボード 「Brya」、そして Alder Lake-M"
date: 2020-12-02T01:17:41+09:00
draft: false
tags: [ "Alder_Lake", "Chromebook" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

*Alder Lake-P* を搭載する Chromebookボード、コードネーム **Brya** に関するパッチが Coreboot に投稿され始めている。  
{{< link >}} [mb/google/brya: Add new google brya mainboard (Ia34130ff) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/47819) {{< /link >}}
**Brya** が Chromebookボードにおけるメインボード、またはリファレンスボードとなり、各社はそれを基に製品化に向けたボードを開発する。  

*Alder Lake* には *Alder Lake-S* と *Alder Lake-P* の 2種類が確認されているが、Linux向けの電力モニタリングツールである [PowerTOP](https://github.com/fenrus75/powertop) に送られたパッチから、*Alder Lake-S* はデスクトップ向け、*Alder Lake-P* はモバイル向けであることが確定した。  
{{< link >}} [Add Sapphire rapids, Alder Lake mobile and desktop support in Powertop by gkammela · Pull Request #77 · fenrus75/powertop](https://github.com/fenrus75/powertop/pull/77) {{< /link >}}
パッチを投稿した [Gayatri Kammela (ユーザー名: gkammela)](https://github.com/gkammela) 氏は Linux Kernel 等を担当する Intel のソフトウェアエンジニアだ。  

上記に関連して、CPUID と x86\_Model について雑に補足すると、プロセッサのタイプ、ファミリー、モデルとなる下記引用部のような場合は、16進数であることを示す `0x` の後に続く数字が `Extended Model`、`Extended Family`、`Family`、`Model` といった風に並び、そして末尾が `Stepping` を意味する。[^coreboot-cpuid]  
Coreboot 内のコードに記述されている、下記 *Alder Lake-S/P* の CPUID をデコードすると、   
*Alder Lake-S* は `Family: 0x6, Model: 0x97, Stepping: 0x0`、  
*Alder Lake-P* は `Family: 0x6, Model: 0x9A, Stepping: 0x0` となる。  


 >        #define CPUID_ALDERLAKE_S_A0	0x90670
 >        #define CPUID_ALDERLAKE_P_A0	0x906a0
 >
 > {{< quote >}} [coreboot/mp_init.h at 21910f00de511d2b3ac1f15bb0fbda7761e3847d · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/21910f00de511d2b3ac1f15bb0fbda7761e3847d/src/soc/intel/common/block/include/intelblocks/mp_init.h#L47) {{< /quote >}}

[^coreboot-cpuid]: [inteltool: Print raw CPUID and make hexadecimal values unambiguous · coreboot/coreboot@dcea700](https://github.com/coreboot/coreboot/commit/dcea700762bc97bd7fcabf2e960d47805129aeb1?branch=dcea700762bc97bd7fcabf2e960d47805129aeb1&diff=unified)


## Alder Lake-M

そうして *Alder Lake-S/P* の立ち位置がはっきりした中で、もう 1つ、*Alder Lake-M* が現れた。  

*Alder Lake-M* を搭載したボードをサポートする初期のパッチが ChromiumOS関連のレポジトリに投稿されている。  
{{< link >}} [adlerb:[TEST] Add new board config for ADL-M (I6b2cfa13) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/depthcharge/+/2565086/) {{< /link >}}
パッチの投稿先のレポジトリ、depthcharge は ChromeOS に採用されているブートローダーであり、まさに初期的なサポートと言える。  
ERB は "Early Engineering Reference Board" の略。[^erb]  

[^erb]: [2nd Generation Intel Core Processor Family with Intel 6 Series Chipset Development Kit User Guide - 2nd-gen-core-6-series-chipset-dev-kit.pdf](https://www.intel.com/content/dam/www/public/us/en/documents/guides/2nd-gen-core-6-series-chipset-dev-kit.pdf)

*Alder Lake-M* のサポートは *tglrvp (Tiger Lake RVP)* をベースとし、ドライバー等も *Tiger Lake* となっていることは特筆すべきだろう。*adlrvp (Alder Lake-P)* を見るとドライバーはしっかり *Alder Lake* のものとなっている。[^adlrvp]  
最初のパッチセットでは ALDERLAKE となっていた部分を、後のバージョンで TIGERLAKE に修正しているため、誤字ということはないだろう。  

[^adlrvp]: [src/board/adlrvp: Add ADL-P RVP board support (I745fc592) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/platform/depthcharge/+/2525915)

 >        CONFIG_DRIVER_SOC_TIGERLAKE=y
 >        ~~~
 >        CONFIG_DRIVER_GPIO_TIGERLAKE=y
 >
 > {{< quote >}} (https://chromium-review.googlesource.com/c/chromiumos/platform/depthcharge/+/2565086/5/board/adlmerb/defconfig) {{< /quote >}}


突然現れた *Alder Lake-M* について考えると、まずパッチが depthcharge に投稿されたことから、*Alder Lake-P* と同様のプラットフォームに向けた、CPU と PCH を 1パッケージに統合した形式ということが浮かぶ。  
そしてドライバーの違いから、*Alder Lake-P* とは異なり、*Tiger Lake PCH* と組み合わされる CPU が *Alder Lake-M* ではないか、となるが、まだ初期的なパッチが投稿されたばかりの段階であり、あくまでも個人の推測である。  

ただ付け加えるならば、その唐突さも合わせ、CPU の素性は *Alder Lake-P* と同じであるかもしれない。  
