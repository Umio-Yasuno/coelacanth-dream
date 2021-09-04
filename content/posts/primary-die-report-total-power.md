---
title: "プライマリーダイがまとめて電力を報告する Aldebaran/MI200 GPU"
date: 2021-09-04T22:35:06+09:00
draft: false
tags: [ "ROCm", "Aldebaran" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

ROCmプラットフォームにおけるテスト、ベンチマーク、認定を行うツールのコレクション [ROCm Validation Suite (RVS)](https://github.com/ROCm-Developer-Tools/ROCmValidationSuite) に、MCM (Multi-Chip Module) 構成を採る GPU かどうかを判定するコードが追加された。  

 * [Logging to identify MCM GPUs. · ROCm-Developer-Tools/ROCmValidationSuite@d9729e5](https://github.com/ROCm-Developer-Tools/ROCmValidationSuite/commit/d9729e5be460d0b7ffdc22e8fc12ec7efc882a71)

MCM構成を採る AMD GPU として、ここでは *Aldebaran/MI200* が対象となっている。また、*Aldebaran/MI200* は GPUダイ 2基で構成され、プライマリーダイとセカンダリーダイとで役割が分かれている。ダイ自体は同一ではないかと思われる。  
MCM における電力管理機能では、関連したパッチが以前 Linux Kernel の AMD GPUドライバーに投稿されていた。  
{{< link >}} [プライマリーダイとセカンダリーダイで構成される Aldebaran/MI200 GPU | Coelacanth's Dream](/posts/2021/06/09/aldebaran-primary-secondary/) {{< /link >}}
そのパッチは、電力情報の取得が可能なのがプライマリーダイのみであるため、取得と電力制限の設定をプライマリーダイのみに行うようにするという内容だった。  
今回でその電力情報がどうなっているかが補足される形となり、ROCmValidationSuite に追加されたログに出力する部分の内容にて、プライマリーダイが自身とセカンダリーダイを合わせたソケットレベルでの電力情報を報告することが明かされた。  

 > 		    /* AMD MCM GPU/s was found in the system */
 > 		    if (true == amd_mcm_gpu_found) {
 > 		
 > 			    msg_stream.str("");
 > 			    msg_stream << "Note: The system has Multi-Chip Module (MCM) GPU/s." << "\n"
 > 				    << "In MCM GPU, primary GPU die shows total socket (primary + secondary) power information." << "\n"
 > 				    << "Secondary GPU die does not have any power information associated with it independently."<< "\n";
 > 			    rvs::lp::Log(msg_stream.str(), rvs::logresults);
 > 		    }
 >
 > {{< quote >}} [Logging to identify MCM GPUs. · ROCm-Developer-Tools/ROCmValidationSuite@d9729e5](https://github.com/ROCm-Developer-Tools/ROCmValidationSuite/commit/d9729e5be460d0b7ffdc22e8fc12ec7efc882a71#diff-7426fef04fdc89a8343fc444c42c42bfb3f15c8c11f6a6e46e6b9edcc5e616ab) {{< /quote >}}

Kernel Mode Driver (KMD) である AMD GPUドライバーでプライマリーダイのみから電力情報を読み取るようにしていることから、ハードウェア側でソフトウェアからは 1つの GPU に見えるよう、より密に統合する処理を実装している可能性が考えられる。  
電力制限もプライマリーダイのみに設定されるため、ダイ間でそれぞれの温度や電力情報を低レイテンシでやり取りしていると思われる。  

ついでに追加されたコードの意味について解説を試みると、まず以下は *Aldebaran/MI200* の DeviceID (PCI ID) であり、同 ID は既に AMD GPUドライバーに記述されている。[^alde-dev_id]  
`DeviceID: 0x7410` は GPU の仮想機能を使用しているときの ID。`MAX_NUM_MCM_GPU` は単に対象とする MCM GPU の DeviceID 数を示している。  

[^alde-dev_id]: [linux/amdgpu_drv.c at 838eb73c8d5fa9bf3dcc75010b0eb819eb5bb7ed · torvalds/linux](https://github.com/torvalds/linux/blob/838eb73c8d5fa9bf3dcc75010b0eb819eb5bb7ed/drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c#L1185)

 > 		/* No of GPU devices with MCM GPU */
 > 		#define MAX_NUM_MCM_GPU 4
 > 		
 > 		/* Unique Device Ids of MCM GPUS */
 > 		static const uint16_t mcm_gpu_device_id[MAX_NUM_MCM_GPU] = {
 > 			/* Aldebaran */
 > 			0x7408,
 > 			0x740C,
 > 			0x740F,
 > 			0x7410};
 >
 > {{< quote >}} [Logging to identify MCM GPUs. · ROCm-Developer-Tools/ROCmValidationSuite@d9729e5](https://github.com/ROCm-Developer-Tools/ROCmValidationSuite/commit/d9729e5be460d0b7ffdc22e8fc12ec7efc882a71#diff-85704b00078c3d83f49dd09ee32cd2d4a2ed2f8f88e96e94d74c5e694ebe8a6b) {{< /quote >}}

上記 DeviceID は `gpu_check_if_mcm_die` 関数で使われているが、引数に取った DeviceID と比較して一致するものがあれば `true` を返すシンプルな処理。  

 > 		/**
 > 		 * @brief Check if the GPU is die (chiplet) in Multi-Core Module (MCM) GPU.
 > 		 * @param device_id GPU Device ID
 > 		 * @return true if GPU is die in MCM GPU, false if GPU is single die GPU.
 > 		 **/
 > 		bool gpu_check_if_mcm_die (uint16_t device_id) {
 > 		
 > 		  uint16_t i = 0;
 > 		  bool mcm_die = false;
 > 		
 > 		  for (i  = 0; i < MAX_NUM_MCM_GPU; i++) {
 > 		    if(mcm_gpu_device_id[i] == device_id) {
 > 		      mcm_die = true;
 > 		      break;
 > 		    }
 > 		  }
 > 		  return mcm_die;
 > 		}

完全に余談な上、プログラマーでもない自分が言うのも何だが、変数 `mcm_die, amd_mcm_gpu_found` は bool型であるのに `if (true == mcm_die)` という風にしているのは少し気になった。  
それと `gpu_check_if_mcm_die` 関数で MCM を Multi-*Core* Module の略としているが、他では Multi-*Chip* Module としているため、これは単なるタイポだろう。  

