---
title: "AMDの新しい小型PC向けチップセット「PRO 500」"
date: 2020-01-28T17:33:44+09:00
draft: false
tags: [ "PCH",  ]
keywords: [ "", ]
categories: [ "Hardware", "Chipset" ]
noindex: false
---

チップセットというよりは1つのスペック規格と言った方がいいかもしれない。  

<br>
AMDの公式サイトにひっそりと"PRO 500"チップセットの名が追加されていた。  
[Socket AM4 Platform | AMD](https://www.amd.com/en/products/chipsets-am4)

 > To satisfy customers who value the smallest form factors, AMD’s X300, A300, and PRO 500 chipsets provide processor-direct access for excellent performance.

X300やA300のように、外部チップセットを持たず、RyzenをよりSoCとして機能させるチップセット――規格――プラットフォーム――と思われる。  

それとB300も一応あるはずだが、何故かハブられている。  
[AMD X300 Motherboards Galore - photos ASRock MSI BioStar and Gigabyte ](https://www.guru3d.com/news-story/amd-x300-motherboards-galore-photos-asrock-msi-biostar-and-gigabyte.html)  
[IdeaCentre A540 (24, AMD) | 24” All-in-one PC | Lenovo US](https://www.lenovo.com/us/en/desktops-and-all-in-ones/ideacentre/aio-500-series/IdeaCentre-A540-24API/p/FFICF500327)  
[ideacentre_A540_24API_Spec.pdf](https://psref.lenovo.com/syspool/Sys/PDF/IdeaCentre/ideacentre_A540_24API/ideacentre_A540_24API_Spec.pdf)  

"PRO 500"という名前と、第三世代RyzenをサポートしていることからPCIeGen4がありそうだが{{< comple >}}"PRO 560"と微妙に違いはするが{{< /comple >}}採用している製品のデータシートを見ると、NVMeのバスも拡張スロットもPCIeGen3となっている。  
[ideacentre_T540_15AMA_G_Spec.PDF](https://psref.lenovo.com/syspool/Sys/PDF/IdeaCentre/ideacentre_T540_15AMA_G/ideacentre_T540_15AMA_G_Spec.PDF)  

フロントへの2x USB 3.1 Gen2は、Ryzenから引き出している可能性があるが、別のPRO 560を採用し、APU（中身としては第二世代Ryzen）も搭載可能な製品でもUSB 3.1 Gen2が記載されているため、別途コントローラーを用意し、そこから出している可能性のが高い。  
[ThinkCentre M75s | Compact, High-Performance PC with AMD Ryzen™ PRO Processing | Lenovo NZ](https://www.lenovo.com/nz/en/desktops-and-all-in-ones/thinkcentre/m-series-sff/ThinkCentre-M75s-1/p/11TC1MD735S)  
ただそれも、第三世代Ryzenを搭載する場合はRadeon 520から映像が出力されるため、同じボードを使っているとは限らず、PRO 560の仕様がうまく掴めない。  

それに拡張スロットが3本しっかりあったりと、外部チップセットのない"PRO 560"を使う意味があまりないように思える。  
その拡張スロット3本も許容した規格が"PRO 500/560"であり、特徴ということなのだろうか？  

OEM向けの規格、チップセットはその通り自作PC市場に出回ることが少なく、資料もまた少ないため{{< comple >}}X/B/A300は第一世代Ryzenの際に発表されたきりで、以降AMD公式から言及することはなかった{{< /comple >}}わからないことが多い……  

<br>
RyzenはI/O周りの機能が豊富で、外部チップセットレスのマザーボードを望む声は少なくない。実際、ASRock DeskMini A300は外部GPUなしで小型に組みたいユーザーから支持されている。  
GPU1つといくつかのM.2スロットだけで十分なユーザーもそれなりにいるはずだが、メーカーからすればそういったマザーボード単体で売るのはあまりおいしくないため、難しいのかも知れない。  
