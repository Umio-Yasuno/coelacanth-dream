---
title: "Linux環境と Ryzen 5 2600、Radeon RX 560 で \"Smart Access Memory\" 機能を試す"
date: 2020-11-05T05:14:49+09:00
draft: false
tags: [ "Linux Kernel", "RADV", "RadeonSI", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

*RDNA 2 / GFX10.3* 世代の GPU、**Radeon RX 6000シリーズ** が発表された際、一緒にソフトウェアの新機能 *AMD Smart Access Memory* が発表されていた。  
{{< link >}} [AMD Smart Access Memory | AMD](https://www.amd.com/en/technologies/smart-access-memory) {{< /link >}}
*AMD Smart Access Memory* は、従来の Windows OS をベースとするシステムでは、CPU から GPU VRAM へは一度に 256MB までしかアクセス出来なかったが、その制限を取り払うことでボトルネックを減らし、ゲームにおける性能を高めるというもの。AMD が示している結果では、API に Vulkan または DX12 を用いるゲームで 5〜11% の性能向上となっている。  

その時はついに公開された *RDNA 2 / GFX10.3* の方に興味が集中していたため、ほとんどスルーしていたが、後に面白いことが判明した。  

{{< pindex >}}
 
 * [Linux では既にサポート済み](#already)
   * [いざ有効化](#enable)
 * [効果の程は](#effect)
   * [環境](#env)
   * [計測方法](#how)
   * [結果](#result)

{{< /pindex >}}

## Linux では既にサポート済み {#already}

以下は Linux Kenrel の AMD GPU ドライバーの開発者である [Alex Deucher](https://gitlab.freedesktop.org/agd5f) 氏が [Phoronix](https://www.phoronix.com/scan.php?page=home) の記事に投稿したコメント。  

 > Smart Access Technology works just fine on Linux. It is resizeable BAR support which Linux has supported for years (AMD actually added support for this), but which is relatively new on windows. You just need a platform with enough MMIO space. On older systems this is enabled via sbios options with names like ">4GB MMIO".
 >
 > {{< quote >}} [Linux Support Expectations For The AMD Radeon RX 6000 Series - Phoronix Forums](https://www.phoronix.com/forums/forum/linux-graphics-x-org-drivers/open-source-amd-linux/1215570-linux-support-expectations-for-the-amd-radeon-rx-6000-series/page4#post1215694) {{< /link >}}

*AMD Smart Access Memory* はリサイズ可能な BAR (Base Address Register) であり、この機能は既に何年も前から Linux 上ではサポートされているが、Windows では比較的新しい機能であるとのこと。  
また、古いシステムでは BIOS の `>4GB MMIO` といった名前のオプションによって有効化されていると氏は述べている。  

### いざ有効化 {#enable}

まず、CPU から GPU VRAM へ一度にアクセスできるサイズは、ユーザーモードドライバーである **RadeonSI (OpenGL) / RADV (Vulkan)** のデバッグ機能で確認できる。  

**RadeonSI** の場合は `AMD_DEBUG=info` を環境変数にセットした状態で (実行コマンドの前の記述するだけでいい)、OpenGL API を用いるソフトウェアを実行、  
**RADV** の場合は `RADV_DEBUG=info` をセットして、Vulkan API を用いるソフトウェアを実行すれば、現在搭載されている AMD GPU についての詳細情報がずらっと出力される。  
そして、その中の `vram_vis_size` が CPU が一度にアクセス可能なサイズを示している。[^vram_vis_size]  
以下は具体的な実行コマンドの一例。`AMD_DEBUG=info` を設定し、GLX の実装情報を示す `glxinfo` を実行、結果を `grep` に受け渡して抽出している。  

      $ AMD_DEBUG="info" glxinfo -B | grep vram_vis_size

自分が最初確認した時は、BIOS のあるオプションを有効化していなかったため、256MB と表示されていた。AMD が解説している通りのサイズだ。  

      vram_vis_size = 256 MB

[^vram_vis_size]: [src/gallium/winsys/radeon/drm/radeon_drm_winsys.c · d8ea50996580a34b17059ec5456c75bb0d1f8750 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/d8ea50996580a34b17059ec5456c75bb0d1f8750/src/gallium/winsys/radeon/drm/radeon_drm_winsys.c#L364) <br> [src/amd/common/ac_gpu_info.c · 3c2489d2e45b3013361c7284ed9de14fe40554cc · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/3c2489d2e45b3013361c7284ed9de14fe40554cc/src/amd/common/ac_gpu_info.c#L345)

次に BIOS側の設定となるが、`>4GB MMIO` といったオプションは `Above 4G Decoding` という名前で存在している。自分が持っている **GIGABYTE GA-A320-HD2 (rev. 1.0)** (現在のメイン機) ではそのようになっており、他社製のマザーボードでもそのようになっているのではないかと思われる。  
それを Enabled に設定して起動、上述したようなコマンドで確かめた結果、`vram_vis_size` の値は、搭載している **RX 560 (Polaris11)** が持つ VRAM と同じサイズである 4096MB となっていた。  

      vram_vis_size = 4096 MB

[Alex Deucher](https://gitlab.freedesktop.org/agd5f) 氏のコメント通り、Linux環境では既にサポートされており、**Ryzen 5 2600 (Zen+)** と **RX 560** という構成でも有効化が可能だった。  

## 効果の程は {#effect}

有効化できたとして、その効果が気になる所だが、まず検証環境に問題がある。  

AMD が示している **AMD Smart Access Memory** 機能による性能向上は、**Ryzen 9 5900X** と **RX 6800 XT** というハードウェア構成で、現実的な使用状況、Vulkan/DX12 のゲームを 4K Ultra 設定で実行している。  
対し自分のハードウェア構成は **Ryzen 5 2600** と **RX 560** という構成で、絶対的な性能に圧倒的な差があり、CPU-GPU のインターフェイスも **RX 560** が Gen3 x8、**RX 6800 XT** は Gen4 x16 だろうから、**RX 6800 XT** の PCIe帯域は 4倍広い。  

そして、AMD はボトルネックを削減できるとしているが、どういった処理がボトルネックになっていて、何のソフトウェアであればそれを検証できるか、という問題もある。  
自分は Linux でも動作する Vulkan ゲームなんて所有しておらず、現実的な使用状況を高度に再現できる Vulkan ベンチマークソフトというのもない。  

それでも一応、VRAM をそれなりに使うソフトとしては [waifu2x-ncnn-vulkan](https://github.com/nihui/waifu2x-ncnn-vulkan) が思い浮かんだため、試してはみた。waifu2x-ncnn-vulkan は **RX 560** の VRAM 4GB をちゃんと使い切ってくれる。  
それと、グラフィクス系として vkmark (1920x1080) を実行したが、vkmark はあまり VRAM を使わないため、CPU からアクセスすることもあまり無いのではないかと思われる。  

#### 環境 {#env}

| Hardware/Software | Desc |
| :-- | :--: |
| CPU | AMD Ryzen 5 2600<br>(6-Core/12-Thread, 3.4GHz) |
| M/B | GIGABYTE GA-A320-HD2 (rev 1.0) |
| Memory | DDR4-2933 16GB |
| GPU | AMD Radeon RX 560<br>(16CU/1024SP, 1080MHz,<br>VRAM 4GB, 7Gbps<br>PCIe Gen3 x8) |
| Linux Kernel | 5.9.1 |
| User Mode Driver | Mesa 20.3.0-devel (git-5957b0c162) |

#### 計測方法 {#how}

実行コマンド:

      waifu2x-ncnn-vulkan:
         $ time -p <waifu2x-ncnn-vulkan binary> -i "<input dir>" -o "<output dir>" -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"

      vkmark (1920x1080):
         $ ./vkmark -s 1920x1080

いつかに RADV/ACO の検証した時とハードウェア構成は変わらず、Linux Kenrel や Mesa のバージョンだけが上がっている。[^radv-aco-test]  
実行コマンドも変わらず、/tmp 下に置いた入力用の画像 5枚を収めたディレクトリを指定し、出力画像も /tmp 下に作成したディレクトリを指定している。  

waifu2x-ncnn-vulkan の設定はデノイズレベルは 1、拡大率は 2x、tile-size はデフォルトの 400、load:proc:save もデフォルトの 1:2:2。
使用画像の解像度は全て 1136x640。  

[^radv-aco-test]: [RADV/ACO 検証: waifu2x-ncnn-vulkan の実行が最大3倍近く高速に | Coelacanth's Dream](/posts/2020/04/26/waifu2x-ncnn-vulkan-speedup-aco/)

#### 結果 {#result}

| waifu2x-ncnn-vulkan<br>(real time) | SAM Off | SAM On |
| :-- | :--: | :--: |
| cunet | 14.27s | 12.71s<br>(+12%) |
| upconv\_7\_anime\_style\_art\_rgb | 5.93s | 5.90s<br>(+0.5%) |

| vkmark<br>1920x1080 | SAM Off | SAM On |
| :-- | :--: | :--: |
| Score | 4729 | 4748<br>(+0.4%) |

cunet モデルでの結果は *AMD Smart Access Memory* 有効で 12% の性能向上が確認できたが、正直有意な性能向上が確認できたのはこれくらいで、他の upconv\_7\_anime\_style\_art\_rgb モデルでの結果と vkmark の結果は誤差だろう。  
AMD が **Zen 3 + RDNA 2** の構成に限定しているのは、安定して性能向上の効果を得られないからなのかもしれない。  
あるいはアクセス可能なサイズを大きくする以外に、別の工夫を施しているか。  

ただ、上述したように、システム構成がアレであるため、自分の構成でまともに検証できるかは怪しく、また、これ以上に労力を掛ける気が起きない。  
現実的なゲームに近いベンチマークとなると、モデルやテクスチャの格納で必要なファイルサイズもいやに大きくなり、テストに使う時間もかなり掛かる。  
自分でハードウェア/ソフトウェアの検証というものをやってみると、いかにテクニカルライターの方々は忍耐強いかが身に沁みてくる。  
もっとまともなシステムを持ち、経験豊富な方が検証を行なってくれることを願う。  

また、`Above 4G Decoding` を有効にすると、Linux Kenrel をブートし、ログイン時間に辿り着くまでの時間が若干増えた。精々 1秒前後だが。  
そういうことで、自分の用途では `Above 4G Decoding` を有効にする意味はほとんど無いのだが、  
最新世代の組み合わせで利用できる新機能として謳われるものを、1、2世代前の構成で有効にしているというのは中々に面白い状況に思えるため、このまましばらくは有効にしておくつもりでいる。  

