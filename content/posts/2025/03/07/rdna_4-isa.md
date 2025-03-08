---
title: "AMD、RDNA 4 Instruction Set Architecture: Reference Guide を公開"
date: 2025-03-07T04:59:14+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "RDNA_4", "GFX12" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD は 2025/03/07 付で「"RDNA4" Instruction Set Architecture: Reference Guide」を公開した。  
発売のタイミングになって初めて公開された *RDNA 4 アーキテクチャ* でサポートする新命令もあり、それらを紹介していきたい。  

 * [New content released on GPUOpen for AMD RDNA™ 4 on-shelf day - AMD GPUOpen](https://gpuopen.com/learn/new_content_released_on_gpuopen_for_amd_rdna_4_on-shelf_day/)
   * ["RDNA4" Instruction Set Architecture: Reference Guide - rdna4-instruction-set-architecture.pdf](https://www.amd.com/content/dam/amd/en/documents/radeon-tech-docs/instruction-set-architectures/rdna4-instruction-set-architecture.pdf)

## Dynamic register allocation / Dynamic VGPR {#dynamic}
2025/02/28 に行われた Radeon RX 9000 シリーズの正式発表時に初めて公開された Dynamic register allocation / Dynamic VGPR だが、コンパイラバックエンドである LLVM ではまだサポートされておらず、どのように機能するかは不明だった。  
それが今回、発売のタイミングになって Dynamic VGPR の詳細と LLVM にサポートを追加するプルリクエストが公開された。  

 * [[AMDGPU] Add GFX12 S_ALLOC_VGPR instruction by rovka · Pull Request #130018 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/130018)
 * [[AMDGPU] Deallocate VGPRs before exiting in dynamic VGPR mode by rovka · Pull Request #130037 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/130037)
 * [[AMDGPU] Dynamic VGPR support for llvm.amdgcn.cs.chain by rovka · Pull Request #130094 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/130094)

まず Dynamic VGPR にはいくつかの制限があり、それはコンピュートシェーダーでのみサポートされ、グラフィクスシェーダーはサポートしないことと、Wave64 モードはサポートされず、Wave32 モードのみをサポートすることである。  
また、Dynamic VGPR を使用する Wave と使用しない Wave を混在して WGP (Workgroup Processor) または CU (Compute Unit) で実行されることはないとされている。  
いずれかの Work-group または Wave で Dynamic VGPR を使用している場合、その WGP または CU で実行できるのは Dynamic VGPR が有効な Work-group または Wave のみとなる。  
Wave32 モードでのみ Dynamic VGPR がサポートされているのは、そもそもの割り当て可能な VGPR ブロック数が Wave32 では最大 16、Wave64 では 8 (1.5倍の VGPR を持つ場合は 24、12) と違いがあるからだろうか？  
また Wave32 であれば Wave64 より早く解放されるため、Dynamic VGPR が効率的に機能するという可能性もある。  

VGPR はブロック単位で割り当てられ、最大 8ブロック、最小 1ブロックが割り当て可能。  
VGPR ブロックサイズは 16-VGPR か 32-VGPR に設定可能であり、16-VGPR の場合は 128-VGPR、32-VGPR の場合は 256-VGPR が Wave あたりに割り当て可能な最大サイズとなる。  
また、VGPR ブロックサイズはチップ全体に設定され、ディスパッチごとに変更することはできない。  

VGPR の割り当てには `S_ALLOC_VGPR` 命令を発行する必要がある。  
`S_ALLOC_VGPR` 命令の引数には定数か SGPR (スカラレジスタ) を指定可能。引数に指定するのは VGPR 数であり、それに最も近い VGPR ブロック数に切り上げて割り当てられる。  
引数が現在割り当てている VGPR 数より小さい場合は VGPR を解放する。  
もしシェーダープログラムの終わりまで達しても VGPR が解放されていない場合、ハードウェア的に `S_ALLOC_VGPR 0`、つまり確実にすべての VGPR を解放する命令が自動で挿入されるが、それは性能に悪影響する可能性があるとされている。  

Dynamic VGPR モードで実行される Wave は、1つの VGPR ブロックが割り当てられて初期化される。  
1つの VGPR ブロック (16-VGPR or 32-VGPR) に収まる範囲であれば、追加の割り当てなしでも正常に実行可能なのだろうか？  

上で書いたように `S_ALLOC_VGPR` 命令については今回初めて公開されたため、オープンソースドライバーや LLVM でサポートされ、それがリリースに含まれるまでは時間が掛かると思われる。  

## `IMAGE_BVH_DUAL_INTERSECT_RAY, IMAGE_BVH8_INTERSECT_RAY` {#bvh}
レイトレーシング系命令として *RDNA 4* では `IMAGE_BVH_DUAL_INTERSECT_RAY, IMAGE_BVH8_INTERSECT_RAY` の 2個の命令が追加された。  
これらの命令も今回初めて公開された。  

 * [[AMDGPU] Add intrinsic and MI for image_bvh_dual_intersect_ray by mariusz-sikora-at-amd · Pull Request #130038 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/130038)
 * [[AMDGPU] Support image_bvh8_intersect_ray instruction and intrinsic. by mariusz-sikora-at-amd · Pull Request #130041 · llvm/llvm-project](https://github.com/llvm/llvm-project/pull/130041)

`IMAGE_BVH_DUAL_INTERSECT_RAY` 命令は 2つの BVH4 ノードに対して、`IMAGE_BVH8_INTERSECT_RAY` 命令は 1つの BVH8 ノードに対して交差判定をテストするための命令となる。  
これらの命令の説明には、両方の交差エンジンを使用して (using both intersection engines) と書かれており、*RDNA 4* では従来のレイトレーシングユニットの内部を倍化するのではなく、2基搭載することでレイトレーシング性能を引き上げたのだろう。  

また、レイトレーシング向けのスタック管理命令として `DS_BVH_STACK_PUSH4_POP1_RTN_B32, DS_BVH_STACK_PUSH8_POP1_RTN_B32, DS_BVH_STACK_PUSH8_POP2_RTN_B64` 命令が追加された。  
`DS_BVH_STACK_PUSH4_POP1_RTN_B32` 命令は *RDNA 3/3.5* における `DS_BVH_STACK_RTN_B32` 命令に相当する。  

