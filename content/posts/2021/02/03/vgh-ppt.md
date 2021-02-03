---
title: "AMD VanGogh APU では PPT もドライバーから調節可能に"
date: 2021-02-03T18:04:06+09:00
draft: false
tags: [ "VanGogh", ]
# keywords: [ "", ]
categories: [ "AMD", "APU", "Hardware" ]
noindex: false
# summary: ""
toc: false
---

以前 Linux Kernel (amd-gfx) に、*VanGogh APU* がサポートする *Fine Grain Clock Gating* を GPU部だけでなく CPUにも適用し、同時に CPUクロックも GPUドライバーのインターフェイスからの調節を可能とするパッチが投稿された。  
{{< link >}} [VanGogh APU がサポートする "Fine Grain Clock Gating" は CPU にも適用　―― 最大 4コアか | Coelacanth's Dream](/posts/2021/01/08/amd-vgh-cpu-fgcg/) {{< /link >}}
そして今回、またも電力管理機能に関係する、*PPT (Package Power Tracking)* をセンサーに出力し、値の上書きをサポートするパッチが投稿された。  

 * [[PATCH 1/2] drm/amd/pm: update the smu v11.5 smc header for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059074.html)
 * [[PATCH 2/2] drm/amd/pm: add support for hwmon control of slow and fast PPT limit on vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059075.html)

## STAPM, PPT

*PPT (Package Power Tracking)* には Slow と Fast の 2種類があり、それぞれに最大消費電力の値がセットされ、それらの値は、主に APU に実装されている、温度や電力に応じてクロックのブースト状態を制御する *STAPM (Skin Temperature Aware Power Management)* 機能に活用される。  

Slow と Fast の違いについては、まず Fast は極短い時間 (~10ms) 動作する最大ブーストクロックにおける電力制限の設定となり、電力を基準としている。  
Slow は Fast よりも長い時間 (~5000ms) 動作するブーストクロックとその電力制限となる。Slow にはプロセッサのスペック表にある TDP の値が反映されていることが多い。また Slow は熱を基準としており、決められた上限に達すると、通常状態として設定された電力制限まで徐々にクロックを下げるようにして動作する。  
Fast PPT は *high boost* や *short boost* 、Slow PPT は *long boost* と、もっと分かりやすい名で呼ばれたりもしている。  

Fast PPT、Slow PPT、そして通常状態の電力制限 (sustained power limit) の値は UEFI に設定してあるとされ、サポートしているボードだけだが Coreboot のコード中からそれらの値を確認できる。  
例えば、*Picasso APU* を搭載する Chromebookボード *berknip* 、製品名では **HP Pro c645 Chromebook Enterprise** となるボードは、Fast PPT 24W、Slow PPT 20W、通常状態は 12W に設定されている。  

 >        	# Set STAPM confiuration. All of these fields must be set >0 to take affect
 >        	register "slow_ppt_limit_mW" = "20000"
 >        	register "fast_ppt_limit_mW" = "24000"
 >        	register "slow_ppt_time_constant_s" = "5"
 >        	register "stapm_time_constant_s" = "200"
 >        	register "sustained_power_limit_mW" = "12000"
 >
 > {{< quote >}} <https://github.com/coreboot/coreboot/blob/4.13/src/mainboard/google/zork/variants/berknip/overridetree.cb> {{< /quote >}}


CPUクロック、PPT を sysfs から上書きする機能は現時点で *VanGogh* のみがサポートするとしている。  

PPT の値を変更するツールは既にあり、[stapmlifier](https://github.com/sibradzic/stapmlifier) や [Ryzen Controller](https://www.ryzencontroller.com/) 等が挙げられる。  
しかし、プロセッサの電力設定を変更することはハードウェアの損傷に繋がる可能性が高い行為と言え、ある程度の覚悟が必要とされる。  
Linux Kernel では Ryzenプロセッサの電圧、電力を読み取る機能がサポートされていたが、それらを読み取る方法は資料に無く、データを得られても、それを人間に読み取りやすい形式に変換する方法も資料化されていない上、CPU ごとにそれは変化する。そのため、わずかではあるが、範囲外の電圧を報告するといったことでハードウェアに損傷を与える懸念があり、サポートから外された。[^zen-power]  

[^zen-power]: <https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?h=v5.10.4&id=dafbfbed308f5e0013cca56ddb3dec0de65604f9>

今回追加された *VanGogh* での PPT変更機能にも同様のことは言えるが、公式がその手段を提供している点、最大消費電力がコード中に記述されており、それを超える値は設定できないようにされている点で異なり、上記ツールを用いるよりは安全と考えられる。自己責任で行わなければならないことには変わりないが。  
それに、ユーザーにそうした変更を行うことを可能とし、外部ツール無しにユーザービリティをより向上させる余地が与えられること自体は喜ばしい。  

## VanGogh APU

それと、最大消費電力については、コード中に 29W であるように記述されている。  
*Picasso APU* や *Renoir APU* がデスクトップ向けモデルまでもをカバーし、モデル次第では TDP 65W までの設定が可能だったことを考えると、その半分以下、低めの値に思える。  
また、この値は Fast PPT に設定可能な最大消費電力であり、TDP が反映される Slow PPT はそれよりも少し低い値となることも考慮しなければならない。  

 >        +	/* convert from milliwatt to watt */
 >        +	smu->current_power_limit = ppt_limit / 1000;
 >        +	smu->max_power_limit = 29;
 >        +
 >        +	ret = smu_cmn_send_smc_msg(smu, SMU_MSG_GetFastPPTLimit, &ppt_limit);
 >        +	if (ret) {
 >        +		dev_err(smu->adev->dev, "Get fast PPT limit failed!\n");
 >        +		return ret;
 >        +	}
 >        +	/* convert from milliwatt to watt */
 >        +	power_context->current_fast_ppt_limit = ppt_limit / 1000;
 >        +	power_context->max_fast_ppt_limit = 30;
 >        +
 >        +	return ret;
 >        +}
 >        +
 >
 > {{< quote >}} [[PATCH 2/2] drm/amd/pm: add support for hwmon control of slow and fast PPT limit on vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059075.html) {{< /quote >}}

以前には、デフォルトでの最小CPUクロックを 1.4GHz (1400MHz) 、最大CPUクロックは 3.5GHz (3500MHz) とするような記述が追加され、そしてパッチ内では、さり気なく CPUコア数は 4コアとするようなコメントがある。[^vgh-cpu-core]  

[^vgh-cpu-core]: [[PATCH 7/7] drm/amd/pm: implement processor fine grain feature for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058001.html)

 >        @@ -1351,6 +1422,11 @@ static int vangogh_set_fine_grain_gfx_freq_parameters(struct smu_context *smu)
 >         	smu->gfx_actual_hard_min_freq = 0;
 >         	smu->gfx_actual_soft_max_freq = 0;
 >         
 >        +	smu->cpu_default_hard_min_freq = 1400;
 >        +	smu->cpu_default_soft_max_freq = 3500;
 >        +	smu->cpu_actual_hard_min_freq = 0;
 >        +	smu->cpu_actual_soft_max_freq = 0;
 >        +
 >         	return 0;
 >         }
 >
 > {{< quote >}} [[PATCH 7/7] drm/amd/pm: implement processor fine grain feature for vangogh](https://lists.freedesktop.org/archives/amd-gfx/2021-January/058001.html) {{< /quote >}}

*Fine Grain Clock Gating* のサポート等も合わせて *VanGogh APU* の姿を想像すると、省電力を重視に開発されたプロセッサが描かれる。  
省電力重視と言っても決して性能をただ犠牲にした訳ではなく、GPU部には *RDNA 2/GFX10.3* アーキテクチャが採用され、LPDDR5メモリのサポートも確定しており、GPU性能、電力効率は *Vega/GFX9* アーキテクチャを採用する現世代 APU より向上しているだろう 。  

普段、絶対性能の高さが重要とされるゲームをしない身としては、最新技術、アーキテクチャを採用しながら省電力を重視したプロセッサというのは魅力的に映る。  
*VanGogh APU* がモバイル向けに展開されるか、自分がそれを買えるかはともかくとして、今登場が一番楽しみな APU である。  

|                   | AMD VanGogh |
| :--               | :--: |
| Platform Codename | Chachani[^chachani]<br>(Aerith?) |
| CPU Core/Thread   | 4/8? |
| CPU Clock Range (default) | 1.4GHz - 3.5GHz? |
| GPU Arch | RDNA 2/GFX10.3<br>(gfx1033?) |
| Memory | DDR4/LPDDR4?/LPDDR5<br>(DDR5?) |
| Power | ~29W? |

[^chachani]: [[PATCH 1/2] drm/amdgpu/display: fix the NULL pointer reference on dmucb on dcn301](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054813.html)

{{< ref >}}

 * [【後藤弘茂のWeekly海外ニュース】AMDの新APU「Bristol Ridge」のパフォーマンスアップ手法 - PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1002613.html)
 * [AMD，次世代モバイル向けAPU「Mullins」と「Beema」の概要を公開。Bay Trailを追撃する切り札となるか？](https://www.4gamer.net/games/239/G023938/20140428069/)
 * [Hot Chips-Ryzen Mobile-Sonu Arora 071720 V17 - with employee intro and end thanks to team - HotChips2020_Mobile_Processors_AMD_Renoir.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_Mobile_Processors_AMD_Renoir.pdf#page=19)

{{< /ref >}}
