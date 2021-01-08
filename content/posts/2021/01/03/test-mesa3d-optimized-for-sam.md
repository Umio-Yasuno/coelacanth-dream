---
title: "Smart Access Memory への最適化は本当に逆効果だったのかを確かめる　【追記】RADVに最適化が適用されてなかったため加筆修正"
date: 2021-01-03T18:12:11+09:00
draft: false
tags: [ "Smart_Access_Memory", "RadeonSI", "RADV" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU", "Experiment" ]
noindex: false
# summary: ""
toc: false
---

*RDNA 2/GFX10.3* GPU、**RX 6800 Series** と共に発表された新機能 *Smart Access Memory* (以下 SAM) だが、Windows でのサポートに合わせてか、最近になって Linux ドライバーでも最適化、サポートの強化が行なわれている。  
2020/12/07 には、**RadeonSI (OpenGL)** 、**RADV (Vulkan)** ドライバーに、*SAM* が有効な場合は VRAMドメインを変更する、バッファを確保する処理を一部変更するといった最適化を目的としたパッチが投稿された。[^optimiz-sam]  

[^optimiz-sam]: [RadeonSI/RADVドライバーに Smart Access Memory に向けた最適化が行なわれる | Coelacanth's Dream](/posts/2020/12/07/radeonsi-sam-optimization/)

しかしその後、多くの環境で性能の低下が確認されたとして、*Zen 3 + RDNA 2* 以外のシステムでは、上記の最適化をデフォルトでは適用しない、という内容のパッチが新たに投稿された。[^dis-sam]  

[^dis-sam]: [RadeonSI では Smart Access Memory をオプションで切り替える方式に、 Zen 3 + RDNA 2 ではデフォルトで有効 | Coelacanth's Dream](/posts/2020/12/24/change-of-radeonsi-sam/)

今回は、*SAM* の効果を再度検証するとともに、最適化の効果を確かめ、*Zen 3 + RDNA 2* 以前の環境で *SAM* を有効化することの意義を考えた。  

{{< pindex >}}
 
 * [環境](#env)
 * [計測方法](#how)
 * [結果](#result)
 * [考察](#consider)
    * [Zen 3 + RDNA 2 以前の環境で SAM を有効にする意義](#significance)
 * [加筆 [2020/01/05]](#revise)

{{< /pindex >}}

## 環境 {#env}

| Hardware/Software | desc |
| :-- | :--: |
| CPU | AMD Ryzen 5 2600<br>(6-Core/12-Thread, 3.4GHz) |
| M/B | GIGABYTE GA-A320-HD2 (rev 1.0) |
| Memory | DDR4-3200 16GB |
| GPU | AMD Radeon RX 560<br>(16CU/1024SP, 1080MHz,<br>VRAM 4GB, 7Gbps<br>PCIe Gen3 x8) |
| Linux Kernel | 5.10.4 |

環境は普段とほとんど変わらない。  
Linux Kernel を v5.10.4 にアップデートした点と、メモリ速度が DDR4-2933 から DDR4-3200 に向上している点は異なる。  
メモリを買い替えた訳ではなく、メモリ電圧を 1.35Vに設定したら意外にも 3200MHz で動作しちゃったため。メモリタイミング等の設定は Auto のままで一切弄っていない。見る人によっては怒りそうな設定かもしれない。  


| User Mode Driver | Mesa3D |
| :-- | :--: |
| Previous Version | Mesa 21.0.0-devel<br>([git-b02e15d1a3](https://gitlab.freedesktop.org/mesa/mesa/-/commits/b02e15d1a3))<br>(Build: 2020/12/02) |
| Recent Version | Mesa 21.0.0-devel<br>([git-9ef2c44ce6](https://gitlab.freedesktop.org/mesa/mesa/-/commits/9ef2c44ce6))<br>(Build: 2021/01/02) |

UMD (User Mode Driver) だが、*SAM* への最適化が組み込まれていない以前のバージョンと、組み込まれている最近のバージョンの 2つで検証を行なった。  
また、最近のバージョンでは *SAM* の有効無効 2つに分けても検証している。  

ドライバーの切り替えについては、レポジトリからインストールするのではなく、独自にビルドし、複数バージョンをインストールする方法であれば、`LIBGL_DRIVERS_PATH` 、 `VK_ICD_FILENAMES` といった環境変数を変更するだけで切り替えられる。  
Mesa3D のビルド方法、インストール方法については約1年前に書いたのがあるため、そちらを参考にしてほしい。近い内に加筆修正するかも。  
{{< link >}} [Radeon向けMesaビルド方法　―― RadeonSI, RADV | Coelacanth's Dream](/posts/2019/12/04/how-to-build-mesa-for-radeon/) {{< /link >}}

## 計測方法 {#how}

検証には、glxgears、[waifu2x-ncnn-vulkan](https://github.com/nihui/waifu2x-ncnn-vulkan)、[vkfft](https://github.com/DTolm/VkFFT)、vkmark、[Basemark GPU](https://www.basemark.com/benchmarks/basemark-gpu/) を採用した。  

glxgears はベンチマークではなく、動作を確認するための基本的なデモアプリケーションなのだが、それ故に GPU への負荷が小さく、CPU 性能の影響を受けやすいため検証に用いている。  
`vblank_mode=0` で垂直同期を無効化、5秒間に処理されたフレーム数が出力されるため、それを 4回分取り、平均を結果とした。  

waifu2x-ncnn-vulkan は、今回入力に使用する画像を 10枚に増やしている。また、3回実行して、実際に掛かった時間の平均を結果とした。  
画像の解像度は全て 1136x640。  
`-j` オプションで使用する VRAM量を増やすように調節することもできるが、処理に掛かる時間はほぼ変わらなかったため、デフォルトの 1:2:2 (load:proc:save) で実行した。  

    //  glxgears
    $ vblank_mode=0 glxgears & sleep 21; killall glxgears

    //  waifu2x-ncnn-vulkan 
    $ for i in {1..3}; do \
    /usr/bin/time -f "real: %es" \
    <waifu2x-ncnn-vulkan binary> -i <input dir> -n 2 -s 2 \
    -m <model path (models-cunet | models-upconv_7_anime_style_art_rgb)> \
    -o <output dir>; done

    //  vkamrk
    $ vkmark -s 1920x1080

vkfft と Basemark はそのまま実行し、表示されたスコアを結果とした。  
vkmark は 1920x1080 の解像度設定で実行している。  

## 結果 {#result}

| SAM | Prev ver<br>(SAM ON) | Recent ver<br>(SAM ON) | Recent ver<br>(SAM OFF) |
| :-- | :--: | :--: | :--: |
| glxgears (FPS) | 14353.595<br>(100%) | 12140.448<br>(84.5%) | 14296.476<br>(99.6%) |
| waifu2x-ncnn-vulkan (real sec)<br>(model: cunet) | 26.73s<br>(100%) | 26.69s<br>(100.1%) | 26.70s<br>(100.1%) |
| waifu2x-ncnn-vulkan (real sec)<br>(model: upconv_7_anime_style_art_rgb) | 10.71s<br>(100%) | 10.70s<br>(100.1%) | 10.70s<br>(100.1%) |
| vkfft (score) | 4204<br>(100%) | 4207<br>(100.1%) | 4237<br>(100.7%) |
| vkmark (score)<br>(1920x1080) | 4531<br>(100%) | 4534<br>(100.1%) | 4511<br>(100.4%) |
| Basemark GPU Vulkan (score)<br>(Medium, 1920x1080) | 23568<br>(100%) | 23797<br>(101.0%) | 23651<br>(100.3%) |


## 考察 {#consider}

結果は、*SAM* を有効化した、最適化が組み込まれていない以前のバージョンを基準とした。  

まず、*Ryzen 5 2600 (Zen+) + RX 560 (Polaris11)* という構成における *SAM* の効果だが、やはりほとんど無い。  
一応、Basemark GPU ではドライバーを最近のバージョン同士で比較して、僅かでも効果があると言えなくもないくらいには *SAM* の効果が出ており、  
今回の検証の中では Basemark GPU が一番現実的なベンチマークであるため、ゲームでは効果があるという見方もできる。  
**RADV** の開発者である [Bas Nieuwenhuizen](https://gitlab.freedesktop.org/bnieuwenhuizen) 氏によれば、{{< comple >}} ハードウェア環境は不明だが {{< /comple >}} Basemark では +3%、SotTR (Shadow of the Tomb Raider) では +1fps (0-4%) の性能向上を確認されている。[^radv-sam]  

前回は、ただ唯一 waifu2x-ncnn-vulkan を cunetモデルで実行した結果で、それなりに大きい効果が確認されたのだが、今回は同様の結果とならず、誤差の範囲となった。[^first-sam-test]  
今回程厳密に結果を取っていなかったため、*SAM* 以外の要因が影響した可能性がある。  
cunet は重いモデルと言え、upconv_7_anime_style_art_rgb モデルと比べると倍以上の時間が掛かる。そのため、問題が無ければ cunet をわざわざ使う理由は無く、それだけが速くなってもあまり意味が無かったりする。  

[^radv-sam]: [radv: Use VRAM for CS & upload buffer if all VRAM is CPU-visible. (!7979) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7979)
[^first-sam-test]: [Linux環境と Ryzen 5 2600、Radeon RX 560 で "Smart Access Memory" 機能を試す | Coelacanth's Dream](/posts/2020/11/05/linux-amd-smart-access-memory/)

glxgears では明確な差が出ており、{{< comple >}} と言っても性能は向上しておらず、むしろ {{< /comple>}} *SAM* を有効化した最近のバージョンでは FPS が 16.5% 低下している。  
glxgears を検証に用いた場合の特性と、他では性能がここまで露骨に下がってはいないことから、GPU 以外で部分でオーバーヘッドが増加していると思われる。  
また、ドライバーが最近のバージョンであっても、*SAM* を無効化すると以前のバージョンと同等の性能に戻ることから、ここが最適化が逆に働いてしまった部分なのだろう。  
上述したように、glxgears は基本的な動作をするデモアプリケーションであるから、PCIe経由の転送ではなく CPU部でオーバーヘッドが増加したと考えられる。  
繰り返しになるが、glxgears はベンチマークではないし、現実的なアプリケーションに則さない。だが、傾向を見ることはできるだろう。  

DX12、Vulkan API を用いるベンチマーク、ゲームでは効果が出やすいことは、API として層が薄く、OpenGL API より CPU部がボトルネックとなりにくいからかもしれない。  

### Zen 3 + RDNA 2 以前の環境で SAM を有効にする意義 {#significance}

これらのことから *SAM* が効果的に働くシチュエーションを考えると、4K のような高解像度設定や、テクスチャ設定をハイクオリティにした場合のような、相対的に GPU がボトルネックになりやすい、もしくは VRAM へのテクスチャ転送が FPS 低下の原因となる、VRAM をほぼ使うようなゲームに行き着く。  
GPU性能と VRAM に余裕のあるものでは、*SAM* の効果がほとんど無いか、性能が低下することが考えられる。  

*SAM* の最適化は *Zen 3 + RDNA 2* であればデフォルトで適用され、他はオプションで有効化する形となったが、*Zen 3 + RDNA 2* なら確実に効果が出るとも考えにくく、アプリケーションと解像度等の設定によっては逆効果になるのではないだろうか  
*SAM* 自体、ピーク性能を引き上げる機能というより、極端に性能が下がるのを防ぐものに見える。  
今後ミドル帯をターゲットとした *RDNA 2/GFX10.3* GPU が出たとして、そこでも同様に *SAM* 最適化がデフォルトで有効化、推奨されるかは気になる所ではある。  

*Zen 3 + RDNA 2* 以前の環境で *SAM* を有効にすることの意義だが、特別性能向上の効果は見られないが、*SAM* によるデメリットも特に無い。  
glxgears では *SAM ON + 最適化* だと明確に FPS が低下していたが、それは極端な例で、あくまでも傾向。  
また、それは最適化が逆効果だったことによるもので、今後追加のパッチが適用されればオプションで最適化を切り換えられるようになるため、目立った問題にはならない。  

組み合わせというより、**RX 560 (Polairs11)** のようなエントリー向け GPU では多くで効果が見られない。上と関連して、GPU性能に余裕がある状況が少なく、先に GPU がボトルネックとなりやすい。  
GPU性能に余裕があれば CPU によらず効果を発揮する余地があるとも考えられ、  
この点、Linux では *Above 4G Decoding* を有効にすれば *SAM* が使えることや、各社 M/Bベンダーの UEFI/BIOSアップデートにより、*Zen 2* や Intel CPU でも Windows環境で *SAM* を有効化可能となっていることは喜ばしい状況と言える。  

## 加筆 [2021/01/05] {#revise}

**RADV (Vulkan)** ドライバーでも **RadeonSI (OpenGL)** ドライバー同様の最適化が既に組み込まれているものとして上文章を書きましたが、  
実際には検証時まだ組み込まれておらず、つい先程組み込まれました。  

 * [radv: Use VRAM for CS & upload buffer if all VRAM is CPU-visible. (!7979) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7979#note_7eccb93bedebb695d0056565fe17f3cbc30978d1)

検証のほとんどに Vulkan アプリケーションを用いながら、ドライバー側に適用されているかの確認を怠っていたこと、申し訳ありませんでした。  

RADVドライバーでは、環境変数 `RADV_PERFTEST=nosam` で *SAM* への最適化を無効化できるようにされており、最適化の有無を切り換えて再度検証を行ないました。  

| SAM | Optimization | No Optimization |
| :-- | :--: | :--: |
| waifu2x-ncnn-vulkan (real sec)<br>(Model: cunet) | 26.68s | 26.69s |
| waifu2x-ncnn-vulkan (real sec)<br>(Model: upconv_7_anime_style_art_rgb) | 10.68s | 10.69s |
| vkfft (score) | 4229 | 4232 |
| vkmark (score)<br>(1920x1080) | 4605<br>(101.4%) | 4540<br>(100%) |
| Basemark GPU (score)<br>(Medium, 1920x1080) | 23970<br>(100.7%) | 23802<br>(100%) |

コンピュート系では最適化の効果が見られないが、グラフィクス系では約1%の効果が出ている、という結果となりました。  
Vulkan ゲームを普段プレイする人は、*Zen 3 + RDNA 2* 以前の CPU/GPU 環境であっても *SAM* を有効にする価値があるかもしれません。  

