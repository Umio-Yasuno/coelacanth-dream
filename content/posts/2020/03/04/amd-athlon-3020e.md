---
title: "Lenovo製品の仕様より A4-3020E が確認される"
date: 2020-03-04T23:10:13+09:00
draft: false
tags: [ "Raven2", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU"]
noindex: false
---

発端は[188号@momomo_us](https://twitter.com/momomo_us)氏のツイート。  
<https://twitter.com/momomo_us/status/1235196239752220672>  

この情報を基に探した所、以下3つの製品仕様を記したPDFファイルが見つかった。  

 * <http://psref.lenovo.com/syspool/Sys/PDF/IdeaPad/IdeaPad_3_14ADA05/IdeaPad_3_14ADA05_Spec.PDF>  
 * <https://psref.lenovo.com/syspool/Sys/PDF/Lenovo/Lenovo_V14_ADA/Lenovo_V14_ADA_Spec.PDF>  
 * <http://psref.lenovo.com/syspool/Sys/PDF/Lenovo/Lenovo_V15_ADA/Lenovo_V15_ADA_Spec.PDF>  

## A4-3020E 仕様

A4-3020Eの基本的な仕様は、  
CPU: 2-Core/2-Thread、Base Freq 1.2GHz、Max Turbo Freq 2.6GHz、L2キャッシュ 1MB、L3キャッシュ 4MB。  
CPUの仕様だけを見れば、先日公式に発表されたR1102Gとまんま同じだ。  
A4というセグメントが付けられてはいるが、ASICもR1102Gと同じRaven2ベース (Zen APU)のSKUだと思われる。  
少なくともキャッシュの仕様から、Zenアーキテクチャであることは確かだ。  

ただ、GPUの仕様は不明なため触れないでおくが、メモリとPCIeレーンの仕様はR1102Gとは違っている。  
R1102Gではメモリはシングルチャネル、外部へのPCIe Gen3は4-Laneまでに抑えられていた。  
対してA4-3020Eは、容量こそ最大8GBとなっているが、オンボード実装されたDDR4メモリ 4GBとは別にSO-DIMMソケットを利用できることから、デュアルチャネルだと考えられる。  
PCIeレーンは、Storageの項目にPCIe 3.0 x4接続のM.2 SSDがあり、また特に注釈や制限されていることを伝える旨がファイル中に無いため、PCIe Gen3 8-Laneまで利用可能だと思われる。  

## おまけ 〜DaliかPollockか〜
A4-3020EがRaven2ベースだとして、[Raven2](/tags/raven2) /[Dali](/tags/dali) /[Pollock](/tags/pollock)のどれに属するか。  
自分としては *Dali* の気がする。  
根拠としては、コンシューマー向けに用いられることが多い RevisionID: CX の予約が *Dali* に1つあり、[^1]  
そしてTDP 6WということがLinux Kernelへのパッチで判明している。[^2]  
Eシリーズは過去の例から通常よりも省電力、低TDPであり、TDP 6Wというのは法則に沿っている。  

まあ相変わらず *Dali* と *Pollock* の違いがどれだけの意味を含んでいるかは不明。  

[^1]: [AMDGPU Database: Device ID, Revision ID, Product Name | Coelacanth's Dream](/posts/2019/12/30/did-rid-product-matome-p2/#dali-gfx909)
[^2]: [[Dali] Raven 2 detection Patch ](https://lists.freedesktop.org/archives/amd-gfx/2020-February/045579.html) <br> <https://lists.freedesktop.org/archives/amd-gfx/attachments/20200205/f22ebc46/attachment.obj>

