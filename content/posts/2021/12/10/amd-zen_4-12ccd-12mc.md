---
title: "最大で CCD 12基、メモリコントローラ 12基となる AMD Zen 4"
date: 2021-12-10T10:53:21+09:00
draft: false
tags: [ "Zen_4", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

Linux Kernel において CPU に関わるドライバー部に、AMD の次世代プロセッサのサポートが追加され始めている。  

{{< pindex >}}
 * [CCD](#ccd)
 * [メモリコントローラ](#mc)
{{< /pindex >}}

## CCD {#ccd}

デバイスの各種センサーを読み取る HWMON (Hardware Monitoring)、その中の AMD CPU の温度センサーを読み取る `k10temp` ドライバーに、`Family 19h Models 10-1Fh, A0-AFh` のサポートを追加するパッチが、AMD のソフトウェアエンジニア [Babu Moger](https://www.linkedin.com/in/babumoger) 氏によって投稿された。  

 * [[PATCH 0/3] k10temp/amd_nb: Add support for AMD Family 19h Models 10h-1Fh and A0h-AFh](https://lore.kernel.org/linux-hwmon/163640828133.955062.18349019796157170473.stgit@bmoger-ubuntu/T/#u)

それら `x86_Model` が何のコードネームを持った AMD CPU に割り当てられているかは不明だが、新世代 (*new generation*) とは説明されている。  
おそらくは *Zen 4 アーキテクチャ* を採用する AMD の次世代サーバ向け CPU と思われ、丁度 2021/11/08 に最大 *Zen 4* 96-Core を持ち、5nmプロセスで製造される *Genoa*、同じく 5nmプロセスで製造され、電力効率と実装密度に最適化された *Zen 4c* を 128-Core 持つ *Bergamo* が発表されている。[^zen_4-zen_4c]

[^zen_4-zen_4c]: [AMD Unveils Workload-Tailored Innovations and Products at The Accelerated Data Center Premiere :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1031/amd-unveils-workload-tailored-innovations-and-products-at)

 > 		Add the new PCI Device IDs to support new generation of AMD 19h family of
 > 		processors.
 >
 > {{< quote >}} [[PATCH 0/3] k10temp/amd_nb: Add support for AMD Family 19h Models 10h-1Fh and A0h-AFh](https://lore.kernel.org/linux-hwmon/163640828133.955062.18349019796157170473.stgit@bmoger-ubuntu/T/#m8738c235c10cad6f598f7c19ff40b862d7304fe1) {{< /quote >}}

そして、従来の Zen系 CPU では CCD(Core-Complex Die) は最大 8基まで搭載可能な構成を採っており (*Zen/Zen+* では最大 4基、*Zen 2* では最大 8基)、`k10temp` ドライバーも最大 8基までをサポートしていたが、  
`Family 19h Models 10-1Fh, A0-AFh` では最大で CCD 12基の構成をサポートするとして、それに対応するためのパッチが、同じく [Babu Moger](https://www.linkedin.com/in/babumoger) 氏によって投稿された。  

 * [[PATCH 1/2] hwmon: (k10temp) Move the CCD limit info inside k10temp_data structure](https://lore.kernel.org/linux-hwmon/163770216907.777059.6947726637265961161.stgit@bmoger-ubuntu/T/#u)

 > 		The current driver can read the temperatures from upto 8 CCDs
 > 		(Core-Complex Die).
 > 		
 > 		The newer AMD Family 19h Models 10h-1Fh and A0h-AFh can support up to
 > 		12 CCDs. Update the driver to read up to 12 CCDs.
 >
 > {{< quote >}} [[PATCH 1/2] hwmon: (k10temp) Move the CCD limit info inside k10temp_data structure](https://lore.kernel.org/linux-hwmon/163770216907.777059.6947726637265961161.stgit@bmoger-ubuntu/T/#m0b7b7dc210f76c4f2e7d046ea8a48350907a0326) {{< /quote >}}

*Zen-Zen 3* では CCD あたり最大 8-Core を搭載しており、この単位をそのままに CCD を 12基とすると 96-Core、*Genoa* の最大 CPUコア数と一致する。  
CCD では L3キャッシュ 1基を共有するコア数、それとおそらくリングバス構成は *Zen 3 アーキテクチャ* から引き継ぎ、IOD (I/O Die) を CCD 12基まで対応を拡張するのだろう。  
他にも *Genoa* では DDR5、PCIe Gen5、CXL に対応することが発表されており、*Rome (Zen 2) / Milan (Zen 3)* では同じ構成のものが使われていた IOD が大きく拡張されることとなる。  

ただ *Bergamo* の 128-Core と CCD 12基という数は合わないため、*Bergamo* では CCD あたり 16-Core に増やし、CCD は 8基となるのではないかと思われる。  
ただリングバスでは構成上、経由するリングストップが増えると、同時に遅延も増えていく。  
*Zen 4c* の特徴として、密度に最適化したキャッシュ階層 (Density-Optimized Cache Hierarchy) があり[^zen_4c-cache]、Intel Atom のように L3キャッシュより上の階層、L2キャッシュを複数のコアで共有し、リングストップの数を減らした構成も *想像* できる。  

[^zen_4c-cache]: [AMD、スケーラビリティ重視の「Zen 4c」を発表 - MSがMilan-Xのベンチを公開 | TECH+](https://news.mynavi.jp/techplus/article/20211109-2182109/)

## メモリコントローラ {#mc}

メモリやキャッシュで発生したエラー、サーマルスロットリング等、ハードウェア部で発生、報告されたエラーを検知し、記録する EDAC (Error Detection And Correction) ドライバーにも、`Models 10-1Fh, A0-AFh` のサポートを追加するパッチが、Yazen Ghannam 氏によって投稿されている。  

 * [[PATCH 0/4] AMD Family 19h Models 10h-1Fh Updates](https://lore.kernel.org/linux-edac/20211208174356.1997855-1-yazen.ghannam@amd.com/T/)

RDDR5 (Registered DDR5)、LRDDR5 (Load-Reduced DDR5) のサポートが追加され、最大メモリコントローラ数が 12基となる。  

 > 		Add a new family type for AMD Family 19h Models 10h to 1Fh. Use this new
 > 		family type for Models A0h to AFh also.
 > 		
 > 		Increase the maximum number of controllers from 8 to 12.
 >
 > {{< quote >}} [[PATCH 0/4] AMD Family 19h Models 10h-1Fh Updates](https://lore.kernel.org/linux-edac/20211208174356.1997855-1-yazen.ghannam@amd.com/T/) {{< /quote >}}
 >
 > 		+	[F19_M10H_CPUS] = {
 > 		+		.ctl_name = "F19h_M10h",
 > 		+		.f0_id = PCI_DEVICE_ID_AMD_19H_M10H_DF_F0,
 > 		+		.f6_id = PCI_DEVICE_ID_AMD_19H_M10H_DF_F6,
 > 		+		.max_mcs = 12,
 > 		+		.ops = {
 > 		+			.early_channel_count	= f17_early_channel_count,
 > 		+			.dbam_to_cs		= f17_addr_mask_to_cs_size,
 > 		+		}
 > 		+	},
 >
 > {{< quote >}} [[PATCH 0/4] AMD Family 19h Models 10h-1Fh Updates](https://lore.kernel.org/linux-edac/20211208174356.1997855-1-yazen.ghannam@amd.com/T/) {{< /quote >}}

*Rome (Zen 2)* 等のチップレット構成を採る AMD CPU では、CCD-IOD間が Read: 32B/cycle、Write: 16B/cycle で接続されており、その関係か、少ない CCD数で構成された一部 SKU (EPYC 7282/7272/7252/7232P) では、メモリチャネルは 8ch ながらソケットあたりの帯域は 85.30 GB/s (4ch 2667MT/s 相当) となっている。[^epyc]
メモリ帯域に対して、搭載 CCD数が少なすぎるとそれを活用することができない。  
`Family 19h Models 10-1Fh, A0-AFh` で CCD数、メモリコントローラ数を同時に増やしたのはそうしたことが関係しているのではないかと思われる。  

モノリシック構成を採る APU では CCX-DataFabric間が Read/Write 共に 32B/cycle なのに対し[^chiplet-bw]、チップレット構成では Write を 16B/cycle に減らしていることについて AMD は、ダイ間のデータ転送による消費電力を減らすためとし、また実際のワークロードではメモリへの書き込みはあまり発生しないと説明している。[^why-write-16byte]

[^why-write-16byte]: [AMD Ryzen 3900X and 3700X (Zen2) Review | TweakTown](https://www.tweaktown.com/reviews/9051/amd-ryzen-3900x-3700x-zen2-review/index.html)
[^chiplet-bw]: [GPUOpen_Let’sBuild2020_AMD Ryzen™ Processor Software Optimization.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_AMD%20Ryzen%E2%84%A2%20Processor%20Software%20Optimization.pdf)
[^epyc]: [Processor Specifications | AMD](https://www.amd.com/en/products/specifications/processors/2316,14566,20376)
