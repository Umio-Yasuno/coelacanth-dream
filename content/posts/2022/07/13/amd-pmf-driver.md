---
title: "Linux Kernel に AMD Platform Management Framework ドライバーを追加するパッチが投稿される"
date: 2022-07-13T04:08:56+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Linux_Kernel", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の次世代プロセッサで強化されたセンサーや電力管理機能のサポートが AMD の開発者によって進められている。  
AMD の Shyam Sundar S K 氏と Mario Limonciello 氏より、Linux Kernel に AMD Platform Management Framework (PMF) ドライバーを追加するパッチが platform-driver-x86 メーリングリストに投稿された。  

 * [[PATCH v1 00/15] platform/x86/amd/pmf: Introduce AMD PMF Driver - Shyam Sundar S K](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-1-Shyam-sundar.S-k@amd.com/)

AMD PMF ドライバーの目的は、センサー情報、OS (スケジューラー) へのヒント、プラットフォーム全体の情報、APUメトリクス (各種クロックや温度、電力、電圧など) に基づいたフレームワークを提供し、性能、電力、システム全体の熱を動的に管理することにある。  
ドライバーの最初の目標には、ユーザーの振る舞いや環境に適応することで AMD PC の静音性、電力効率を引き上げ、エンドユーザーエクスペリエンスを向上させることを挙げている。  
今後のより大きい目標には、OEM によるカスタマイズを容易にし、また OEM が独自アルゴリズムとソリューションを追加するためのフレームワークを提供すること、プラットフォームデバイスのアクティブな電力管理により、スタンバイ状態および動的なプラットフォームの消費電力を最適化することを挙げている。  

今回の一連のパッチでは、Static Power Slider (SPS)、Cool and Quiet Framework (CnQF)、Auto Mode、ファンコントロールが機能に追加されている。  

