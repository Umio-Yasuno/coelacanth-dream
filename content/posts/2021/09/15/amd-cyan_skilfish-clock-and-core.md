---
title: "Cyan Skilfish APU の GPUクロックと CPUコア数"
date: 2021-09-15T22:46:53+09:00
draft: false
tags: [ "RDNA", "Cyan_Skilfish", "gfx1013" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

*RDNA アーキテクチャ* APU *Cyan Skilfish* の各種クロックを、他 AMD GPU/APU 同様に取得し表示するパッチ、GPUクロックと電圧を範囲内で設定可能にするパッチが投稿された。  
{{< link >}} [HWレイトレーシングをサポートする RDNA APU 「Cyan Skilfish (gfx1013)」 | Coelacanth's Dream](/posts/2021/08/01/cyan_skilfish-apu-gfx1013/#8-core) {{< /link >}}

 * [[PATCH 1/3] drm/amdgpu: update SMU PPSMC for cyan skilfish](https://lists.freedesktop.org/archives/amd-gfx/2021-September/068809.html)
    * [[PATCH v3 2/3] drm/amdgpu: update SMU driver interface for cyan skilfish(v3)](https://lists.freedesktop.org/archives/amd-gfx/2021-September/068810.html)
    * [[PATCH v3 3/3] drm/amdgpu: add some pptable funcs for cyan skilfish(v3)](https://lists.freedesktop.org/archives/amd-gfx/2021-September/068811.html)
 * [[PATCH v3] drm/amdgpu: add manual sclk/vddc setting support for cyan skilfish(v3)](https://lists.freedesktop.org/archives/amd-gfx/2021-September/068812.html)

*Cyan Skilfish* では、GPUクロック (SCLK, GPU gfx/compute engine clock) と、電圧の目標値 (target value) が設定可能となっている。ただ、*Cyan Skilfish* は (現時点のドライバーで?) DPM (Dynamic Power Management) をサポートしておらず、設定可能な値は 1つだけとなる。  
GPUクロックは 1000 MHz - 2000 MHz、電圧は 700 mv - 1129 mv の範囲内で設定可能とされている。  


 > 		+/* unit: MHz */
 > 		+#define CYAN_SKILLFISH_SCLK_MIN			1000
 > 		+#define CYAN_SKILLFISH_SCLK_MAX			2000
 > 		+#define CYAN_SKILLFISH_SCLK_DEFAULT			1800
 > 		+
 > 		+/* unit: mV */
 > 		+#define CYAN_SKILLFISH_VDDC_MIN			700
 > 		+#define CYAN_SKILLFISH_VDDC_MAX			1129
 > 		+#define CYAN_SKILLFISH_VDDC_MAGIC			5118 // 0x13fe
 > 		+
 >
 > {{< quote >}} [[PATCH v3] drm/amdgpu: add manual sclk/vddc setting support for cyan skilfish(v3)](https://lists.freedesktop.org/archives/amd-gfx/2021-September/068812.html) {{< /quote >}}

恐らくは設定した値が実質最大 GPUクロック、電圧として動作するのだと思われる。  
GPUクロックがデフォルトで 1800 MHz、最大で 2000 MHz というのは、[AMD Radeon™ RX 5700 XT 50th Anniversary](https://www.amd.com/en/products/graphics/amd-radeon-rx-5700-xt-50th-anniversary#product-specs) の最大クロックが 1980 MHz であるから、*RDNA アーキテクチャ* らしいクロックと言えるかもしれない。  
*RDNA 2 アーキテクチャ* では高クロック動作に向けた設計により、同じ消費電力で *RDNA アーキテクチャ* より 30% 高いクロックを実現している。[^rdna_2-clk]  

[^rdna_2-clk]: [AMD Unveils Next-Generation PC Gaming with AMD Radeon™ RX 6000 Series – Bringing Leadership 4K Resolution Performance to AAA Gaming :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/978/amd-unveils-next-generation-pc-gaming-with-amd-radeon-rx)

## Cyan Skilfish の CPUコア数は 8 か 6 か {#core-count}

最近の AMD APU では、AMD GPU ドライバー側でも CPUコア、CPU L3キャッシュのクロック、温度、消費電力を取得するようになっている。  
ドライバーにはそれら読み取った値を格納する変数が用意されている訳だが、そこに使われている配列のサイズから APU のコア数を言わば逆に読み取ることができる。  
例えば *VanGogh APU (Zen 2 + RDNA 2)* であれば以下のように 4コア、L3キャッシュ 1基 (CCX 1基) という CPU構成が、  

 > 		  //3rd party tools in Windows need info in the case of APUs
 > 		  uint16_t CoreFrequency[4];     //[MHz]
 > 		  uint16_t CorePower[4];         //[mW]
 > 		  uint16_t CoreTemperature[4];   //[centi-Celsius]
 > 		  uint16_t L3Frequency[1];       //[MHz]
 > 		  uint16_t L3Temperature[1];     //[centi-Celsius]
 >
 > {{< quote >}} [linux/smu11_driver_if_vangogh.h at f4994be248b62da0411e9e0f300373f2e56efe5e · torvalds/linux](https://github.com/torvalds/linux/blob/f4994be248b62da0411e9e0f300373f2e56efe5e/drivers/gpu/drm/amd/pm/inc/smu11_driver_if_vangogh.h#L213) {{< /quote >}}

*Renoir APU (Zen 2 + Vega)* では 8コア、L3キャッシュ 2基 (CCX 2基) という構成が読み取れる。  

 > 		
 > 		  uint16_t CoreFrequency[8];            //[MHz]
 > 		  uint16_t CorePower[8];                //[mW]
 > 		  uint16_t CoreTemperature[8];          //[centi-Celsius]
 > 		  uint16_t L3Frequency[2];              //[MHz]
 > 		  uint16_t L3Temperature[2];            //[centi-Celsius]
 > 
 > {{< quote >}} [linux/smu12_driver_if.h at e098bc9612c2b60f94920461d71c92962a916e73 · torvalds/linux](https://github.com/torvalds/linux/blob/e098bc9612c2b60f94920461d71c92962a916e73/drivers/gpu/drm/amd/pm/inc/smu12_driver_if.h#L189) {{< /quote >}}

ちなみに *Yellow Carp APU (Zen 3 ? + RDNA 2)* では 8コア、L3キャッシュ 1基 (CCX 1基) となっている。[^yc-smu]  

[^yc-smu]: [linux/smu13_driver_if_yellow_carp.h at 385bb92fdc5813c5f6a8168d6bba8680f2c1d0de · torvalds/linux](https://github.com/torvalds/linux/blob/385bb92fdc5813c5f6a8168d6bba8680f2c1d0de/drivers/gpu/drm/amd/pm/inc/smu13_driver_if_yellow_carp.h#L173)

今回で *Cyan Skilfish* に対しても同様のコードが追加されたのだが、以前は存在した、8コアであることを示すような記述を削除して、6コアを想定した記述が追加されている。  
L3キャッシュは 2基 (CCX 2基) となっているため、*Cyan Skilfish APU* の CPUアーキテクチャは CCX を 4コアで構成する *Zen/+/2 アーキテクチャ* だと考えられる。コア数から、各 CCX から 1コアを無効化した 3+3 という設定だろうか。  

 > 		+typedef struct SmuMetricsTable_t {
 > 		+	//CPU status
 > 		+	uint16_t CoreFrequency[6];              //[MHz]
 > 		+	uint32_t CorePower[6];                  //[mW]
 > 		+	uint16_t CoreTemperature[6];            //[centi-Celsius]
 > 		+	uint16_t L3Frequency[2];                //[MHz]
 > 		+	uint16_t L3Temperature[2];              //[centi-Celsius]
 > 		+	uint16_t C0Residency[6];                //Percentage
 > 		
 > 		-#define NUMBER_OF_PSTATES		8
 > 		-#define NUMBER_OF_CORES			8
 >
 > {{< quote >}} [[PATCH v3 2/3] drm/amdgpu: update SMU driver interface for cyan skilfish(v3)](https://lists.freedesktop.org/archives/amd-gfx/2021-September/068810.html) {{< /quote >}}

以前は、*Cyan Skilfish* のディスプレイエンジンがドライバーでサポートされていないこと、CPU 8コアを持つこと、また *Cyan Skilfish (gfx1013)* がレイトレーシング命令をサポートしていること等から、**AMD 4700S Desktop Kit** が *Cyan Skilfish* ではないかと考えた。  
公式に発表はされていないが **AMD 4700S Desktop Kit** はパッケージ、ダイの形状から **PS5 SoC** の一部、主に GPU を無効化した CPU ではないかと言われており[^ps5-4700s]、**PS5 SoC** もまた HWレイトレーシングに対応しているため、*Cyan Skilfish・AMD 4700S・PS5* で繋がりが生まれる。  
{{< link >}} [HWレイトレーシングをサポートする RDNA APU 「Cyan Skilfish (gfx1013)」 | Coelacanth's Dream](/posts/2021/08/01/cyan_skilfish-apu-gfx1013/#8-core) {{< /link >}}
だが *Cyan Skilfish* が 6コアだとすると、その正体がまた少し戻って曖昧となる。  
**AMD 4700S** 以外に **PS5 SoC** をベースにしたモデルが存在する **可能性** もある。  

それと、**AMD 4700S** ではアーキテクチャのバックエンド部に変化があるらしく、FPU の性能が他の *Zen 2 アーキテクチャ* を採用する AMD CPU より低いらしいが、FPUパイプラインのデータ幅がどうなっているか気になる所だ。  
Zen系 CPU では FPUパイプラインのデータ幅を示す情報を `CPUID` 命令を使って読み取れるようになっており、EAXレジスタに `0x8000001A` を入れて `CPUID` 命令を実行し、EAXレジスタに出力された値の bit 2 が `1` であれば FP256 (256-bit) 、bit 0 が `1` であれば FP128 (128-bit) を示す。[^cpuid]  
*Zen/+ アーキテクチャ* では FP128、*Zen 2/3 アーキテクチャ* であれば FP256 と表示される。  
*Zen 2 アーキテクチャ* を採用する **AMD 4700S** が FP256 か FP128 のどちらだとしても、他とは部分的に異なる謎の *Zen 2? アーキテクチャ* であることに変わりはないが。  

[^cpuid]: <https://www.amd.com/system/files/TechDocs/55898_pub.zip>


[^ps5-4700s]: [PS5-Custom-Chip mit Einschränkungen: Das Ryzen 4700S Desktop Kit im Test - Hardwareluxx](https://www.hardwareluxx.de/index.php/artikel/hardware/komplettsysteme/57076-ps5-custom-chip-mit-einschraenkungen-das-ryzen-4700s-desktop-kit-im-test.html)

{{< ref >}}
 * [drm/amdgpu AMDgpu driver — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu.html)
 * Preliminary Processor Programming Reference (PPR) for AMD Family 19h Model 01h, Revision B1 Processors Volume 1 of 2
    * <https://www.amd.com/system/files/TechDocs/55898_pub.zip>
{{< /ref >}}
