---
title: "AMDGPUドライバーに SMU 13.0.5 のサポートを追加するパッチ"
date: 2022-02-27T10:10:15+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Linux_Kernel", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMDGPUドライバーに SMU (System Management Unit) 13.0.5 のサポートを追加するパッチが、amd-gfxメーリングリストにて公開されている。  
SMU 13.0.5 では、*Yellow Carp (Rembrandt)* に採用されている SMU 13.0.{1,3} とは別の電力管理テーブル、インターフェイスを用いる。  
最近では SMU 13.0.8 のサポートを追加するパッチも公開されているが、機能的には SMU 13.0.{1,3} と共通としている。[^smu_13_0_8]  

[^smu_13_0_8]: [[PATCH] drm/amd/pm: Add support for MP1 13.0.8](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075431.html)

* [[PATCH] drm/amdgpu: add gfxoff support for smu 13.0.5](https://lists.freedesktop.org/archives/amd-gfx/2022-February/076009.html)
* [[PATCH 0/3] Add SMU 13.0.5 Support](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075620.html)
    * [[PATCH 1/3] drm/admgpu/pm: add smu 13.0.5 driver interface headers](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075622.html)
    * [[PATCH 2/3] drm/amd/pm: update smc message sequence for smu 13.0.5](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075621.html)
    * [[PATCH 3/3] drm/amd/pm: add smu_v13_0_5_ppt implementation](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075623.html)
* [[PATCH] drm/amd/pm: refine smu 13.0.5 pp table code](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075959.html)

{{< pindex >}}
 * [SMU 13.0.5](#smu)
    * [SmartShift をサポートせず？](#ss)
    * [CPU, L3キャッシュ](#cpu)
 * [GPUクロック](#clock)
{{< /pindex >}}

## SMU 13.0.5 {#smu}
まず、SMU 13.0.5 は以下引用部から、APU に採用されることを想定した IPブロックとされる。  
だが、他 APU の SMU とは違う部分がいくつか見られる。  

 > 		+void smu_v13_0_5_set_ppt_funcs(struct smu_context *smu)
 > 		+{
 > 		+	smu->ppt_funcs = &smu_v13_0_5_ppt_funcs;
 > 		+	smu->message_map = smu_v13_0_5_message_map;
 > 		+	smu->feature_map = smu_v13_0_5_feature_mask_map;
 > 		+	smu->table_map = smu_v13_0_5_table_map;
 > 		+	smu->is_apu = true;
 > 		+}
 >
 > {{< quote >}} [[PATCH 3/3] drm/amd/pm: add smu_v13_0_5_ppt implementation](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075623.html) {{< /quote >}}

### SmartShift をサポートせず？ {#ss}

パッチでは、SMU 13.0.5 は SmartShift用のインターフェイスをサポートしないとしている。  

SmartShift は APU と dGPU を組み合わせたプラットフォーム向けの機能であり、全体での消費電力枠をワークロードに応じて APU, dGPU に適切に割り振ることで性能を最適化する。  
Linux Kernel における AMDGPUドライバーでは、SamrtShift のバイアスを変更するためのインターフェイスと、SmartShift下で APU, dGPU がどれだけ電力枠を使っているかを出力するインターフェイスが実装されている。  
{{< link >}} [AMD Smart Shift への対応を進める Linux AMDGPU ドライバー | Coelacanth's Dream](/posts/2021/06/07/linux-amd-smart-shift/) {{< /link >}}

SMU 12.0.{1,2} を採用する *Renoir, Lucienne, Green Sardine (Cezanne), Barcelo APU* 、SMU 13.0.{1,3} を採用する *Yellow Carp (Rembrandt) APU* ではサポートされており、SMU 13.0.5 ではそこから削除した形となっている。  
後のパッチで削除されているが、元から `#if 0 .. #endif` マクロで無効化されていた部分であり、最初から SmartShift をサポートしないことは明確に決まっていたのかもしれない。  

 > 		-#if 0
 > 		-	case METRICS_SS_APU_SHARE:
 > 		-		/* return the percentage of APU power with respect to APU's power limit.
 > 		-		 * percentage is reported, this isn't boost value. Smartshift power
 > 		-		 * boost/shift is only when the percentage is more than 100.
 > 		-		 */
 > 		-		if (metrics->StapmOpnLimit > 0)
 > 		-			*value =  (metrics->ApuPower * 100) / metrics->StapmOpnLimit;
 > 		-		else
 > 		-			*value = 0;
 > 		-		break;
 > 		-	case METRICS_SS_DGPU_SHARE:
 > 		-		/* return the percentage of dGPU power with respect to dGPU's power limit.
 > 		-		 * percentage is reported, this isn't boost value. Smartshift power
 > 		-		 * boost/shift is only when the percentage is more than 100.
 > 		-		 */
 > 		-		if ((metrics->dGpuPower > 0) &&
 > 		-		    (metrics->StapmCurrentLimit > metrics->StapmOpnLimit))
 > 		-			*value = (metrics->dGpuPower * 100) /
 > 		-				  (metrics->StapmCurrentLimit - metrics->StapmOpnLimit);
 > 		-		else
 > 		-			*value = 0;
 > 		-		break;
 > 		-#endif
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: refine smu 13.0.5 pp table code](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075959.html) {{< /quote >}}

### CPU, L3キャッシュ {#cpu}
APU に採用される SMU には CPUコア、L3キャッシュのクロック、消費電力、温度用の値がテーブル内に用意されているが、SMU 13.0.5 では現時点で当該部をコメントアウトしている。  
この部分から *Yellow Caro (Rembrandt)* は 8-Core, L3キャッシュ 1基、*VanGogh (Aerith)* は 4-Core, L3キャッシュ 1基ということを読み取ることが可能だった。  

SMU 13.0.5 がここで用いている構造体 (struct) `gpu_metrics_v2_1` は、8-Core、L3キャッシュ 2基までを想定している。[^metrics-v2_1]  
そのため、SMU 13.0.5 を採用する APU ではそれに収まらないか、先の SmartShift のことと合わせて、CPU部周りのセンサーをサポートしていないことが考えられる。  

 > 		+	/*
 > 		+	memcpy(&gpu_metrics->temperature_core[0],
 > 		+		&metrics.CoreTemperature[0],
 > 		+		sizeof(uint16_t) * 8);
 > 		+	gpu_metrics->temperature_l3[0] = metrics.L3Temperature;
 > 		+	*/
 >
 > {{< quote >}} [[PATCH 3/3] drm/amd/pm: add smu_v13_0_5_ppt implementation](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075623.html) {{< /quote >}}

[^metrics-v2_1]: [linux/kgd_pp_interface.h at 8d0749b4f83bf4768ceae45ee6a79e6e7eddfc2a · torvalds/linux](https://github.com/torvalds/linux/blob/8d0749b4f83bf4768ceae45ee6a79e6e7eddfc2a/drivers/gpu/drm/amd/include/kgd_pp_interface.h#L714-L762)

## GPUクロック {#clock}

`SMU_13_0_5_UMD_PSTATE_GFXCLK   1100` というマクロが追加されているが、これは中間的な GPUクロックを示していると思われる。  

 > 		 #define __SMU_V13_0_5_PPT_H__
 > 		
 > 		 extern void smu_v13_0_5_set_ppt_funcs(struct smu_context *smu);
 > 		+#define SMU_13_0_5_UMD_PSTATE_GFXCLK   1100
 > 		
 > 		 #endif
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: refine smu 13.0.5 pp table code](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075959.html) {{< /quote >}}

`SMU_13_0_5_UMD_PSTATE_GFXCLK` は以下引用部で使われている。  
変数 `i` は GPUクロックのレベル、`cur_value` はクロックを読み取った時点でのクロックを示していると思われる。  
変数 `i` が設定された最大 GPUクロックに一致する場合は 2、最小 GPUクロックに一致する場合は 0 が、それ以外の場合は 1 が代入され、それを元に現在のクロックレベルを示す `*` を追加で出力する。  

 > 		+	case SMU_GFXCLK:
 > 		+	case SMU_SCLK:
 > 		+		ret = smu_v13_0_5_get_current_clk_freq(smu, clk_type, &cur_value);
 > 		+		if (ret)
 > 		+			goto print_clk_out;
 > 		+		min = (smu->gfx_actual_hard_min_freq > 0) ? smu->gfx_actual_hard_min_freq : smu->gfx_default_hard_min_freq;
 > 		+		max = (smu->gfx_actual_soft_max_freq > 0) ? smu->gfx_actual_soft_max_freq : smu->gfx_default_soft_max_freq;
 > 		+		if (cur_value  == max)
 > 		+			i = 2;
 > 		+		else if (cur_value == min)
 > 		+			i = 0;
 > 		+		else
 > 		+			i = 1;
 > 		+		size += sysfs_emit_at(buf, size, "0: %uMhz %s\n", min,
 > 		+				i == 0 ? "*" : "");
 > 		+		size += sysfs_emit_at(buf, size, "1: %uMhz %s\n",
 > 		+				i == 1 ? cur_value : SMU_13_0_5_UMD_PSTATE_GFXCLK,
 > 		+				i == 1 ? "*" : "");
 > 		+		size += sysfs_emit_at(buf, size, "2: %uMhz %s\n", max,
 > 		+				i == 2 ? "*" : "");
 > 		+		break;
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: refine smu 13.0.5 pp table code](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075959.html) {{< /quote >}}

`i == 1` の場合、つまりは現在の GPUクロックが最小でも最大でもない場合、クロックレベル 1: には、現在の GPUクロックがそのまま出力される。  
そして `SMU_13_0_5_UMD_PSTATE_GFXCLK` は、三項演算子であるから、`i == 1` でない場合、GPUクロックが最小か最大の状態にあるときに出力される。  
以上のことから、中間的な、仮の GPUクロックが 1100 (MHz) ということと思われる。  

 > 		 +		size += sysfs_emit_at(buf, size, "1: %uMhz %s\n",
 > 		 +				i == 1 ? cur_value : SMU_13_0_5_UMD_PSTATE_GFXCLK,
 > 		 +				i == 1 ? "*" : "");
 >
 > {{< quote >}} [[PATCH] drm/amd/pm: refine smu 13.0.5 pp table code](https://lists.freedesktop.org/archives/amd-gfx/2022-February/075959.html) {{< /quote >}}

同様の値は *Yellow Carp (Rembrandt)* 、*VanGogh (Aerith)* にも用意されているが、どちらも SMU 13.0.5 と同じ 1100 (MHz) に設定されており、単にドライバー開発者が決めている仮クロックとも考えられる。[^yc-clk] [^vgh-clk]  

 > 		/* UMD PState Vangogh Msg Parameters in MHz */
 > 		#define VANGOGH_UMD_PSTATE_STANDARD_GFXCLK       1100
 > 		#define VANGOGH_UMD_PSTATE_STANDARD_SOCCLK       600
 > 		#define VANGOGH_UMD_PSTATE_STANDARD_FCLK         800
 > 		#define VANGOGH_UMD_PSTATE_STANDARD_VCLK         705
 > 		#define VANGOGH_UMD_PSTATE_STANDARD_DCLK         600
 >
 > {{< quote >}} [vangogh_ppt.h « smu11 « swsmu « pm « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/pm/swsmu/smu11/vangogh_ppt.h?h=next-20220225) {{< /quote >}}

[^yc-clk]: [yellow_carp_ppt.h « smu13 « swsmu « pm « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/pm/swsmu/smu13/yellow_carp_ppt.h?h=next-20220225)
[^vgh-clk]: [vangogh_ppt.h « smu11 « swsmu « pm « amd « drm « gpu « drivers - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/drivers/gpu/drm/amd/pm/swsmu/smu11/vangogh_ppt.h?h=next-20220225)
