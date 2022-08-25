---
title: "過去の AMD GPU コードネームに関するメモ ―― Tiran, Maui, Barmuda, Pirate Islands"
date: 2022-08-21T04:41:11+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU", "Note" ]
tags: [ "GFX8", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

結果的に世に出ることはなかった、オープンソースドライバーにサポートが追加されなかった過去の AMD GPU のコードネームに関するメモ。  

* [GPUOpen-Archive/CodeXL: CodeXL is a comprehensive tool suite that enables developers to harness the benefits of CPUs, GPUs and APUs.](https://github.com/GPUOpen-Archive/CodeXL)

上記レポジトリの `CodeXL/Components/ShaderAnalyzer/AMDTBackEnd/Include/Common/asic_reg` 下には、*GFX7-8* 世代の AMD GPU の `FamilyID` や ASIC/Chip に対する `RevisionID (RevID)` や製品に対する `DeviceID` を記述したヘッダファイルが置かれている。  
それら ID はオープンソースドライバーにおいても用いられているため、照らし合わせることができ、現在サポートされているか否かを判断できる。  

## GFX7, Sea Islands (CI) {#ci}
以下は *GFX7 /Sea Islands (CI)* 世代の ASIC RevID だが、自分にとっては見慣れないコードネームが 2つ、*Tiran* と *Maui* がある。  
どちらもオープンソースドライバーにサポートは追加されていない。  

 > 		enum
 > 		{
 > 		    CI_TIRAN_P_A0   = 1,  // Tiran is obsolete, please do not use
 > 		
 > 		    CI_BONAIRE_M_A0 = 20,
 > 		    CI_BONAIRE_M_A1 = 21,
 > 		
 > 		    CI_HAWAII_P_A0  = 40,
 > 		
 > 		    CI_MAUI_P_A0    = 60,
 > 		
 > 		    CI_UNKNOWN      = 0xFF
 > 		};
 >
 > {{< quote >}} [CodeXL/ci_id.h at master · GPUOpen-Archive/CodeXL](https://github.com/GPUOpen-Archive/CodeXL/blob/master/CodeXL/Components/ShaderAnalyzer/AMDTBackEnd/Include/Common/asic_reg/ci_id.h) {{< /quote >}}

### Tiran {#tiran}
*Tiran* はコメント部にあるように、記述はあるが使用することはない ID とされており、DeviceID を `0x6600` とする記述もあるが、それは *GFX6 /Southern Islands (SI)* 世代の *Oland* に割り当てられた DeviceID であるため、製品化の予定は元から、あるいは早くから無かったのではないかと推察される。  

しかし検索エンジンを頼ると、わずかではあるが *Tiran* について AMD が発表していた情報とそれを取り上げた記事が見つかった。  
テクノロジー関連のニュースを取り扱っている [SemiAccurate](https://www.semiaccurate.com/) で Charlie Demerjian 氏が 2013/05/20 に書いた記事に、AMD/ATI の 「GPU Tape out Schedule」と題したスライドが掲載されている。  
そこにはパフォーマンスセグメントに位置する *Sea Islands* として *Tiran (stacked die)* が記載されている。[^tiran-stacked-die]  
HBM のような DRAMダイを積層したメモリを採用する GPU として *Tiran* が計画されていたのだろう。  
その後 *Tiran* について言及した記事を見つけられなかった。  

[^tiran-stacked-die]: [Nvidia GPU roadmaps aren't an improvement with Volta - SemiAccurate](https://www.semiaccurate.com/2013/05/20/nvidias-volta-gpu-raises-serious-red-flags-for-the-company/)

そして 2015年に初めて HBM メモリを採用した AMD GPU 製品として、コードネーム *Fiji* が発表される。  
AMD は同年の HotChips 27 にて *Fiji* と HBM メモリの講演をしており、そこでは Interposer を介して GPUダイと DRAM を接続したプロトタイプが 3種類掲載されている。  
内 2種類は GPUダイが *RV635 (GFX3 /R600)*、*Cypress (GFX4 /Evergreen)* だと注釈があるのだが、*Fiji Replica* の前段となるプロトタイプ GPU にはない。  
その GPU は GPUダイが 502mm2、Interposer は 818mm2 だとされており、*Fiji Replica* は GPUダイ 592mm2、Interposer 1011mm2 であるため明らかに異なる。  
そのプロトタイプ GPU が *Tiran* ではないのだろうか。GPUダイと DRAM の積層とダイサイズが巨大なことは、先に触れたスライドの内容と一致する。  

[^hc27-fiji]: [COMPUTEX 2015 - HC27.25.520-Fury-Macri-AMD-GPU2.pdf](https://old.hotchips.org/wp-content/uploads/hc_archives/hc27/HC27.25-Tuesday-Epub/HC27.25.50-GPU-Epub/HC27.25.520-Fury-Macri-AMD-GPU2.pdf)

*Tiran* が製品として出なかった理由は不明。CodeXL にあるファイルは最初のコミットが 2016/04/20 であり、*Tiran* の名が出てからだいぶ時間が経ってから公開されたものである。  
初期は製品化を目標としていたが途中でキャンセルされた可能性もある。  
製造コストに対する性能、GPUダイ側の性能が不十分とされ、1世代を経て機能が強化され、同時に GPUダイサイズをさらに巨大化できるくらいには製造コストが下げられた *Fiji* が製品として登場したのではないか、という想像はできるが、あくまでもオタクの想像である。  

### Maui {#maui}
*Maui* はパフォーマンスセグメントに位置する GPU とされてはいるが、そういったコードネームがある以上の情報は見つけられなかった。  
先に触れたスライドにも *Maui* の名はない。  

それなりに情報が見つかったのは *Tiran* のみで、この後も同じような展開が続く。  

 > 		//
 > 		// MAUI device IDs (Performance segment)
 > 		//
 > 		#define DEVICE_ID_CI_MAUI_P_66E0                0x66E0  // unfused
 > 		
 >
 > {{< quote >}} [CodeXL/ci_id.h at master · GPUOpen-Archive/CodeXL](https://github.com/GPUOpen-Archive/CodeXL/blob/master/CodeXL/Components/ShaderAnalyzer/AMDTBackEnd/Include/Common/asic_reg/ci_id.h) {{< /quote >}}

## GFX8 /Volcanic Islands (VI) {#vi}
### Bermuda {#bermuda}
*Bermuda* は *Tonga, Fiji, Ellesmere (Polaris10)* 以外に *GFX8* 世代のパフォーマンスセグメントに位置する GPU とされている。  
これもコードネームがあった以上の情報はない。  
ASIC RevisionID は再度割り当てられることは基本なく、開発初期に決められるものと思われる。  
そのことから *Maui* も *Bermuda* も計画されてはいたが、開発途中でキャンセルされたのだろう。ちょうどそれら AMD GPU の世代はプロセス移行の時期にあったため、製造プロセスが影響した可能性は考えられる。  

 > 		enum {
 > 		    VI_ICELAND_M_A0   = 1,
 > 		
 > 		    VI_TONGA_P_A0     = 20,
 > 		    VI_TONGA_P_A1     = 21,
 > 		
 > 		    VI_BERMUDA_P_A0   = 40,
 > 		
 > 		    VI_FIJI_P_A0      = 60,
 > 		
 > 		    VI_ELLESMERE_P_A0 = 80,
 > 		    VI_ELLESMERE_P_A1 = 81,
 > 		
 > 		    VI_BAFFIN_M_A0    = 90,
 > 		    VI_BAFFIN_M_A1    = 91,
 > 		
 > 			VI_LEXA_V_A0      = 0xA0,
 > 		
 > 		    VI_UNKNOWN        = 0xFF
 > 		};
 >
 > {{< quote >}} [CodeXL/vi_id.h at master · GPUOpen-Archive/CodeXL](https://github.com/GPUOpen-Archive/CodeXL/blob/master/CodeXL/Components/ShaderAnalyzer/AMDTBackEnd/Include/Common/asic_reg/vi_id.h) {{< /quote >}}

## Pirate Islands {#pi}
*Pirate Islands* は GPUダイではなく、GPUファミリーに付けられたコードネーム。  
これもあまり情報はない。当時の情報というよりは噂では *Bermuda, Fiji, Treasure Islands* が *Pirate Islands* ファミリーとしてあったらしいが、*Bermuda* と *Fiji* は *GFX8 /Volcanic Islands (VI)* ファミリーとなっており、*Treasure Islands* は CodeXL に記述がない。[^pi]  
*Volcanic Islands* ファミリーと *Carrizo* ファミリーの後に位置するが、それだけである。  
しかし `FamilyID` は確かに割り当てられており、今も他に割り当てられてはない。  
実際に製品が登場した *Carrizo* ファミリーの後の GPU ファミリーは *GFX9 /Arctic Islands (AI)* となる。*Arctic Islands (AI)* の `FamilyID` は `141`。  

*GFX8* と *GFX9* の中間に位置する *Pirate Islands* がどんな計画だったかは、まったく不明となっている。  

 > 		#define FAMILY_VI                      130          // Volcanic Islands: Iceland (V), Tonga (M)
 > 		
 > 		#define FAMILY_CZ                      135          // Carrizo, Nolan, Amur
 > 		
 > 		#define FAMILY_PI                      140          // Pirate Islands
 >
 > {{< quote >}} [CodeXL/atiid.h at master · GPUOpen-Archive/CodeXL](https://github.com/GPUOpen-Archive/CodeXL/blob/master/CodeXL/Components/ShaderAnalyzer/AMDTBackEnd/Include/Common/asic_reg/atiid.h#L142-L146) {{< /quote >}}

[^pi]: [ASCII.jp：Pirate Islandsは今秋登場か？　AMDのGPUロードマップ (3/3)](https://ascii.jp/elem/000/000/873/873659/3/)

{{< ref >}}
 * [RadeonFeature](https://www.x.org/wiki/RadeonFeature/)
 * [src/amd/addrlib/src/amdgpu_asic_addr.h · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/main/src/amd/addrlib/src/amdgpu_asic_addr.h)
 * [ASCII.jp：20nmプロセスへの移行を着実に進めるAMDのGPUロードマップ (1/2)](https://ascii.jp/elem/000/000/932/932940/)
 * [ASCII.jp：20nmが白紙になり28nmで再構築するNVIDIAのGPUロードマップ (1/3)](https://ascii.jp/elem/000/000/930/930502/)
 
 * *Tiran*
    * [Nvidia GPU roadmaps aren't an improvement with Volta - SemiAccurate](https://www.semiaccurate.com/2013/05/20/nvidias-volta-gpu-raises-serious-red-flags-for-the-company/)
    * [COMPUTEX 2015 - HC27.25.520-Fury-Macri-AMD-GPU2.pdf](https://old.hotchips.org/wp-content/uploads/hc_archives/hc27/HC27.25-Tuesday-Epub/HC27.25.50-GPU-Epub/HC27.25.520-Fury-Macri-AMD-GPU2.pdf)
    * [AMD Hawaii Radeon R9 290X architektura rozbor - Sea Islands, Volcanic Islands a zvraty v roadmapě | Diit.cz](https://diit.cz/clanek/amd-hawaii-radeon-r9-290x-architektura-rozbor/sea-islands-volcanic-islands)
    * [Radeon Fury Nano fotografie PCB | Diit.cz](https://diit.cz/clanek/radeon-fury-nano-fotografie-pcb)
{{< /ref >}}
