---
title: "RX 6500 XT の実効メモリ帯域"
date: 2022-01-13T16:27:03+09:00
draft: false
tags: [ "Beige_Goby", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU", "Note" ]
noindex: false
# summary: ""
---

*Beige Goby/Navi24* ベースの SKU に関するメモ書き。  

## RX 6500 XT の実効メモリ帯域 {#eff-bw}
**RX 6500 XT** のスペックに、ピークメモリ帯域に加えて、Infinity Cache 16MB も計算に入れた実効メモリ帯域 (Effective Memory Bandwidth) が記載されている。  
*現時点* で **RX 6500 XT** の実効メモリ帯域は 232 GB/s とされている。  
他の Infinity Cache を搭載する *RDNA 2* GPU に実効メモリ帯域は記載されていないため、SKU間で比較はできない。  

 * [AMD Radeon RX 6500 XT Graphics Card | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6500-xt#product-specs)

実効メモリ帯域 (Effective Memory Bandwidth) という言葉は、*RDNA 2 アーキテクチャ* 正式発表時に Infinity Cache の効果を表すのに使われていた。  
そこでは *Sienna Cichlid/Navi21* を元に実効メモリ帯域を計算しており、GDDR6 16Gbps 256-bit ではピークメモリ帯域が 512 GB/s となるのに対し、同じメモリ構成で Infinity Cache 128MB を搭載すると、1664 GB/s の実効メモリ帯域が得られ、実に 3.25倍の帯域を実現できる、という主張がされていた。[^rdna_2]  
1664 GB/s という数値の根拠はフットノートに記載されている。まず Infinity Cache はメモリチャネルと同数のブロックがあり、キャッシュブロックには 64 Byte/Cycle の帯域でアクセスできる。  
測定時の Boost Clock は 1.94 GHz に設定される。  
そして 4K解像度のゲーム実行時の Infinity Cache の平均ヒットレートを 58% としている。  
以上から、雑ではあるが実効メモリの計算式は以下のようになる。  

([number of memory channel] x [cache bus width] x [hit rate] x [clock]) + ([memory bw])

[^rdna_2]: [RDNA 2 Architecture | One Gaming DNA | AMD](https://www.amd.com/en/technologies/rdna-2)

スペックには記載されていないが、*Navy Flounder/Navi22* ベースの **RX 6700 XT** にも、AMD Partner Hub では実効メモリ帯域を出している。Infinity Cache 96MB (12ch, 1.94GHz)、1440p解像度のゲームで平均ヒットレートは 60% としている。[^rx-6700-xt]  
*Navy Flounder/Navi22* の実効メモリ帯域は 1278 GB/s となり、同メモリ構成に対しては約 3.32倍、GDDR6 16Gbps 256-bit に対しては約 2.5倍の帯域を得られる。  
*Sienna Cichlid/Navi21*、*Navy Flounder/Navi22* と比べて、*Beige Goby/Navi24* は実効メモリ帯域の効果が 1.6倍と控えめ。  
このあたりがメモに残して整理する切っ掛けになった。  

[^rx-6700-xt]: [AMD Radeon™ RX 6700 XT Graphics Cards | AMD Partner Hub](https://www.amd.com/en/partner/amd-radeon-rx-6700-xt)

**RX 6500 XT** は GDDR6 64-bit (4ch) だが、他 *RDNA 2* GPU より動作クロックの高い 18Gbps品を採用しており、ピークメモリ帯域は 144 GB/s となる。GPU の Boost Clock は 2.815 GHz。  
だが実効メモリ帯域 232 GB/s の根拠がスペックページに記載されていないため、上記 *Sienna Cichlid/Navi21*、*Navy Flounder/Navi22* の測定環境に合わせ、GDDR6 16Gbps 64-bit (128 GB/s)、Boost Clock 1.94 GHz と仮定して計算する。  
解像度は **RX 6500 XT** のターゲット帯である 1080p と考える。  
以上を前提に *Baige Goby/Navi24* の平均ヒットレートを求めると、約 20.9% と出た。  
他 *RDNA 2* GPU より平均ヒットレートが低く、やはり Infinity Cache の規模、効果は控えめなものに感じられる。  


そして話が曖昧なものとなってしまうが、少し前まで **RX 6500 XT** の実効メモリ帯域は異なる数値が記載されていた。Wayback Machine で 2022/01/04 のスナップショットを確認すると、その時は 305 GB/s とされている。[^archive]  
実効メモリ帯域 305 GB/s の場合、平均ヒットレートは約 35.6% となる。  

AMD は HotChips33 での発表 **AMD RDNA(TM) 2 Graphics Architecture** で、Infinity Cache のサイズと平均ヒットレートの関係を各解像度 (1080p, 1440p, 4K) ごとに出している。[^hc33-rdna_2]  
位置関係から、Infinity Cache 16MB、1080p では、平均ヒットレートは 20.9% より 35.6% の方が近く見える。  

[^archive]: <https://web.archive.org/web/20220104152019/https://www.amd.com/en/products/graphics/amd-radeon-rx-6500-xt>
[^hc33-rdna_2]: [PowerPoint Presentation - 2021 Hot Chips RDNA2 Pomianowski Final 20210820.pdf](https://hc33.hotchips.org/assets/program/conference/day2/2021%20Hot%20Chips%20RDNA2%20Pomianowski%20Final%2020210820.pdf#page=7)

しかし結局は根拠が出されぬままに変更されたスペックであり、何となくマーケティングの都合をいやに感じてしまう。  

