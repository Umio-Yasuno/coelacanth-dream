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

`llvm.amdgcn.image.bvh.intersect.ray` は MIMG(Memory Image Instructions) であり、アドレスフィールドには `A16` 要素が用いられ、これはテクスチャサンプラーを使わない画像処理では `uint16` 、サンプラーを使うものでは `f16` となる。  
このことから、HWレイトレーシングをサポートする *RDNA 2 /GFX10.3* GPU は Hot Chips 32 で内部構造が明になった **Xbox Series X** 同様に、テクスチャユニットとレイトレーシングユニットを共有する構成になっているのではないかと考えられる。  

今回の命令の追加により、LLVM をシェーダーコンパイラのバックエンドに一部採用しているオープンソース・ドライバー [RADV (LLVM)](/tags/radv)、*AMDVLK* での Vulkan API における HWレイトレーシングのサポートの道が開けてきた。  
独自のメモリ効率と性能に優れる [ACOバックエンド](/tags/aco) に今回の LLVM の変更を反映させれば、*ACOバックエンド* でのサポートも可能となる。  

余談だが、LLVM 無しに *RADVドライバー* をビルド可能にするマージリクエストが Mesa に投稿されたことがあった。  
*ACOバックエンド* は LLVMバックエンド同等の機能を備え、かつ性能に優れるからというのが理由だったが、ある理由により、マージリクエストは組み込まれずに閉じられた。  
理由の1つはデバッグのためであり、安定バージョンを用いてるユーザーに起こった問題が LLVM と *ACO* のどちらに起因するものか検証が面倒になるというもの。  
もう1つは、AMD が将来的な GPU ISA を *ACO* の開発チームと共有するかが不透明であり、最新の ISA に対応するため、LLVM は残す必要があるというものだった。  
AMD が公式にリリースする *AMDVLK* は LLVMバックエンドを用いている。*ACOバックエンド* は性能に優れ、ユーザーからも好意的に受け入れられているが、AMD がどのように捉えているかは不明であり、将来的にどうなるか、取り入れるつもりがあるのかははっきりしない。  

AMD は *RDNA 2* に関する発表を 2020/10/28 に行なうと発表しており <del>(発表の発表)</del>、それに向けてか[Sienna CIchlid が USB-Cコントローラーを内蔵](/posts/2020/09/11/amd-sienna_cichlid-usb_c/) や [VCN 3 の AV1 HWデコードサポート](/posts/2020/09/16/amd-vcn_3-av1-dec/)等、オープンソース・ソフトウェアに *RDNA 2 /GFX10.3* のより詳細なパッチを投稿している。  
発表日が近づくにつれ、この動きは加速すると見られる。  
パッチから部分的な姿を想像しつつ、盛大な正式発表を心待ちにしている。  

{{< ref >}}

 * [［GDC 2018］ついにDirectXがレイトレーシングパイプラインを統合。「DirectX Raytracing」が立ち上がる](https://www.4gamer.net/games/033/G003329/20180320141/)
 * [西川善司の3DGE：新世代GPU「Turing」のリアルタイムレイトレーシングは「本物」なのか？ その正体に迫る](https://www.4gamer.net/games/121/G012181/20180816069/)
 * [RDNA_Shader_ISA.pdf](https://developer.amd.com/wp-content/resources/RDNA_Shader_ISA.pdf)

{{< /ref >}}