## サポートするプラットフォーム {#supported}
AMD PMF ドライバーがサポートする APU プラットフォームには `AMD_CPU_ID_PS (DeviceID: 0x14e8)`、実験を目的とした *Yellow Carp/Rembrandt APU (DeviceID: 0x14b5)* が挙げられる。  
`AMD_CPU_ID_PS` は k10temp ドライバー、AMD PMC (Power Management Controller) ドライバーでもサポートが進められており、次世代の AMD プロセッサ *Family 19h Model 70h-7Fh* とも呼ぶことができる。ここでの `AMD_CPU_ID_*` は Root Port の DeviceID を指している。  
{{< link >}} [Linux Kernel で Family 19h Model 60h (CB), Family 19h Model 70h (PS) のサポートが進む | Coelacanth's Dream](/posts/2022/07/10/fam19h-60h-70h/) {{< /link >}}

 > 		+
 > 		+/* List of supported CPU ids */
 > 		+#define AMD_CPU_ID_PS			0x14e8
 > 		+
 >
 > {{< quote >}} [[PATCH v1 00/15] platform/x86/amd/pmf: Introduce AMD PMF Driver](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-1-Shyam-sundar.S-k@amd.com/T/#m1acde5ada3349464a899bbb28f30748475733eb0) {{< /quote >}}
 >
 > 		 /* List of supported CPU ids */
 > 		+#define AMD_CPU_ID_RMB			0x14b5
 > 		 #define AMD_CPU_ID_PS			0x14e8
 >
 > {{< quote >}} [[PATCH v1 14/15] platform/x86/amd/pmf: Force load driver on older supported platforms - Shyam Sundar S K](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-15-Shyam-sundar.S-k@amd.com/) {{< /quote >}}

## SPS {#sps}
SPS (Static Power Slider) は Windows OS における電源オプションに近いとしており、ユーザーは balanced/low-power/performance といったようなモードとそのモードに関連付ける温度 (STT: Skin Temperature Tracking) を設定することができる。  
モードは SMU (System Management Unit) 内部の PMFW (Power Management Firmware?) を介してシリコン側に適用される。  

 * [[PATCH v1 04/15] platform/x86/amd/pmf: Add support SPS PMF feature - Shyam Sundar S K](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-5-Shyam-sundar.S-k@amd.com/)

## CnQF {#cnqf}
CnQF (Cool and Quiet Framework) は SPS のコンセプトを拡張したものであり、ワークロードやシステムの電力傾向に基づき、電力制限とファンを動的に管理する。  
CnQF では QUIET/BALANCED/TURBO/PERFORMANCE の 4つのモードがあり、OEM はモード間の移行閾値を設定することができる。  

 * [[PATCH v1 09/15] platform/x86/amd/pmf: Add support for CnQF - Shyam Sundar S K](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-10-Shyam-sundar.S-k@amd.com/)

## Auto Mode {#auto}
Auto Mode ではシステムの消費電力を追跡し、それを基に QUIET/BALANCED/PERFORMANCE の 3つのモードを自動で切り替える。  
CnQF と Auto Mode は排他的であり、Auto Mode がサポートされている場合は Auto Mode が CnQF よりも優先される。  

 * [[PATCH v1 12/15] platform/x86/amd/pmf: Add support for Auto mode feature - Shyam Sundar S K](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-13-Shyam-sundar.S-k@amd.com/)

AMD PMF ドライバーは PMFW から APUメトリクステーブルを定期的に読み取り、システムの振る舞いや消費電力を理解するのに用いる。読み取る間隔はカーネルパラメーターから設定可能。  

テーブルには `apu_power` と `dgpu_power` の情報があり、それらから大体のシステム電力を読み取ることができる。  
また `core_freq[8]` などから最大 CPU 8-Core、`l3_freq` などが配列でないことから CCX 1基構成の APU を想定していると思われる。  

 > 		+struct smu_pmf_metrics {
 > 		+	u16 gfxclk_freq; /* in MHz */
 > 		+	u16 socclk_freq; /* in MHz */
 > 		+	u16 vclk_freq; /* in MHz */
 > 		+	u16 dclk_freq; /* in MHz */
 > 		+	u16 memclk_freq; /* in MHz */
 > 		+	u16 spare;
 > 		+	u16 gfx_activity; /* in Centi */
 > 		+	u16 uvd_activity; /* in Centi */
 > 		+	u16 voltage[2]; /* in mV */
 > 		+	u16 currents[2]; /* in mA */
 > 		+	u16 power[2];/* in mW */
 > 		+	u16 core_freq[8]; /* in MHz */
 > 		+	u16 core_power[8]; /* in mW */
 > 		+	u16 core_temp[8]; /* in centi-Celsius */
 > 		+	u16 l3_freq; /* in MHz */
 > 		+	u16 l3_temp; /* in centi-Celsius */
 > 		+	u16 gfx_temp; /* in centi-Celsius */
 > 		+	u16 soc_temp; /* in centi-Celsius */
 > 		+	u16 throttler_status;
 > 		+	u16 current_socketpower; /* in mW */
 > 		+	u16 stapm_orig_limit; /* in W */
 > 		+	u16 stapm_cur_limit; /* in W */
 > 		+	u32 apu_power; /* in mW */
 > 		+	u32 dgpu_power; /* in mW */
 > 		+	u16 vdd_tdc_val; /* in mA */
 > 		+	u16 soc_tdc_val; /* in mA */
 > 		+	u16 vdd_edc_val; /* in mA */
 > 		+	u16 soc_edcv_al; /* in mA */
 > 		+	u16 infra_cpu_maxfreq; /* in MHz */
 > 		+	u16 infra_gfx_maxfreq; /* in MHz */
 > 		+	u16 skin_temp; /* in centi-Celsius */
 > 		+	u16 device_state;
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH v1 08/15] platform/x86/amd/pmf: Get performance metrics from PMFW - Shyam Sundar S K](https://lore.kernel.org/platform-driver-x86/20220712145847.3438544-9-Shyam-sundar.S-k@amd.com/) {{< /quote >}}

私見だが、Intel はハイブリッドアーキテクチャとそれを補助する HFI/EHFI などの機能でワークロードに対する消費電力を最適化を試みているのに対し、AMD は SMU の管理機能を強化することで CPU だけでなくシステム全体で消費電力の最適化を試みているように見える。  
{{< link >}} [Linux Kernel に Intel HFI をサポートするパッチ | Coelacanth's Dream](h/posts/2022/01/02/intel-hfi/) {{< /link >}}
どちらが優れているかはともかく、どちらを選んでもユーザーは恩恵を受けられる。  
