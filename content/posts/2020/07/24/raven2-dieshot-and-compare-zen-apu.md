---
title: "AMD Raven2 のダイ観察 & Zen系 APU を比較"
date: 2020-07-24T18:49:07+09:00
draft: false
tags: [ "Raven2", "DieShot", "Raven", "Renoir" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

これまでにもCPUやGPUといったチップの赤外線写真、ダイショットを撮影、パブリック・ドメインで公開してきた [Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/) 氏が、  
14nmプロセスで製造される AMD の低コスト省電力な APU、[Raven2](/tags/raven2) のダイショットを公開した。  
{{< link >}} [AMD@14nm@Zen@Dali@Athlon_3000G@YD3000C6M20FH_A0_2002PGT___… | Flickr](https://www.flickr.com/photos/130561288@N04/50145360897/) {{< /link >}}

氏の測定では、*Raven2* のダイサイズは 148.55mm<sup>2</sup>。  
以前、*FP5 パッケージ* から大体の大きさを割り出したことがあったが、それよりももう少し小さかったようだ。  

{{< pindex >}}

 * [Raven2 推測](#rv2-guess)
 * [Zen系 APU 比較](#compare-rv-rv2-rn)

{{< /pindex >}}

## Raven2 推測 {#rv2-guess}

まず、AMD の資料[^amd-prr-fam17h-mod20h] やオープンソースコード[^pal-raven2] から判明している *Raven2* の規模は以下のようになっている。  

 * CPU: 2-Core/4-Thread, L3cache 4MB
 * GPU: 1-ShaderEngine, 3CU, 1RB(Render Backend), L2cache 2基(512KB)
   * Display Controller 3基
 * USB-C 10Gbps 2ポート(DisplayPort-Altmode は 1ポートのみ), USB2 6ポート
 * SATA 最大 2レーン

[^amd-prr-fam17h-mod20h]: [Processor Programming Reference (PPR) for AMD Family 17h Model 20h, Revision A1 Processors (PUB)](https://www.amd.com/system/files/TechDocs/55772-A1-PUB.zip)
[^pal-raven2]: [pal/ndDevice.cpp at ea5db60841dab7d067f5010f28a980ef222bdf81 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/ea5db60841dab7d067f5010f28a980ef222bdf81/src/core/os/nullDevice/ndDevice.cpp#L912)

わかりやすいものだけ区分け、色付けをしてある。  

{{< figure src="/image/2020/07/24/raven2-dieshot.webp" title="Raven2" caption="Die Size: 148.55mm<sup>2</sup><br>画像元: <cite>[AMD@14nm@Zen@Dali@Athlon_3000G@YD3000C6M20FH_A0_2002PGT___… | Flickr](https://www.flickr.com/photos/130561288@N04/50145360897/)</cite>" >}}

CPU部は特にわかりやすく、2-Core に対し、*RavenRidge* や *Renoir* と同容量である L3cache 4MB が付いている。  
ダイエリアとしては、Zen 2-Core と L3cache 4MB がほぼ同じ面積であり、こうして見ると *Zen アーキテクチャ* が小規模であるように見えてしまう。  
GPU部は、CU は 3基、L1K/L1I Cache は 1基で、他の [GFX9](/tags/gfx9) 系同様に CU 3基で共有する構成を取っている。その点で言えば、*Raven2* の GPU部は [GFX9](/tags/gfx9) 系で可能な最小規模であるかもしれない。  
*ACE /Rasterizer /Geometory /RenderBackend* 部は正直適当。RenderBackend も 1基のみであるため、自分には判別が付かない。  
次にわかりやすいのは *Display PHY(物理層)* だろうか。他の AMDGPU と似た形をしている。左端下に 2基が固まっており、中央下の右寄りに *PCIe PHY* 等と組み合わされた *Display PHY* が 1基ある。恐らくそれが DisplayPort-Altmode に対応した部分なのだろう。1ポートのみというのにも一致する。  
さらにその右にある 6基の PHY が *USB PHY* であり、USB2 6ポートという話と一致する。  

それっぽいけどイマイチ確信が持てないのが中央付近の *Display Controller* 3基であり、*Display PHY* の個数に対応している。*RavenRidge* も同じような位置に 4基存在し、そちらも *Display PHY* に対応している。  

## Zen系 APU 比較 {#compare-rv-rv2-rn}

{{< figure src="/image/2020/07/24/rv-rv2-rn.webp" title="L: RavenRidge / C: Raven2 / R: Renoir" caption="画像元: [AMD@14nm@Zen(Zeppelin)@Raven_Ridge@Ryzen_3_2200G@YD2200C5M… | Flickr](https://www.flickr.com/photos/130561288@N04/39716562275/in/photolist-26MvFxA-24LC1aX-23vBLg6-24NZors-24LC1Ux-GJVDw3-GJVGyq-24LBYDv-23MTPRU-23vBQnT-23vCQsF-GJVJs5-24NZ8Yu-23puXQP-24kELtH-G5R3xb-GaWsPs-2aPxAsP-YpUML3-YtN4Pd-Zk7AUm-23cM7Lo-ZAEavU-CpZfG3-236mrbq-26X4AhT-22Qiw2V) <br> &emsp;[AMD@14nm@Zen@Dali@Athlon_3000G@YD3000C6M20FH_A0_2002PGT___… | Flickr](https://www.flickr.com/photos/130561288@N04/50145360897/) <br> &emsp;[AMD@7nm@Zen2@Renoir@Ryzen_3_4300U@100-000000085_9JB4977P00… | Flickr](https://www.flickr.com/photos/130561288@N04/50016639913/)" >}}

CPU Core、CPU L3$ (4MB)、GPU CU は自分が画像から割り出したものであり、誤差が含まれることを留意していただきたい。  

| | RavenRidge | Raven2 | Renoir |
| :-- | :--: | :--: | :--: |
| Process | 14nm | 14nm | 7nm |
| Die Size | 208.84mm<sup>2</sup> | 148.55mm<sup>2</sup> | 155.01.mm<sup>2</sup> |
||
| CPU Arch | Zen | Zen | Zen 2 |
| CPU Core Area<br>(non include L2$) | 5.18mm<sup>2</sup> | 5.18mm<sup>2</sup> | 2.37mm<sup>2</sup>  |
| CPU Core | 4-Core | 2-Core | 8-Core |
| Total CPU L3$ | 4MB | 4MB | 8MB |
| L3$ (4MB) Area | 10.91mm<sup>2</sup> | 10.91mm<sup>2</sup> | 5.39mm<sup>2</sup> |
||
| GPU Arch | gfx902 | gfx909 | gfx909 |
| GPU CU | 11CU | 3CU | 8CU |
| GPU CU Area | 3.79mm<sup>2</sup> | 3.96mm<sup>2</sup> | 1.68mm<sup>2</sup> |
| GPU RB | 2RB | 1RB | 2RB |
| GPU L2$ | 1024KB | 512KB | 1024KB |
||
| DDR4 128-bit PHY Area | 16.50mm<sup>2</sup> | 16.50mm<sup>2</sup> | 15.74mm<sup>2</sup>  |

GPUアーキテクチャとしては *gfx902* と *gfx909* の間に違いはあまり存在しないが、*Raven2* GPU CU のダイ面積が同じプロセスの *RavenRidge* から微増している。  
理由としては 3CU で共有する L1K/L1I Cache 部が考えられ、何でか少し縦に伸びている。  
ただ、測定誤差の分が理由として大きいだろう。  

*Renoir* は CPU Core、CPU L3$ (4MB)、GPU CU のダイ面積すべてで *RavenRidge*、*Raven2* から半分近く、半分より若干小さい面積を達成している。  
TSMC 7nm の効力だけでなく、AMD の設計における努力が窺える。  

*Renoir* と他 2つの違いには、LPDDR4xメモリの対応があげられる。  
*DDR4 128-bit PHY* のダイ面積を測定してみたところ、CPU Core等ほどは小さくなっておらず、やはり I/O の物理層は微細化効果が効きにくいと思わせるが、同時に LPDDR4 対応によるダイ面積への影響はさほどないことがわかる。  
それと、*Display PHY* の形状、配線が大きく異なるように見えたが、これは *Renoir* で *Navi1x* と同世代の DCN2.1 となり、DSC に対応したことが影響していると思われる。  
