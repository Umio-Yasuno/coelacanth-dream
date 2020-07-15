---
title: "AMD、メモリ 8ch、PCIe 128レーンに対応した Threadripper Pro 4モデルを発表"
date: 2020-07-15T02:15:50+09:00
draft: false
tags: [ "Zen_2", "Threadripper" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
---

―― ただし一部モデルのメモリ帯域は4chまで  

AMD は、2020/07/14付で、PCIe Gen4 128レーンを備え、メモリは 8チャネル ECC RIMM、LRIMM、UDIMM DDR4-3200、容量 2TB まで対応した、*Zen 2 アーキテクチャ* 採用のワークステーション向けプロセッサ、**Ryzen Threadripper Pro** を 4モデル発表した。  
メモリのデータを暗号化する **AMD Memory Guard** にも対応し、ビジネス向けのサポートも付く。  
OEM向けとされ、製品は [Lenovo ThinkStation P620](https://thinkstation-specs.com/thinkstation/p620/) に搭載されて登場する。  
{{< link >}} [AMD Announce World’s First 64-Core PRO Workstation, the Lenovo™ ThinkStation™ P620, the Pinnacle of Performance for Modern Professionals | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-announce-worlds-first-64-core-pro-workstation-lenovotm) {{< /link >}}

| Model | Core/Thread | Base/Boost Clock | Total Cache | TDP | PCIe Gen4 |
| :-- | :--: | :--: | :--: | :--: | :--: |
| Ryzen Threadripper<br>Pro 3995WX | 64/128 | 2.7/4.2 GHz | 288 MB | 280 W | 128-Lane |
| Ryzen Threadripper<br>Pro 3975WX | 32/64 | 3.5/4.2 GHz | 144 MB | 280 W | 128-Lane |
| Ryzen Threadripper<br>Pro 3955WX | 16/32 | 3.9/4.3 GHz | 72 MB | 280 W | 128-Lane |
| Ryzen Threadripper<br>Pro 3945WX | 12/24 | 4.0/4.3 GHz | 70 MB | 280 W | 128-Lane |

## 帯域は 4ch相当までとなる Pro 3955WX と Pro 3945WX
**EPYC 7002シリーズ** 同等のメモリと PCIe Gen4レーンを持ち、サーバ向けI/Oダイ(sIOD) とソケット仕様をフルスペックで解放した **Ryzen Threadripper Pro** だが、当然制約も引き継がれている。  

{{< figure src="/image/2020/07/15/castle_peak-diagram.webp" title="Castle Peak" caption="画像出典: [AMD Ryzen™ Processor Software Optimization – GPUOpen_Let’sBuild2020_AMD Ryzen™ Processor Software Optimization.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_AMD%20Ryzen%E2%84%A2%20Processor%20Software%20Optimization.pdf)" >}}

AMD はサーバ向け **EPYC 7002シリーズ (Rome)**、デスクトップ向け **Ryzen 3000シリーズ (Matisse)** 、ハイエンド向け **Ryzen Threadripper 3000シリーズ (Castle Peak)** において、CPUコアをまとめた *CCD* と I/O機能をまとめた *IOD* を分離するチップレットアーキテクチャを導入したが、  
*CCD* と *IOD* 間の接続帯域は、Read と Write で異なり、Read 2 (32B/C) : Write 1 (16B/C)のバランスとなっている。{{< comple >}} これはダイ間のデータ転送に伴う消費電力を削減するためとされている {{< /comple >}}

これによりメモリ帯域は実質 *CCD* の数に制限される。  
メモリコントローラと DDR4メモリが 16Byte/Cycle で接続されているため、8ch の帯域を満たすのに Read だけなら *4 CCDs* でも十分だが、その場合 Write の帯域は 4ch相当となり、Write 帯域を満たすなら最大搭載可能数である *8 CCDs* の構成である必要がある。  
Read 帯域も *4 CCDs* 未満の場合は満たすことができず、4ch相当の帯域となる。  
*sIOD* は 2つの Data Fabric を持ち、*CCD* 数は対称的となるため、*3 CCDs* の構成を取ることは出来ず、*4 CCDs* 未満のパターンには *2 CCDs* 、*1 CCD* があたる。  

*2 CCDs* 構成である **EPYC 7282 /7272 /7252 /7232P** はその旨が記載されており[^rome-2ccd]、性能は 4ch DDR4-2667MHz に最適化され、搭載DIMM数を増やしても帯域は増えないとある。  
**EPYC 7232P** は L3cache 32MB だが、[ServeTheHome](https://www.servethehome.com)によるレビュー記事内の `lstopo` コマンド実行結果を見ると、L3cache 8MB が 4基存在することがわかる。[^sth-epyc-7232p-topo]  
このことから **EPYC 7232P** は *2 CCDs* 構成だが、*CCX* あたりの L3cache を半分に制限していると考えられる。  

[^sth-epyc-7232p-topo]: [AMD EPYC 7232P In GB NVMe Server Topology | ServeTheHome](https://www.servethehome.com/amd-epyc-7232p-review-hard-to-buy-but-solid-part/amd-epyc-7232p-in-gb-nvme-server-topology/)

[^rome-2ccd]: [2nd Gen AMD EPYC™ 7282 | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7282#product-specs)<br>[2nd Gen AMD EPYC™ 7272 | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7272#product-specs)<br>[2nd Gen AMD EPYC™ 7252 | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7252#product-specs)<br>[2nd Gen AMD EPYC™ 7232P | Server Processor | AMD](https://www.amd.com/en/products/cpu/amd-epyc-7232p#product-specs)

よって、L3cacheサイズから *2 CCDs* 構成と考えられる **Pro 3955WX** と **Pro 3945WX** はRead 帯域においても 4ch 相当であると思われる。  
それでもメモリ容量は 2TBまで対応しているため、Pro としてメモリ性能で劣るばかりではない。  

**Pro 3975WX** も Write 帯域は 4ch相当になるだろうが、チップレットアーキテクチャはダイ分離に伴うメモリレイテンシと帯域不足を補うため、L3cacheを前世代から大幅に増やしている。  
Write 帯域の制限がそのまま性能の限界に繋がることもそうないと思われる。  
また、チップレットアーキテクチャの制限と書いたが、AMD はこれにより、同コア同スレッド数でも *CCD* 数を変えることで特長の異なるモデルを複数出すことに成功している。  
こうした柔軟性はチップレットアーキテクチャの利点と言えるだろう。  
