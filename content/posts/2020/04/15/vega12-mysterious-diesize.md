---
title: "Vega12 謎?のダイサイズ"
date: 2020-04-15T21:53:34+09:00
draft: false
tags: [ "Vega12", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

<https://bilibili.com>にて[Morisummers](https://space.bilibili.com/1734177)氏が[Vega12](/tags/vega12)の開発用ボードを分析した記事を投稿した。  
[Radeon Pro Vega20拆解 - 哔哩哔哩](https://www.bilibili.com/read/cv5598542)  

ボードに搭載されていたのは *Vega12 XTA* と、開発用でありながら製品のMacBook Pro 2018に搭載された **Radeon Pro Vega 20** と同一のものだった。  
GPU-Zの表示では、PCIe Gen3は8レーン。  
そして、氏の測定によれば、*Vega12* のダイサイズは 252mm<sup>2</sup>。

この 252mm<sup>2</sup>はざっくり言うと *Vega10* の半分近い大きさだ。  
しかし、 *Vega10* と *Vega12* の規模にはそれなりの差がある。  
*Vega10* の総CU数が64基なのに対し、*Vega12* は20基と1/3近く、RBEは *Vega10* が16基(64 ROP)、 *Vega12* は8基(32 ROP)と半分だ。  
直感的には *Vega12* のダイサイズはもう少し小さくてもいいように思える。  

| | Vega10[^4][^3] | Vega12[^2][^3] |
| :--- | :---: | :---: |
| ShaderEngine | 4 | 4 |
| Total CU / SP | 64 / 4096 |  20 / 1280 |
| Total TMU | 128 | 80 |
| Total ROP | 64 | 32 |
| Total L2$ | 4 MB | 2 MB |
| Memory Type<br>/ Bus Width | HBM2<br>/ 2048-bit | HBM2<br>/ 1024-bit |
| GFX Core | GFX9 | GFX9 |
| GPU ID | gfx900 | gfx904 |
| DCE | v12.0 | v12.0 |
| Display<br>Controllers | 6 | 6 |
| UVD | v7.0 | v7.0 |
| VCE | v4.0 | v4.0 |
| CMOS | 14nm | 14nm |
| DieSize | 509.73mm<sup>2</sup> | 252mm<sup>2</sup>

[^2]: [Vega12 - pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L854)
[^3]: [RadeonFeature](https://www.x.org/wiki/RadeonFeature/)
[^4]: [Vega10 - pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L835)

## そこまで謎でもなかったダイサイズ
そこで以前作った *Vega10* のダイアグラムから規模を合わせた *Vega12* のダイアグラムを作成してみた。[^1]  

{{< figure src="/image/2020/04/15/vega12-diagram.png" title="Vega12 推測ダイアグラム" caption="ダイサイズ: 252mm<sup>2</sup>" >}}

[^1]: [Vega10/Vega20のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/03/24/vega10-vega20-dieshot-guess/#vega10)

直感に反して、幅は *Vega10* のほぼ半分とダイサイズに沿った大きさとなった。高さは合わせたため、*Vega10* と同じ。  
ダイショットから推測したダイアグラムを再配置した、というものであるため、当然正確性には欠けることを留意したい。  

*Vega12* が大きくなった理由としては、まず 4-ShaderEngineという構成であることが考えられる。  
言い換えれば、フロントエンド部が占める大きさが *Vega10* と変わらない。  
総CU数が20基と少なめなのに 4-ShaderEngineとしたのは、コンピュートタスクをより並列に実行し、性能効率を高める目的があったのではないかと思う。  

次に、メモリにHBM2を採用していることが考えられる。  
GDDR系のPHY(物理層)であれば、縦横の比率が大きく、*Polaris10* のように ShaderEngine をぐるりと囲む配置を取れるが[^5]、HBM2 PHYはGDDR系よりは縦横の比率が小さい。  
そのため、ダイサイズを喰わないように配置するのが難しいのではないと思う。  

[^5]: [Polaris10/Polaris11のダイ観察 & 推測](/posts/2020/03/30/polaris10-polaris11-dieshot-guess/)

このことから言えるのは、規模から単純にダイサイズを求めるのは難しく、またダイサイズから規模を図るのも難しいということだ。  
