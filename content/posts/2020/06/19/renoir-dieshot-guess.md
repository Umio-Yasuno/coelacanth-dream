---
title: "AMD Renoir のダイ観察 & 推測"
date: 2020-06-19T01:41:57+09:00
draft: false
tags: [ "Renoir", "gfx909", "DieShot", "GCN", "GFX9" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

これまでにも数多くのチップを撮影、その内部構造を詳細に公開してきた [Fritzchens Fritz | Flickr](https://www.flickr.com/photos/130561288@N04/) 氏が、  
先日、*Zen 2* CPU と *Vega* GPU を持ったAMD の 7nm APU、[Renoir](/tags/renoir)のダイショットをパブリック・ドメインで公開した。  
{{< link >}}[AMD@7nm@Zen2@Renoir@Ryzen_3_4300U@100-000000085_9JB4977P00… | Flickr](https://www.flickr.com/photos/130561288@N04/50017165886/in/photostream/){{< /link >}}
氏の測定によると、*Renoir* のダイサイズは 155.01mm<sup>2</sup>。  

そして、氏のダイショットを元に内部構造を推測、各ユニットを区分けしてみた。  
**？** とその数は自信の無さを表す。  

{{< pindex >}}

 * [Renoir 推測](#renoir-guess)
    * [Renoir GPUスペック](#renoir-gpu-spec)
 * [解説的な](#explain)
    * [3つの 共有CUキャッシュ](#shared-cu-cache)
    * [5つの Display PHY](#five-display-phy)
    * [PCIeレーン](#pcie-lane)
 * [Zen 2](#zen-2)

{{< /pindex >}}

## Renoir 推測 {#renoir-guess}

{{< figure src="/image/2020/06/19/renoir-dieshot-guess.webp" title="AMD Renoir" caption="DieSize: 155.001mm<sup>2</sup><br>画像元:<cite>[AMD@7nm@Zen2@Renoir@Ryzen_3_4300U@100-000000085_9JB4977P00… | Flickr](https://www.flickr.com/photos/130561288@N04/50016639913/)</cite>" >}}

### Renoir GPUスペック {#renoir-gpu-spec}
大体の *Renoir* GPUスペックは以下となっている。  

 * 1-SE(ShaderEngine) [^1]
   * SE あたりの RB(RenderBackend)数 2基 [^1]
   * SE あたりの CU数 8基 [^1]
 * 総 L2cache サイズ 1MB (4x 256KB) [^1]
 * ディスプレイコントローラ数 4基 [^2]
 * 2つの USB-C からそれぞれ画面出力が可能 [^4][^5]

[^1]: <https://github.com/GPUOpen-Drivers/pal/blob/e1b2dde021a2efd34da6593994f87317a803b065/src/core/os/nullDevice/ndDevice.cpp#L926>
[^2]: <https://www.x.org/wiki/RadeonFeature/#radeondisplayhardware>
[^4]: [AMD Ryzen Mobile 4000シリーズの詳細を明らかに (1) Ryzen Mobile 4000シリーズのアーキテクチャ詳細 | マイナビニュース](https://news.mynavi.jp/article/20200316-997459/)
[^5]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/dc/dcn21/dcn21_resource.c?h=amd-staging-drm-next&id=fb5b6e40b4938444bb489ba892598f01b52c4f26#n804>

前世代の APU *Raven* と同じ *GFX9 /Vega* GPU ではあるが、*Raven* の *gfx902* に存在したバグを修正し、ハードウェアでのシェーダーの電力プロファイル機能に対応した[^3] *gfx909* となった。  

[^3]: <https://github.com/GPUOpen-Drivers/pal/blob/82390674c40402c81dbf311a5ee488fb972894f5/src/core/hw/gfxip/gfx9/gfx9Device.cpp#L4224>

### 解説的な {#explain}
#### 3つの 共有CUキャッシュ {#shared-cu-cache}
*Renoir* もやはりというか、複数の CUで共有する L1cache (Scalar 16KB, Instruction 32KB) が8CU に対して 3基あり、2基または 3基の CUで共有する構成だった。  
これはどうも *GFX9 /Vega* アーキテクチャで共通する特徴らしく、*Vega10 /Vega20* は 1-SE あたりの 16CU に対して L1cache 6基[^11]、*Raven* は 11CUに対して L1cache 4基となっている。  

[^11]: [AMD Vega10/Vega20のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/03/24/vega10-vega20-dieshot-guess/)

GCNアーキテクチャでは L1cache を最大 4CUで共有することが可能であり、*GFX9 /Vega* の前世代、*GFX8* 世代の *Polaris11* は 8CU に対して L1cache 2基であり[^7]、*Renoir* もそうした構成が取れたはずだ。  
AMD の資料では、*Vega10* も変わらず 4CUで共有することになっているため[^6]、歩留まり向上策なのかもしれない。  

[^6]: <https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf#page=27>
[^7]: [Polaris10/Polaris11のダイ観察 & 推測 | Coelacanth's Dream](/posts/2020/03/30/polaris10-polaris11-dieshot-guess/#polaris11)

#### 5つの Display PHY {#five-display-phy}
ディスプレイコントローラ 4基に対して、物理層(PHY)が 5基となるが、  
Linux Kernel の AMD GPU でディスプレイに関する部分を見ても[^5] PLL(Phase Looked Loop) と DDC(D/Dコンバーター)が 5基あるとされ、3x HDMI/DisplayPort + 2x USB-C の中で選択して 4画面を出力できるということだと思われる。  

#### PCIeレーン {#pcie-lane}
*Renoir* の持つ PCIeレーン数は、5x 4-Lane + 2x 2-Lane で計 24-Laneある **ように** 見えた。  
AM4ソケットの仕様で考えると、16-Lane をディスクリートGPUに、4-Lane を NVMe SSDに、4-Lane をチップセットとの接続に充てられそうだが、  
2x 2-Lane は USB にも使わられるはずで、そして AM4ソケットは CPU から 4つの USB 3.2 Gen1/2 を出す。  
<del>そのため、デスクトップ向け *Renoir* が使えるのは 20-Laneで、  
ディスクリートGPUは前世代から変わらず 8-Lane、NVMe SSD に 4-Lane、チップセットに 4-Lane、その他 PCIeスロットや NVMe SSD に 4-Lane、という感じになるのではないかと思う。</del>  

{{< ins >}}

**ASRock B550 PG Velocita** 、**GIGABYTE B550 AORUS PRO (rev. 1.0)** のマニュアルには、  
*Renoir* 、*New Generation AMD Ryzen™ with Radeon™ Graphics processors* で PCIe 3.0 x16が利用可能と記述されており、自分が USB の分を勘違いしてるか見落としているだけの可能性が高い。  
{{< link >}}<https://download.asrock.com/Manual/B550%20PG%20Velocita.pdf#page=32>{{< /link >}} {{< link >}}<https://download.gigabyte.com/FileList/Manual/mb_manual_b550-aorus-pro-ac_1001_e.pdf#page=7>{{< /link >}}

GIGABYTE は PCIeスロット、M.2、USBそれぞれが CPU側か B550 PCH側かをしっかり書いてあり、とても分かりやすく、確かだと言える。  

{{< /ins >}}

## Zen 2 {#zen-2}
{{< figure src="/image/2020/06/19/zen2-core-renoir.webp" title="Zen 2 CPU Core" caption="画像元:<cite>[AMD@7nm@Zen2@Renoir@Ryzen_3_4300U@100-000000085_9JB4977P00… | Flickr](https://www.flickr.com/photos/130561288@N04/50016639913/)</cite><br>参考:<cite>[Zen 2: The AMD 7nm Energy-Efficient High-Performance x86-64 Microproc…](https://www.slideshare.net/AMD/zen-2-the-amd-7nm-energyefficient-highperformance-x8664-microprocessor-core)</cite>" >}}

ついでに *Zen 2* CPUコアの内部も区分けしてみたが、内容としては ISSCC 2020 で AMD が発表した資料を丸写ししただけ。[^8]  
言い換えれば *Renoir* で CPU部は L3cacheの規模以外に大きく弄っていないということになる。  
しかし小さい改良点はあり、Spectre v2対策に `IBRS_FW` が追加されている。[^9][^10]  
それにどれほどの効果があるのかはわからん。  

[^8]: [Zen 2: The AMD 7nm Energy-Efficient High-Performance x86-64 Microproc…](https://www.slideshare.net/AMD/zen-2-the-amd-7nm-energyefficient-highperformance-x8664-microprocessor-core)
[^9]: [Ryzen 7 4700U Windows 10 Vs. Ubuntu Linux Benchmarks - OpenBenchmarking.org](https://openbenchmarking.org/result/2005141-PTS-WINLINUX22)
[^10]: [AMD Ryzen 9 3900X Linux Benchmarks Performance - OpenBenchmarking.org](https://openbenchmarking.org/result/1907073-HV-RYZEN939067)
