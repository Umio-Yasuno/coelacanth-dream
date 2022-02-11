---
title: "RDNA 2 (gfx1030) を正式にサポートした ROCm v5.0 がリリース"
date: 2022-02-11T13:59:43+09:00
draft: false
categories: [ "Software", "AMD", "GPU" ]
tags: [ "ROCm", "RDNA_2", "gfx1030", "Sienna_Cichlid" ]
noindex: false
# summary: ""
# keywords: [ "", ]
---

ROCm v5.0 がリリースされた。  
ついに *RDNA 2* GPU のサポートが正式に追加されたが、*Sienna Cichlid/Navi21 (gfx1030)* をベースにする *Radeon Pro V620/W6800* のみのサポートとなる。  

* [RadeonOpenCompute/ROCm at roc-5.0.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-5.0.x)

HIP (Heterogeneous-Compute Interface for Portability) における統合メモリ (CPU+GPU) のサポートが強化され、`__managed__` キーワードが新たにサポートされた。  
変数の前に `__managed__` を宣言することで、CPU、GPU の両方で単一のポインタを使用してアクセスするようになる。  
メモリの割り当ては Linux Kernel の AMDGPUドライバーに実装された HMM (Heterogeneous Memory Management) によって行われる。[^hip-managed]  

[^hip-managed]: <https://github.com/RadeonOpenCompute/ROCm/tree/roc-5.0.x#managed-memory-allocation>
[^managed-sample]: [HIP/hipManagedKeyword.cpp at rocm-5.0.x · ROCm-Developer-Tools/HIP](https://github.com/ROCm-Developer-Tools/HIP/blob/rocm-5.0.x/tests/src/runtimeApi/memory/hipManagedKeyword.cpp)

ROCM-SMI ツールでは、異なる *Infinity Fabric/XGMI (Global Memory Interconnect)* 帯域を持つトポロジに対応した。[^xgmi-bw]  
*MI250X* はパッケージ内の GCD 2基と、異なるパッケージ間、どちらも XGMI で接続されるが帯域は異なるため、それを意識した対応ではないかと思われる。  
{{< link >}} [OLCF Frontier と同様のノード構成となる Crusher ―― EPYC 7A53 + 4x MI250X | Coelacanth's Dream](/posts/2022/01/14/olcf-crusher/) {{< /link >}}

[^xgmi-bw]: <https://github.com/RadeonOpenCompute/ROCm/tree/roc-5.0.x#display-xgmi-bandwidth-between-nodes>

[ROCR-Runtime](https://github.com/RadeonOpenCompute/ROCR-Runtime) では新たな環境変数 `HSA_COOP_CU_COUNT` が追加された。[^coop-cu]  
これは同期などの集合的な処理を行う単位となるスレッドグループ (Cooperative Groups) に対し、正しい使用可能な CU数を取得するためのものと思われる。  
`HSA_COOP_CU_COUNT` が 1 の場合、ROCr-Rutime は `hsa_agent_get_info()` の `HSA_AMD_AGENT_INFO_COOPERATIVE_COMPUTE_UNIT_COUNT` 属性を通して、Cooperative Groups用の正しい CU数を返す。  
1 以外か未設定の場合は、`HSA_AMD_AGENT_INFO_COMPUTE_UNIT_COUNT` と同じ CU数、おそらくは全体の CU数を返す。  
ROCr-Runtime のソースコードを見るに、*CDNA 2/MI200/Aldebaran (gfx90a)* を想定した機能となっている。GPUID は `gfx{Major, Minor, Stepping}` のフォーマットで表現される。  

{{< bq cite="[ROCR-Runtime/amd_gpu_agent.cpp at 83b46ab91086e10edbc6100d5e55cac11c9b5d7a · RadeonOpenCompute/ROCR-Runtime](https://github.com/RadeonOpenCompute/ROCR-Runtime/blob/83b46ab91086e10edbc6100d5e55cac11c9b5d7a/src/core/runtime/amd_gpu_agent.cpp#L1013-L1023)" >}}
 > 		    case HSA_AMD_AGENT_INFO_COOPERATIVE_COMPUTE_UNIT_COUNT:
 > 		      if (core::Runtime::runtime_singleton_->flag().coop_cu_count() &&
 > 		          (isa_->GetMajorVersion() == 9) && (isa_->GetMinorVersion() == 0) &&
 > 		          (isa_->GetStepping() == 10)) {
 > 		        uint32_t count = 0;
 > 		        hsa_status_t err = GetInfo((hsa_agent_info_t)HSA_AMD_AGENT_INFO_COMPUTE_UNIT_COUNT, &count);
 > 		        assert(err == HSA_STATUS_SUCCESS && "CU count query failed.");
 > 		        *((uint32_t*)value) = (count & 0xFFFFFFF8) - 8;  // value = floor(count/8)*8-8
 > 		        break;
 > 		      }
 >             return GetInfo((hsa_agent_info_t)HSA_AMD_AGENT_INFO_COMPUTE_UNIT_COUNT, value);
{{< /bq >}}

[^coop-cu]: <https://github.com/RadeonOpenCompute/ROCm/tree/roc-5.0.x#new-environment-variable>

{{< ref >}}
* [Volta-Architecture-Whitepaper-v1.1-jp.pdf](https://images.nvidia.com/content/pdf/tesla/Volta-Architecture-Whitepaper-v1.1-jp.pdf)
{{< /ref >}}

ROCm v5.0 では *CDNA 2/MI200/Aldebaran (gfx90a)* を意識した変更点がいくつかあるものの、サポートリストには追加されていない。  

それと、ROCm から少し外れるが、HIP/CUDA API を動的に読み込み、実行時に API を切り替え可能な [Orochi](https://github.com/GPUOpen-LibrariesAndSDKs/Orochi) が Apache License 2.0 下で公開された。  
Orochi を使うことで、CUDAコードを HIPコードに変換してからコンパイルする必要がなくなり、単一のバイナリを維持することができる。  

* [GPUOpen-LibrariesAndSDKs/Orochi](https://github.com/GPUOpen-LibrariesAndSDKs/Orochi)

## 曖昧なサポート状況 {#support}
まず自分が熱心な ROCmユーザーではなく、GPUプログラマーでもないことを断っておく。  

以前、ROCm v4.5 から OpenCLランタイムが *Polaris (gfx803)* をサポートしなくなったことに触れたが、ROCm v5.0 でこれといった進展は無かった。  
ROCmSupportアカウントは、意図的に (gfx803 に関する) コードを削除していないが変更した可能性はあると回答しているものの、探したところ ROCclr (Radeon Open Compute Common Language Runtime) に *GFX8* 世代の OpenCL サポートを無効化するコミットがあった。  

* [RX 470,480,570,580 no longer supports OpenCL on ROCm 4.5 · Issue #1659 · RadeonOpenCompute/ROCm](https://github.com/RadeonOpenCompute/ROCm/issues/1659)
* [SWDEV-1 - Disable OpenCL support for gfx8 in ROCm path · ROCm-Developer-Tools/ROCclr@16044d1](https://github.com/ROCm-Developer-Tools/ROCclr/commit/16044d1b30b822bb135a389c968b8365630da452)

また、*Vega10 (gfx900)* ベースのサーバー向け SKU *Instinct MI25* が EOL (End of Life) に達し、ROCm v4.5 が *Instinct MI25* をサポートする最後の公式リリースになるとされていたが、ROCm v5.0 でも何故かサポートリストに入ったままになっている。  
{{< link >}} [ROCm v4.5 がリリース ―― CPU+GPU の統合メモリをサポート、RDNA系のサポートは v5.0 に？ | Coelacanth's Dream](/posts/2021/10/30/rocm-4_5-release/#polaris-rdna) {{< /link >}}

ROCm v5.0 では既知の問題 (Known Issues) として、dGPU の VBIOS を上書きする AMDVBFlashツールが挙げられているが、自分が知っている限り Linux環境向けの AMDVBFlash は公開されていない。そして ROCm は現在 Linux系 OS しかサポートしていない。[^amdvbflash]  

[^amdvbflash]: <https://github.com/RadeonOpenCompute/ROCm/tree/roc-5.0.x#incorrect-dgpu-behavior-when-using-amdvbflash-tool>

全体的にサポート状況が曖昧という印象を受ける。  
ROCm は複数のライブラリ、ソフトウェアから構成されるプラットフォーム、ソフトウェアスタックであり、サポートリストに載っていない AMD GPU ではどこまでサポートされているかが判然としない。  
*RDNA 1/2* GPU の ROCmサポートは以前から多くに望まれており、そうした issue が複数存在するが、ROCmプラットフォーム全体でサポートしているか否かではなく、各種ツール、OpenCL/HIPランタイム、HPC向け計算ライブラリ、ML向けライブラリ、それぞれでサポート状況を示せば、サポートリストに載っていないことから発する要望やバージョン更新による混乱を減らせるのではないかと思う。  

