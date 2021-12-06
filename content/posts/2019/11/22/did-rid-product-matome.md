---
title: "AMDGPUのDID、RID、Productをびみょうにまとめた"
date: 2019-11-22T02:22:24+09:00
draft: false
tags: [ "Radeon" ]
keywords: [ "Radeon", "AMDGPU" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

[amdgpu.ids - drm](https://gitlab.freedesktop.org/mesa/drm/blob/master/data/amdgpu.ids)がRadeon VII以降更新されないため、個人的なまとめとして出来る限り補完した。  

モバイル向けのリビジョンは、主に[notebookcheck.net](https://www.notebookcheck.net)のレビュー記事内のGPU-Zスクリーンショットを元にしている。  
他も概ねそのようにして集めた。  

| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15DD | | | **Raven** |
| | C3 | 2700U | Mobile |
| | C4 | 2500U | |
| | C5 | 2200U | |
| | C6 | 2400G | Desktop |
| | C8 | 2200G | |
| | CB | 200/220/240GE | |
| | CC | 2300U | Mobile |
| | D1 | PRO 2500U | |
| | | 2600H | |
| | | 2800H | |
| 15D8 | | | **Picasso** |
| | C1 | 3750H | Mobile |
| | C2 | 3500U (3550H) | |
| | C8 | 3400G | Desktop |
| | C9 | 3200G | |
| | D1 | (PRO) 3700U | Mobile |
| | D2 | PRO 3500U | |
| | D3 | (PRO) 3300U | |
| | E1/E2 | 3580/3780U | Surface Edition |
| | E3/E4 | | **Dali** |
| 15DD/15D8 | | | **Raven2** |
| 15DD | E2 | | |
| | | 300U | Mobile |
| | CC | 3000G | |
| | | 3200U | |
| | | R1505G | Embedded |
| | | R1606G | |
| 1636 | | | **Renoir** |
||
| 7310 | | | **Navi10** |
| 7312* | | Radeon Pro W5700 | |
| 7318 | | | |
| 7319 | | | |
| 731A | | | |
| 731B | | | |
| 731F | C0 | RX 5700XT 50th | (XTX) |
| | C1 | RX 5700XT | (XT) |
| | C4 | RX 5700 | (XL) |
| 7340 | | | **Navi14** |
| | C1 | RX 5500M | (GamingXTM) |
| | C3 | RX 5300M | (XLM) |
| | C5 | RX 5500XT | (GamingXTX) |
| | C7 | RX 5500 | (GamingXT) |
| 7341 | 00 | | (WKS Pro-XL) |
| 7347 | 00 | | (WKS Pro-XTM) |
| 734F | 00 | | (WKS Pro-XLM) |
||
| 7360 | | | **Navi12** |
| 7362 | | | (Navi12 VF) |
||
| 738C | | | **Arcturus** |
| 7388 | | | |
| 738E | | | |
| 7390 | | | (Arcturus VF) |

DeviceIDとChipNameだけがわかって、RevisionID、ProductNameがさっぱりな項目が多い。  
そこはこれから地道に埋めていきたい。  


このブログは、私がスマホから簡単に確認できるデータベースとしても使っているが、  
ここまで表がでかいとカクつきそうだ。  
いや確実にカクつく。  
