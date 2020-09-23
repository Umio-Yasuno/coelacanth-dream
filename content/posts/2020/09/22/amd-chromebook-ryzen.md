---
title: "AMD、Chromebook向け \"Zen\" プロセッサを正式発表"
date: 2020-09-22T22:59:22+09:00
draft: false
tags: [ "Chromebook", "Picasso", "Dali" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

AMD は現地時間 2020/09/22 付で、*Zenアーキテクチャ* をベースとする Chromebook向けプロセッサ **Ryzen & Athlon C-Series** を 5モデルを発表した。  
前世代のChromebook向け AMDプロセッサと比較して、Webブラウジングでは 178%、マルチタスク、コンテンツ作成では 212% 高速であるとしている。  
今回発表されたモデルの中では最上位となる **Ryzen 7 3700C** は、Chromebook では最高のグラフィクス性能を持ち、14nm世代の Intel Coreプロセッサ{{< comple >}}**Intel Core i7-8550U (Kaby Lake)** 、**Intel Core i5-10210U (Comet Lake-U)** 、**Intel Core i3-10110U (Comet Lake-U)** {{</comple>}}と比較しても優れたグラフィクス性能を示したという。  

 * [AMD Launches First “Zen”-based Chromebook mobile processors for faster web browsing, improved office productivity and better multitasking :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/969/amd-launches-first-zen-based-chromebook-mobile)

| Model | Cores/Threads | TDP(W) | Boost/Base Freq | GPU Cores(CU) | GPU Clock |
| :-- | :--: | :--: | :--: | :--: | :--: | 
| Ryzen 7 3700C | 4C/8T | 15W | 4.0/2.3 GHz | 10 | 1400 MHz |
| Ryzen 5 3500C | 4C/8T | 15W | 3.7/2.1 GHz | 8 | 1200 MHz |
| Ryzen 3 3250C | 2C/4T | 15W | 3.5/2.6 GHz | 3 | 1200 MHz |
| Athlon Gold 3150C | 2C/4T | 15W | 3.3/2.4 GHz | 3 | 1100 MHz |
| Athlon Silver 3050C | 2C/2T | 15W | 3.2/2.3 GHz | 2 | 1100 MHz |

基本的なスペックは末尾 C を U に置き換えたモデルと一致するが、**Ryzen 5 3500U** **Ryzen 7 3700U** が cTDP 35W までに対応しているのに対し、C-Series では 25W までとなっていたり、  
**Athlon Gold 3150U** では GPUクロックが最大 1000MHz となっているが、**3150C** では 1100MHz と、細かな違いが存在する。  
**3700C, 3500C** が [12nm Picasso APU](/tags/picasso)を、**3250C, 3150C, 3050C** が [14nm Raven2/Dali APU](/tags/dali)をベースとする点は変わらない。  

前世代からはプロセッサの性能だけでなくマルチメディアエンジンも世代が進み、HEVCデコード/エンコード、VP9デコードに対応しているため使い勝手はかなり良くなっているように思う。  

発表が唐突なものに思えるが、Google が日本時間では 2020/10/01 の午前3時に [Launch Night In](https://launchnightin.withgoogle.com/jp/) というイベントを予定しているため、それに合わせたのかもしれない。  
それと、より省電力、低コストな [Pollock APU](/tags/pollock) をベースとし、この前 SKU名が判明した**AMD 3015Ce** が見当たらないが、  
{{< link >}} [Chromebook向け Pollock APU、AMD 3015Ce | Coelacanth's Dream](/posts/2020/09/09/amd-3015ce-chromebook/) {{< /link >}}
同じく [Pollock APU](/tags/pollock) をベースとする **AMD 3015e** も搭載製品の発表と同時に AMD公式サイトにスペックが記載される、という感じであったため、**AMD 3015Ce** も **Ryzen & Athlon C-Series** 搭載製品と一緒にちゃっかり出てくるかもしれない。  

**Ryzen & Athlon C-Series** を搭載するChromebookは Acer、ASUS、HP、Lenovo より 2020Q4 に発売される。  
