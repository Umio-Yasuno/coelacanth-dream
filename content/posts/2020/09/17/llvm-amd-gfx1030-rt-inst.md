---
title: "LLVM に RDNA 2 でレイトレーシングを実行する命令が追加される"
date: 2020-09-17T08:19:03+09:00
draft: false
tags: [ "RDNA_2", "Sienna_Cichlid", "Navy_Flounder" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMDGPU においてコンパイラバックエンド等へ採用されている LLVM に、*RDNA 2 /GFX10.3* でレイトレーシングを行なうための命令、`llvm.amdgcn.image.bvh.intersect.ray` が追加された。  
{{< link >}} [[AMDGPU] gfx1030 RT support · llvm/llvm-project@91f503c](https://github.com/llvm/llvm-project/commit/91f503c3af190e19974f8832871e363d232cd64c) {{< /link >}}
手法としては BVH(Bounding Volume Hierarchy, バウンディングボリューム階層構造) であり、命令はレイの交差判定を高速化させるためのものと思われる。  

テクスチャといった画像処理に用いられる `A16` アドレスから、HWレイトレーシングをサポートする *RDNA 2 /GFX10.3* GPU は Hot Chips 32 で内部構造が明になった **Xbox Series X** 同様に、テクスチャユニットとレイトレーシングユニットを共有する構成になっているのではないかと考えられる。  

今回の命令の追加により、LLVM をシェーダーコンパイラのバックエンドに一部採用しているオープンソース・ドライバー、[RADV (LLVM)](/tags/radv) での Vulkan API における HWレイトレーシングのサポートの道が見えてきた。  
独自のメモリ効率と性能に優れる [ACOバックエンド](/tags/aco) に今回の LLVM の変更を反映させれば、*ACOバックエンド* でのサポートも可能となる。  


{{< ref >}}

 * [［GDC 2018］ついにDirectXがレイトレーシングパイプラインを統合。「DirectX Raytracing」が立ち上がる](https://www.4gamer.net/games/033/G003329/20180320141/)
 * [西川善司の3DGE：新世代GPU「Turing」のリアルタイムレイトレーシングは「本物」なのか？ その正体に迫る](https://www.4gamer.net/games/121/G012181/20180816069/)

{{< /ref >}}
