---
title: "AMD、CDNA 3 White Paper を公開"
date: 2023-12-14T02:10:33+09:00
draft: false
categories: [ "Hardware", "APU", "GPU", "AMD" ]
tags: [ "CDNA_3", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD は先日 *CDNA 3 アーキテクチャ* を採用したデータセンター向け APU/GPU、AMD Instinct MI300 Series を発表した。  
同時に **CDNA 3 White Paper** を公開しており、そこではメディア向け資料でも触れられていない *CDNA 3 アーキテクチャ* の概要も含まれているため、一部をピックアップしていきたい。  

 * [AMD CDNA™ Architecture](https://www.amd.com/en/technologies/cdna.html)
 * [Documentation for AMD Processors, Accelerators, and Graphics](https://www.amd.com/en/search/documentation/hub.html)
   * <https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-3-white-paper.pdf>

先に書いておくと、**CDNA 3 White Paper** には **MI300A, MI300X** を *M1300A, M1300X* と誤字している箇所や **MI250X** の Vector FP32 性能を何故か Packed FP32 を使用していない場合の 128 (FLOPS/clock/CU) としていたりと問題があるが、大きいものではないと考えられるため無視する。  

**MI300A** は XCD (accelerator complex die) 6基と CCD (CPU complex die) 3基、IOD (I/O Die) または AID (Active interposer die) と呼ぶチップレット 4基、それと HBM3 スタック 8基、  
**MI300X** は XCD (accelerator complex die) 8基、IOD (AID) 4基、それと HBM3 スタック 8基で構成されている。  
IOD (AID) には XCD 2基または CCD 3基を積層することができ、I/O 部と Infinity Cache (L3 Cache, Last Level Cache) 64MiB が実装されている。また、メディアエンジンも実装されていると考えられる。  

メディアエンジン数は **MI300A** と **MI300X** で異なり、**MI300A** が 3グループなのに対し、**MI300X** は 4グループとなっている。  
**MI300A** では CCD を積層する分の IOD が持つメディアエンジンを何らかの理由で無効化しているか、XCD 用と CCD 用で IOD が異なる可能性が考えられる。  

## XCD
XCD は TSMC 5nm プロセスで製造され、物理的には CU 40基が実装されているが、有効な CU 数は最大 38基となる。  
また、L2 Cache 4MiB が実装されている。  

*CDNA 2*, MI200 Series での GCD (Graphics Compute Die) は物理的には CU 112基、有効な CU 数は最大 110基、L2 Cache 8MiB が実装されていたため、XCD は GCD より小さい規模となるが、パッケージレベルでは XCD は 6-8基 (228CU-304CU) を搭載することが可能であり、総合的には *CDNA 3* の方が多い CU 数となる。  

## Cache
*CDNA 3* では CU ごとに持つ L1ベクタキャッシュと複数の CU で共有する L1命令キャッシュが *CDNA 2* から倍増されており、L1ベクタキャッシュは 32KiB、L1命令キャッシュは 64KiB となる。  

L1命令キャッシュは CU 2基で共有する構成となっており、また帯域も向上しているとする。  
L1命令キャッシュの構成について White Paper では、実際のケースでは同じ命令ストリームを複数の CU グループで実行することがほとんどであり、(複数の CU で共有することで?) 必要とするダイエリアをほとんど一定に保ちつつ、キャッシュ可能なウィンドウサイズとヒット率を増加させることができるとしている。  

L1ベクタキャッシュのキャッシュラインサイズも 128Byte に増やされ、これによって帯域が増加している。  
加えて、L1ベクタキャッシュからコア自体へのリクエストバスと、L2 Cache からのフィルパスも拡張されている。  

従来のアーキテクチャでは L2 Cache はメモリサイドキャッシュとして配置、機能していたが、Infinity Cache の追加により役割が大きく変わった。  
L2 Cache は各 XCD におけるインターフェイスの単一ポイントとして機能し、XCD 内のメモリトラフィックを統合する。Infinity Cache へのメモリアクセスの削減を目的とし、Write Back, Write Allocate 設計のキャッシュとなる。  
L2 Cache に代わって Infinity Cache がメモリサイドキャッシュとなる。メモリサイドキャッシュであるため、それより下位レベルのキャッシュから追い出されたデータを保持することはない。  
また、Infinity Cache には複数の XCD が持つ L2 Cache をカバーするスヌープフィルターが含まれており、これにより XCD 間のコヒーレントなリクエストは、XCD 内で利用率の高い L2 Cache を邪魔することなく Infinity Cache 部でほとんど解決することができる。  

Infinity Cache は IOD に実装されており、IOD あたり 64MiB となっている。  

