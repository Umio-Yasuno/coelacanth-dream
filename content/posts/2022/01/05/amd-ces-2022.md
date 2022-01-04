---
title: "AMD CES 2022 個人的まとめ"
date: 2022-01-05T00:45:05+09:00
draft: false
tags: [ "Yellow_Carp", "Beige_Goby", "Barcelo" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
noindex: true
# summary: ""
---

CES 2022 に併せたライブストリーミングにて AMD より新たな CPU、GPU、APU が発表された。  
発表内容すべてをまとめる気力はないため、一部を個人的な記録としてまとめる。内容すべてを知りたいときは各種ニュースサイトか公式発表を確認してほしい。以前にも書いたが、このサイトはあくまでもオタクの日記。  

 * {{< youtube title="AMD 2022 Product Premiere" id="_jX-hKvUQDU" >}}
 * [AMD Unveils New Power-Efficient, High-Performance Mobile Graphics for Premium and Thin-and-Light Laptops, and New Desktop Graphics Cards :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1038/amd-unveils-new-power-efficient-high-performance-mobile)
 * [AMD Unveils New Ryzen Mobile Processors Uniting “Zen 3+” core with AMD RDNA 2 Graphics in Powerhouse Design :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1039/amdunveils-new-ryzen-mobile-processors-uniting-zen)
 * [AMD Presents Latest High-Performance Computing Technologies in 2022 Product Premiere Livestream :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/1040/amd-presents-latest-high-performance-computing-technologies)

{{< pindex >}}
 * [Ryzen 6000U/H/HX (Yellow Carp [Rembrandt])](#yc)
 * [Radeon RX 6500 XT / RX 6400 (Beige Goby [Navi24])](#bg)
{{< /pindex >}}

## Ryzen 6000U/H/HX (Yellow Carp [Rembrandt]) {#yc}

6nmプロセスで製造され、*Zen 3+* と *RDNA 2* を組み合わせた APU をベースとする **Ryzen 6000U/H/HX シリーズ** が正式発表された。  
*Zen 3+* では電力管理機能が強化され、クロックの切り替えレイテンシ削減と新たな Deep Sleepステートにより、性能と電力効率が一緒に強化されているとする。  
GPU部は *RDNA 2 アーキテクチャ* となっただけでなく、L2キャッシュと RB (RenderBackend) の規模は倍に、CU数は *Renoir/Lucienne/Cezanne (Green Sardine) /Barcelo* の 8基から 12基へと 1.5倍になった。[^rdna_2]  
補足すると、RB (Render Backend) あたり 8 ROP相当という意味での RB+ は APU だと *Stoney Ridge* や *Raven (Zen, Vega, 14nm)* の世代から実装、有効化されていた。  
そのため Pixel Rate は RB数同様のスケールとなっている。*RDNA 2 アーキテクチャ* の採用と併せて 4倍ということにはならない。  
{{< link >}} [一部の AMD GPU で実装、有効化されている RB+ とは何か | Coelacanth's Dream](/posts/2020/11/10/what-is-rbplus/) {{< /link >}}

単純な数値の比較ではあるが、Shader Processor と Pixel Rate において、*Tiger Lake-U* や *Alder Lake-P/M* に搭載されている Intel Gen12 GT2 は最大で 768 SP、24 ROP相当であり、それら規模では今まで AMD APU より上だった。(実性能は動作クロックやドライバー、メモリ性能等が関わってくる)  
*Yellow Carp* ではそれを追い抜き返した形となる。  

| iGPU | Renoir /Lucienne<br>/Cezanne (Green Sardine) /Barcelo | Yellow Carp<br>(Rembrandt) | Gen12 GT2<br>(Tiger Lake /Alder Lake-P/M) |
| :-- | :--: | :--: | :--: |
| | 8 CU | 6 WGP<br>(12 CU) | 96 EU |
| Shader Processor | 512 SP | 768 SP | 768 SP |
| Pixel Rate | 16 | 32 | 24 |

ダイサイズはプロセッサ仕様によると 210mm2 としており、*Cezanne* の約 180mm2 より大きくなっている。  
CPU部は *Zen 3* と電力管理機能を除いてほとんど同じとされているため、PCIe Gen4, USB4 に対応した I/O部、*RDNA 2 アーキテクチャ* の採用と同時に規模が大きくなった GPU部の影響と思われる。  

[^rdna_2]: {{< youtube id="_jX-hKvUQDU" start="683" >}}

個人的なトピックだった、*Yellow Carp (Rembrandt)* のハードウェア AV1デコードの有無については、びみょうにはっきりしないところがある。  
まず、半年近く前 (2021/07/14) に投稿されたパッチで、*Yellow Carp* がHWデコードをサポートするフォーマットの中に MPEG2, MPEG4, VC1, AV1 は無かった。  
現時点で該当部分に変更はされていない。  
{{< link >}} [他 RDNA 2 GPU とは対応コーディックが異なる Yellow Carp APU と Beige Goby GPU | Coelacanth's Dream](/posts/2021/07/14/yc-bg-vcn/#yc) {{< /link >}}
だが発表会中では、**Ryzen 6000 シリーズ** は AV1デコードに対応するとしていた。[^yc-av1]  
パッチで追加されたコード部は DRM API 経由で `amdgpu_asic_query_video_codecs` を実行したときに対応するフォーマットの情報を返すのに使われているが、User Mode Driver である Mesa3D ではその中で対応サイズの情報だけを使っていた。  

少しだけ話がずれてしまうが、AMD のプロセッサ仕様ページが最近アップデートし、Intel ark に近い形式となった。コードネームやアーキテクチャ、ダイサイズ等の情報が追加され、同時に見やすくなったと感じる。  
**Ryzen 6000 シリーズ** のページも既に公開されており、*Rembrandt* の名が確認できる。  
対応ビデオフォーマットの情報もあるのだが、**Ryzen 5 6600U** と **Ryzen 7 6800U** とでズレがあり、参考にしにくい。  

 * [AMD Ryzen™ 5 6600U | AMD](https://www.amd.com/en/product/11596)
 * [AMD Ryzen™ 7 6800U | AMD](https://www.amd.com/en/product/11591)

どちらにも AV1デコードの表記が無いのだが、何故か対応サイズが 1080p までだったり、**Ryzen 5 6600U** には MPEG2, VC1 があったりと、公開直後である現時点ではやはり参考にできない。  
一応、Mesa3D でも *Beige Goby (Navi24)* と *Yellow Carp (Rembrandt)* は MPEG2, MPEG4, VC1 に対応しないとしている。`CHIP_BEIGE_GOBY` と `PIPE_VIDEO_FORMAT_MPEG4_AVC` はそれぞれ `enum` で管理されている。[^enum-amd-family] [^enum-format]  

 > 		   switch (param) {
 > 		   case PIPE_VIDEO_CAP_SUPPORTED:
 > 		      if (codec < PIPE_VIDEO_FORMAT_MPEG4_AVC &&
 > 		          sscreen->info.family >= CHIP_BEIGE_GOBY)
 > 		         return false;
 >
 > {{< quote >}} [src/gallium/drivers/radeonsi/si_get.c · c9a47c85da9538ae82445f9754192d84afc7006d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/c9a47c85da9538ae82445f9754192d84afc7006d/src/gallium/drivers/radeonsi/si_get.c#L583-587) {{< /quote >}}

そういう訳で *Yellow Carp* の AV1デコードについてはまだ少し判然としないところがある。  

[^enum-amd-family]: [src/amd/common/amd_family.h · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/amd/common/amd_family.h#L115-117)
[^enum-format]: [src/gallium/include/pipe/p_video_enums.h · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/gallium/include/pipe/p_video_enums.h#L35-46)
[^yc-av1]: {{< youtube id="_jX-hKvUQDU" start="986" >}}

## Radeon RX 6500 XT / RX 6400 (Beige Goby [Navi24]) {#bg}

新たな *RDNA 2* dGPU **Radeon RX 6500 XT / RX 6400** が発表された。  
メモリバス幅、Infinity Cache の規模から *Beige Goby (Navi24)* をベースにしていると思われる。  
*RDNA 2* dGPU では最も小規模で、**Ryzen 6000 シリーズ** と同じく 6nmプロセスで製造されることが特徴に挙げられる。  
**RX 6400** は 1スロット、TBP (Total Board Power) 53W、WGP 6基 (CU 12基)、GDDR6 16Gbps 4GB、  
**RX 6500** は 2スロット、TBP 107W、WGP 8基 (CU 16基)、GDDR6 18Gbps 4GB となっている。  
2つで TBP の差が大きいが、有効 WGP数以上に、Game Clock では 571 MHz、Boost Clock では 494 MHz の差が要因になっていると思われる。  

 * [AMD Radeon™ RX 6400 | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6400#product-specs) 
 * [AMD Radeon RX 6500 XT Graphics Card | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6500-xt#product-specs)

上 2つのページでは、動画エンコードと AV1デコードには非対応とされており、これはパッチでの情報と一致する。  
だが現時点で **RX 6300M/6500M** は AV1デコードに対応と表記されており、信頼しきれない部分がある。  

 * [AMD Radeon™ RX 6300M | AMD](https://www.amd.com/en/product/11501)
 * [AMD Radeon™ RX 6500M | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-6500m#product-specs)

1080p でのゲーム性能において、**RX 570 (Polaris19, 14nm)** より高速であることがアピールされている。  
トランジスタ数が、*Polaris10* は 5.7 B、*Beige Goby (Navi24)* は 5.4 B とされており、微細化によるトランジスタ密度向上、ユニット数の増加ではなく、アーキテクチャの進化と倍以上にもなる動作クロックの向上で **RX 6500 XT** は **RX 570** 以上の性能を達成している。  
**RX 6500 XT** は GDDR6 18Gbps を採用しているが、メモリバス幅は 64-bit であり、**RX 570** と比較してメモリ帯域は 64% 程度 (144 GB/s vs 224 GB/s) である。この点、Infinity Cache 16 MiB がかなり活きているように思う。  
しかし、1080p より上の解像度や GPGPU 等においては 16 MiB ではカバーしきれず、最大 4GB というメモリサイズもあって、性能は低下すると考えられる。  
*RDNA 2 アーキテクチャ* の中でも、ゲームに特化しているという特徴が濃く出ているようにも感じる。  

| RDNA 2 | Sienna Cichlid<br>(Navi21) | Navy Flounder<br>(Navi22) | Dimgrey Cavefish<br>(Navi23) | Beige Goby<br>(Navi24) |
| :-- | :--: | :--: | :--: | :--: |
| Shader Engine | 4 | 2 | 2 | 1 ? |
| WGP (CU) | 40 (80) | 20 (40) | 16 (32) | 8 (16) ? |
| RB+ | 16 | 8 | 8 | 4 |
| Memory Bus | 256-bit | 192-bit | 128-bit | 64-bit |
| Infinity Cache | 128 MiB | 96 MiB | 32 MiB | 16 MiB |
| Transistor | 26.8 B | 17.2 B | 11.1 B | 5.4 B |


{{< ref >}}
 * [src/gallium/drivers/radeonsi/si_get.c · c9a47c85da9538ae82445f9754192d84afc7006d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/c9a47c85da9538ae82445f9754192d84afc7006d/src/gallium/drivers/radeonsi/si_get.c)
 * [AMD@7nm@Zen3@Cezanne@Ryzen_5_5600G@100-000000252_EF_2127SU… | Flickr](https://www.flickr.com/photos/130561288@N04/51374148416/in/photostream/)
{{< /ref >}}
