---
title: "AMD Sienna Cichlid は USB-Cコントローラーを内蔵、PD機能もサポート"
date: 2020-09-11T15:31:27+09:00
draft: false
tags: [ "RDNA_2", "Sienna_Cichlid", "Navi21" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

Linux Kernel (amd-gfx) に投稿されたパッチから、*Sienna Cichlid* が [Navi10](/tags/navi10) 同様にUSB-Cコントローラーを内蔵、USB PD(Power Delivery) 機能をサポートすることが分かった。  
{{< link >}} [[PATCH] drm/amdgpu: Include sienna_cichlid in USBC PD FW support.](https://lists.freedesktop.org/archives/amd-gfx/2020-September/053671.html) {{< /link >}}

パッチの内容としては、USB-C PD ファームウェアを読み込む前の判定文に *Sienna Cichlid* を追加したのみの簡潔なものであり、USB-C部に関しては *Navi10* と変わらないと考えられる。  
データ転送、DisplayPort Alternate Mode をサポートする点も同じだろう。  


*Sienna Cichlid* ベースの製品でどれだけ USB-Cコネクタを実装したカードが登場するかは、*Navi10* では **Radeon Pro W5700** だけであることと、  
映像出力と、デバイスコントローラー等とのUSB接続を 1本のUSB-Cケーブルで実現するVR HMD向けの規格 VirtualLink が実質終了してしまったことから[^virtuallink]、コンシューマ向けの製品で実装するとは考えにくく、やはりプロ向けの一部製品のみとなるのではないかと思う。  
USB-C は多機能であるが、その分検証にコストと時間が掛かり、それは製品の価格に反映される。  
表記上の TDP が増える点も受けが悪い。注釈や記述方法でいくらでも回避できそうだが、**Radeon Pro W5700** はニュースリリースでの脚注に USB を含まなければ 190W相当と書いてあっただけで[^w5700-announce]、製品ページではそれも無く TDP: 205W と記載されている。[^w5700-product]  
いっそのこと USB も含めての TDP とし、USB で増えた分をブースト機能で活かすということも考えられるが、USB を使えばブースト機能の効果が低下するとも解釈できるためマーケティング的にも難しいのかもしれない。  

[^virtuallink]: [ASCII.jp：GeForce RTX 3090で夢の8Kゲーミングは実現するのか？HDMI 2.1とDLSSの役目を解説 (4/4)](https://ascii.jp/elem/000/004/026/4026224/4/)
[^w5700-announce]: [AMD Launches World’s First 7nm Professional PC Workstation Graphics Card for 3D Designers, Architects and Engineers | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-launches-worlds-first-7nm-professional-pc-workstation)
[^w5700-product]: [Radeon™ Pro W5700 | Workstation Graphics Card | AMD](https://www.amd.com/en/products/professional-graphics/radeon-pro-w5700#product-specs)

そういえば SoC としての側面が強い [APU Renoir](/tags/renoir) はディスプレイ出力可能な USB-Cコネクタをネイティブで 2つサポートしていたが[^rn-usbc]、*Navi10 /Sienna Cichlid* は 1つのみなのだろうか？

[^rn-usbc]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/dcn21/dcn21_resource.c?h=amd-staging-drm-next&id=b4eeba998577ee6c656e81eb762fba5c78efa5cc#n804>

{{< ref >}}

 * [【やじうまPC Watch】Radeon Pro W5700のUSB機能はGPUが提供、データ転送も可能 - PC Watch](https://pc.watch.impress.co.jp/docs/news/yajiuma/1220150.html)
 * [ASCII.jp：GeForce RTX 3090で夢の8Kゲーミングは実現するのか？HDMI 2.1とDLSSの役目を解説 (4/4)](https://ascii.jp/elem/000/004/026/4026224/4/)

{{< /ref >}}
