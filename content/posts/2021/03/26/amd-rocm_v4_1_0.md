---
title: "AMD、ROCm v4.1 をリリース"
date: 2021-03-26T11:28:47+09:00
draft: false
tags: [ "ROCm", "Arcturus" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD は 2021/03/24 に ROCm v4.1 をリリースした。  
*CDNAアーキテクチャ* を採用する **Arcturus/MI100** を正式にサポートした ROCm v4.0 のリリースが 2020/12/19 で、そこから少し間が空いたが、ROCm v4.0 では当初から目的としてあった、機械学習、HPC向けの機能を満たすソフトウェアスタックとなることを達成したため、リリース間隔が緩やかになったのだろう。  
{{< link >}} [AMD、MI100 を正式にサポートした ROCm v4.0.0 をリリース & RDNA/GFX10系は 2021年にサポート予定 | Coelacanth's Dream](/posts/2020/12/19/amd-rocm-v4_0/) {{< /link >}}

 * [RadeonOpenCompute/ROCm at rocm-4.1.0](https://github.com/RadeonOpenCompute/ROCm/tree/rocm-4.1.0#Whats-New-in-This-Release)

ROCm v4.1 では、新たな TargetID 機能をサポートした。これにより各 AMD GPUアーキテクチャに割り当てられた GPUID (GFXID) やそれが持つ切り替え可能な機能を指定し、特定の GPU、機能に向けたコードをコンパイルすることができるようになる。  
切り替え可能な機能には、CU内の各SRAM {{< comple >}} ベクタレジスタ、スカラレジスタ、LDS (Local Data Share)、L1キャッシュ {{< /comple >}} の ECC を有効にする `SRAM ECC`、  
ページフォールトが発生した時の、物理メモリを仮想メモリに割り当てる Demand Paging (デマンドページング) やページの移行を行なう Page Migration に使われる `XNACK` 機能が存在する。  
{{< link >}} [Linux Kernel に CPU + GPU の統合メモリ空間をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/07/add-svm-to-amdgpu-kfd/) {{< /link >}}

他には RDC (ROCm Data Center) Toolや各種ライブラリのサポート強化、HIP や OpemMPオフロードの利便性向上、バグ修正等が行われている。  
また、これは ROCm v4.0 からだが、ROCmソフトウェアで *公式的に* サポートする GPU は *Vega/GFX9* 系と *CDNA (Arcturus/MI100)* のみで、*Polaris* 系等の *GFX8* は動作はするが完全なサポートは保証しないことが明言されている。  
*GFX8* では `XNACK` 機能や仮想メモリの 49-bitアドレッシング、FP16パックド実行等、今後の HPC に必要な機能をサポートしていないため、納得はいくが、*Vega/GFX9 アーキテクチャ* を採用する GPU (**Vega56/64, VII**) は現在一般市場にはほとんど出回っていないため、*RDNA/GFX10 アーキテクチャ* の正式サポートが望まれる。  
AMD は 2021年に *RDNA/GFX10アーキテクチャ* をサポートすると発表しているが、具体的なスケジュールは出されておらず、  
また、2021年にはエクサスケールスパコン *Frontier* の納入や、**Aldebaran/MI200** を搭載する *LUMI* の一部が稼働予定にあり、AMD としてはそちらのサポートを優先したいことが予想できる。  
{{< link >}} [MI200 は Frontier、LUMI に採用され、LUMI は 2021年半ばから運用開始予定 | Coelacanth's Dream](/posts/2021/02/22/mi200-hpc-system/) {{< /link >}}

それと、以前から ROCm はアップグレードによるインストールをサポートせず、クリーンインストールを推奨しているが、これは本当で、自分が ROCm v4.1 を {{< comple >}} OpenCL周りのパッケージだけだが {{< /comple >}} インストールした時、アップグレードではうまく認識しなかったのが、一度パッケージを削除して再度インストールしたところすんなり認識した。  

