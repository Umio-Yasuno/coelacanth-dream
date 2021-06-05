---
title: "Aldebaran/gfx90a への対応を進める ROCmソフトウェア"
date: 2021-04-15T21:15:11+09:00
draft: false
tags: [ "ROCm", "Aldebaran", "gfx90a", "MI200", "LLVM" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

コードネーム *Aldebaran (GPUID: gfx90a)* は *CDNA 2 アーキテクチャ* を GPUアーキテクチャに採り、スーパーコンピューター *Frontier* や *LUMI* に採用されると見られ、  
{{< link >}} [MI200 は Frontier、LUMI に採用され、LUMI は 2021年半ばから運用開始予定 | Coelacanth's Dream](/posts/2021/02/22/mi200-hpc-system/) {{< /link >}}
AMD GPU のオープンソースソフトウェアスタック **ROCm** では既に *Aldebaran/gfx90a* への対応、最適化作業が進められている。  

 * [Initial implementation support for aldebaran by yoichiyoshida · Pull Request #1300 · ROCmSoftwarePlatform/Tensile](https://github.com/ROCmSoftwarePlatform/Tensile/pull/1300)
 * [Add gfx90a to AMDGPU_TARGETS if supported by cgmb · Pull Request #240 · ROCmSoftwarePlatform/rocSOLVER](https://github.com/ROCmSoftwarePlatform/rocSOLVER/pull/240)
 * [Enable xldops igmmm on gfx90a by zjing14 · Pull Request #836 · ROCmSoftwarePlatform/MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen/pull/836)
 * [Add gfx90a targets (#203) · ROCmSoftwarePlatform/rocSPARSE@7e91c24](https://github.com/ROCmSoftwarePlatform/rocSPARSE/commit/7e91c24f3decb314b0fdf13d4dd3ec7fd7ec7bd6)
 * [Add gfx90a target (#404) · ROCmSoftwarePlatform/rocFFT@c005983](https://github.com/ROCmSoftwarePlatform/rocFFT/commit/c00598360d340ea85b52ae3fe6c9b4228964f651)

## Aldebaran/gfx90a のメモリモデル {#memory-model}

コンパイラバックエンドである LLVM に *Aldebaran/gfx90a* のメモリモデルについても解説が一部追加された。  

 * [[AMDGPU] Update gfx90a memory model support · ROCm-Developer-Tools/llvm-project@4658cd4](https://github.com/ROCm-Developer-Tools/llvm-project/commit/4658cd4c18ba73257a8642fd757c2124ad840204)

これまでの AMD GPU は基本的に CPU から独立したメモリを持つ dGPU と、CPU とメモリを共用する APU に分かれていた。  
しかし *Aldebaran/gfx90a* では CPU との *XGMI/Infinity Fabric* 接続に対応し、メモリ空間の統合が実現するとされ、そうした枠組みからは外れる。  
さらにはマルチダイ構成を取るとも考えられ、ある GPUダイから見て、近い (ローカル) メモリと片方の GPUダイに接続された遠い (リモート) メモリが存在することになる。  
{{< link >}} [Aldebaran GPU は 2-Die 構成か | Coelacanth's Dream](/posts/2021/03/05/amd-aldebaran-2die-mcm/) {{< /link >}}
追加された部分では、そうした *Aldebaran/gfx90a* におけるコヒーレント保つため、メモリサイドキャッシュである L2キャッシュの使用、あるいは無効化についてどのように制御されるかが解説されている。  

ある GPU から近いメモリに対してのメモリアクセスはキャッシュされ、一貫性を保つためにキャッシュの無効化やライトバックを行う必要はない。遠いメモリに対してはキャッシュせずに直接メモリアクセスするため、同様に特別な操作は行わない。  
CPU から GPU への XGMIリンクを用いたメモリアクセスは CPU側にはキャッシュされる場合があるが、GPU から CPU へのアクセスはキャッシュしないことで一貫性を保つとしている。  

## 8x Aldebaran のトポロジ構成? 

マルチGPU環境におけるコミュニケーションライブラリ [RCCL (ROCm Communication Collectives Library)](https://github.com/ROCmSoftwarePlatform/rccl) に *Aldebaran/gfx90a* 8基のトポロジ構成一例が追加された。  

 * [Add gfx90a target (#344) · ROCmSoftwarePlatform/rccl@1fe0314](https://github.com/ROCmSoftwarePlatform/rccl/commit/1fe031402a044e3cc4652d11d512015b5c426810)

そのトポロジ構成では、*AMD Milan* に *Aldebaran/gfx90a* 8基を PCIe Gen4x16 で接続するシングルノードの構成を採っている。  
一応補足すると、RCCL では CPU を、*AMD Rome* を familyID: 143、modelID: 49 以上、*AMD Milan* は familyID: 175 と表現している。familyID については元の CPU Family を 0xX0 + 0xF としているものと思われ、*AMD Rome* は Family: 0x17 (23) であるから、0x8 と 0xF に分解、オフセットを追加し 0x80 (128) + 0xF (15) で familyID: 143 となる。  
GPUID についても、*gfx90a* ではなく *gcn="910"* となっているが、GPUID は Major, Minor, Stepping の 3つで表現されており、省略せずに 10進数で記述すると *{9, 0, 10}* という形になる。  
{{< link >}} [AMD GPU の GPU ID は何を意味するか | Coelacanth's Dream](/posts/2020/06/22/amdgpu-gpuid-mean/) {{< /link >}}

そして GPU同士リンク *XGMI/Infinity Fabric* は例では以下の表のようになっている。  

| XGMI<br>topo | c1 | c5 | c9 | cd | d1 | d5 | d9 | dd |
| :--  |:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| c1   |  - | +  | +  |    |    |    |  + |    |
| c5   | +  | -  |    |  + |    | +  |    |    |
| c9   | +  |    |  - |  + |  + |    |    |    |
| cd   |    | +  | +  |  - |    |    |    | +  |
| d1   |    |    | +  |    | -  | +  |  + |    |
| d5   |    | +  |    |    | +  | -  |    | +  |
| d9   | +  |    |    |    | +  |    |  - | +  |
| dd   |    |    |    | +  |    | +  |  + | -  |

表形式ではわかりにくいため図式したのが以下。GPU 1基あたり XGMI 3-リンク持っている。  
Linux Kernel への *Aldebaran* 関連パッチから XGMI に最適化された SDMAエンジン数は 3基であることがわかっており、リンク数はそれと一致する。  
{{< link >}} [Linux Kernel に 「Aldebaran」 GPU をサポートするパッチが投稿される　―― CDNA 2/MI200 のコードネーム? | Coelacanth's Dream](/posts/2021/02/25/amd-aldebaran-gpu/#sdma) {{< /link >}}
*Arcturus/MI100/gfx908* では XGMI に最適化された SDMAエンジン数は 6基であったから、8-GPU でもほとんどの GPU とは直接繋がり、他 GPU を経由しなければならない GPU が 1基だけ存在する、というネットワーク構成が可能だった。  
*Aldebaran* は 3基であるから、一番遠い GPU には GPU 3基を経由する必要が出てしまう。  
だが同時に *Aldebaran* は 2-ダイで構成されるMCMと考えられており、それぞれが独立していると仮定すると下側のようなネットワークになる。  
これならダイ間の経由であれば高速に通信を行うことができ、上側よりは通信による性能への影響を減らせると考えられる。  

しかし *Aldebaran* 8基がそれぞれ *AMD Milan* と PCIe Gen4x16 で接続されているように記述されている点は気になる。  
*AMD Milan* は 1基では最大 PCIe Gen4 128-Lane を持つため、NIC との接続にも一部レーンを使用している今回の例では数が合わないように思える。よって、実際どうかは当然不明だが、下側のような構成を取っている *可能性* は考えられる。  


{{< dia >}}
```
   +----+   +----+  
  /| c1 |===| c5 |\ 
 / +----+   +----+ \ 
 |   ||       ||   | 
 | +----+   +----+ | 
 | | c9 |===| cd |\| 
 | +----+   +----+ | 
 |   |             |\
 |   |             ||
 | +----+   +----+ /|
 | | d1 |===| d5 |/ |
 | +----+   +----+  |
 |   ||       ||    |
 \ +----+   +----+ / 
  \| d9 |===| dd |/ 
   +----+   +----+   

/* ? */
 +-------+___+-------+
 | c1/c9 |___| d1/d9 |
 +-------+   +-------+
    | |         | |
 +-------+___+-------+
 | c5/cd |___| d5/dd |
 +-------+   +-------+
```
{{< /dia >}}

