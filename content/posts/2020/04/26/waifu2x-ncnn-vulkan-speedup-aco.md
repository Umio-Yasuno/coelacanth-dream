---
title: "RADV/ACO 検証: waifu2x-ncnn-vulakan の実行が最大3倍近く高速に"
date: 2020-04-26T00:04:50+09:00
draft: false
tags: [ "ACO", "GCN", "RADV" ]
keywords: [ "waifu2x-ncnn-vulkan" ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
---

Radeonの性能をさらに引き出す *ACO* だが、GPGPUではどうだろうか？ というのを今回 waifu2x-ncnn-vulkan で検証してみた。  

## インデックス

 * [ACOとは](#aco)
 * [waifu2x-ncnn-vulkan](#waifu2x-ncnn-vulkan)
 * [環境](#env)
  * [計測方法](#how)
 * [結果](#result)
  * [推測](#guess)
  * [課題](#issue)
 * [最適化で Radeon はさらに輝く](#shinning-radeon)

## ACOとは {#aco}
オープンソースのAMDGPU向けVulkanドライバー、RADVの新たなシェーダーコンパイラバックエンド。  
*ACO* の IR(Intermediate Representaions, 中間言語)[^3]は完全に SSA(Static Single Assignment form, 静的単一代入)[^2]ベースであり、  
レジスタ割り付けも SSA で行なうため、シェーダーのレジスタ要求を正確に事前計算することができる。  
Just-in-timeコンパイルを念頭に置いてに記述されており、反復の速いデータ構造を使用している。  
記述言語は C\+\+。  

[^2]: [静的単一代入 - Wikipedia](https://ja.wikipedia.org/wiki/%E9%9D%99%E7%9A%84%E5%8D%98%E4%B8%80%E4%BB%A3%E5%85%A5)
[^3]: [中間表現 - Wikipedia](https://ja.wikipedia.org/wiki/%E4%B8%AD%E9%96%93%E8%A1%A8%E7%8F%BE)

RADV はそれまで[LLVM](https://llvm.org/)をバックエンドに採用していたが、*LLVM* のプロジェクト規模は大きく、シェーダーコンパイラバックエンドとして使う上で何かしらの問題が発覚し、修正を行なったとしてもそれがリリースサイクルに乗るまでは時間がかかる。  
バックエンドを別に開発することでそれを避けられるというのが利点の1つであり、また積極的な "DivergenceAnalysis" の実装と正確なレジスタ割り付けにより、効率的なバイナリの生成を *ACO* は可能とする。  
(DivergenceAnalysis: 発散分析? SIMD実行モデルにおける性能低下原因となる条件分岐を最適化する手法。日本語の文献が見つけられなかったため、自分のこの解説はかなり怪しい。)

[X.Org Developer's Conference 2019](https://xdc2019.x.org/event/5/)での発表スライドでは、*LLVM* から使用するレジスタ量は若干増えたが、レジスタあふれの頻度はスカラレジスタで -95.91%、ベクタレジスタで -100.00%(!)とかなり減り、Waveも -5.24%減少、コードサイズは -7.90%に減少したとしている。[^6]　  
(Wave: GPUで実行するスレッドをまとめた単位。スレッド数は GCN は64スレッド、RDNA は基本32スレッド。)  
ゲーミング性能も Doom[^5]では約25%の向上を確認している。[^7]  
[Phoronix](https://www.phoronix.com/scan.php?page=home)による検証でも、平均で概ね +20%近くの性能向上が見られている。[^4]  

[^4]: [Radeon Software 20.10 vs. Upstream Linux AMD Radeon OpenGL / Vulkan Performance - Phoronix](https://www.phoronix.com/scan.php?page=article&item=radeon-software-20&num=6)
[^5]: [Steam：DOOM](https://store.steampowered.com/app/379720/DOOM/)
[^6]: <https://xdc2019.x.org/event/5/contributions/334/attachments/431/683/ACO_XDC2019.pdf#page=29>
[^7]: <https://xdc2019.x.org/event/5/contributions/334/attachments/431/683/ACO_XDC2019.pdf#page=33>

それと、*ACO* は **AMD Compiler** を縮めた名前、と開発者の Daniel Schürmann 氏は述べている。[^1]  

[^1]: [aco: Initial commit of independent AMD compiler (93c8ebfa) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/93c8ebfa780ebd1495095e794731881aef29e7d3)

最新ドライバーのビルド方法は別記事で。  
{{< link >}}[Radeon向けMesaビルド方法 | Coelacanth's Dream](/posts/2019/12/04/how-to-build-mesa-for-radeon/){{< /link >}}
## waifu2x-ncnn-vulkan
高性能なニューラルネットワークの推論フレームワーク、[ncnn](https://github.com/Tencent/ncnn)を用いた画像の高解像度化ソフトウェア。  
Vulkan APIでGPGPU処理を行なうため、広いプラットフォームに対応しており、Intel / AMD / NVIDIA GPUで実行できることが特徴。  
{{< link >}}[nihui/waifu2x-ncnn-vulkan: waifu2x converter ncnn version, runs fast on intel / amd / nvidia GPU with vulkan](https://github.com/nihui/waifu2x-ncnn-vulkan){{< /link >}}

waifu2x-ncnn-vulkan のビルド方法は別記事にて。  
{{< link >}}[waifu2x-ncnn-vulkan ビルド方法(Linux向け)](/posts/2020/04/26/how-to-build-waifu2x-ncnn-vulkan-for-linux/){{< /link >}}

ncnn はVulkan拡張 `VK_KHR_shader_float16_int8` に対応しているのだが、RADV/LLVM ではそれが有効となっているのに対し、RADV/ACO では無効とされていた。  
そのため比較検証を見送っていたが、先日 RADV/ACO での対応が mesa の masterリポジトリに組み込まれたため、検証が可能となった。[^8]  

[^8]: [radv/aco: enable 8/16-bit storage and int8/int16 on GFX8+ (5c5c2dd4) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/5c5c2dd48fe0910dc79d3187bed99a52b5ed2848)

## 環境 {#env}

| | |
| :--- | :---: |
| CPU | Ryzen 5 2600<br>(6-Core/12-Thread, 3.4GHz)
| Memory | 16GB 2933MHz |
| GPU | AMD Radeon RX 560<br>(16CU/1024SP, 1080MHz, VRAM 4GB)
| Linux Kernel | 5.4.21 |
| User Mode Driver | Mesa 20.1.0-devel (git-a64d266134) |
| Vulkan Driver | AMD RADV(LLVM) POLARIS11 (LLVM 9.0.1)<br>AMD RADV/ACO POLARIS11 (LLVM 9.0.1) |
| waifu2x-ncnn-vulkan | version<br>20200224 |

### 計測方法 {#how}
実行コマンド:

	RADV/ACO:
		$ RADV_PERFTEST="aco" time -p <waifu2x-ncnn-vulkan binary> -i "<input dir>" -o <output dir> -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"
		
	RADV/LLVM:
		$ time -p <waifu2x-ncnn-vulkan binary> -i "<input dir>" -o <output dir> -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"

waifu2x-ncnn-vulkan の設定はデノイズレベルは 1、拡大率は 2x、tile-size はデフォルトの 400、load:proc:save もデフォルトの 1:2:2。  

/tmp ディレクトリ下にテスト用画像を収めるための入力ディレクトリと、実行後の画像を収める出力ディレクトリを作成。使用する画像は5枚。  
waifu2x-ncnn-vulakn は入力元にディレクトリを指定することで、その中の画像をまとめて処理することができ、それによってプログラムの呼び出しと終了処理の時間が減るため、GPUの処理時間に焦点を当てた結果を得られる。  

使用画像もここに掲載しようと思ったのだが、面倒くさいことを避けるべく、画像に関しては解像度 1136x640、そしてソフトウェア名に似合った感じであることだけを記す。  
さすがに二次元系でパブリック・ドメインな画像は見つからなかった。  

それぞれ3回実行し、timeコマンドで得られる現実にかかった時間を平均し、結果とする。  

## 結果 {#result}

| Result<br>(models-cunet) | RADV(LLVM) | RADV/ACO |
| :--- | :---: | :---: |
| real time<br>(lower is better) | 50.75 s | 34.28 s<br>(1.48x) |

| Result<br>(models-upconv\_7\_anime\_style\_art\_rgb) | RADV(LLVM) | RADV/ACO |
| :--- | :---: | :---: |
| real time<br>(lower is better) | 22.19 s | 6.48 s<br>(3.42x) |

RADV/ACO は、RADV(LLVM) より `models-cunet` では 1.48倍、`models-upconv_7_anime_style_art_rgb` では 3.42倍も高速という結果となった。  
同じモデル、同じパラメーター、そこまで高精度なデータタイプは用いないため、出力結果の画像に違いはなかった。  

### 推測 {#guess}
まず、どちらも RADV/ACO の方が高速という結果となったが、これは *ACO* の正確なレジスタ割り付けが効いているものと思われる。  
レジスタあふれの頻度が減ったことで、そのままメモリにアクセスする頻度も減り、効率的な実行が可能となった。  
そして、Vulkan の Compute Shader で実行される waifu2x-ncnn-vulkan では、CU(Compute Unit)以外のユニットに性能が左右されにくく、*ACO* の利点が最大限発揮されたはずだ。  

また、`models-upconv_7_anime_style_art_rgb` では差が `models-cunet` よりも大きくついているが、これはモデルのサイズが前者の方が 1.1MBと半分近く( `models-cunet` は 2.6MB)小さく、レイヤー数も少ないため、その分キャッシュヒットしやすく、さらにメモリアクセスが少なくなったのではないだろうか。  

#### 課題 {#issue}
何ともぼやけた推測となってしまった。  
AMD が提供する [RGA (Radeon GPU Analyzer)](https://github.com/GPUOpen-Tools/radeon_gpu_analyzer)や[Radeon™ GPU Profiler](https://github.com/GPUOpen-Tools/radeon_gpu_profiler)といったツールを使いこなせれば、詳細な分析ができたのかもしれないが、  
それらツールでモニタやプロファイル生成することもうまくいかず、  
RADV にあるシェーダーや SPIR-V をダンプする機能から作成したファイルを使おうにもうまくいかず、断念した。  

## 最適化で Radeon はさらに輝く {#shinning-radeon}
*ACO* によって **Radeon** の性能はより引き出される。  
ゲームといったグラフィック処理では RADV(LLVM) から 1.1倍(+10%)近い性能向上を確認でき、この 10%というのは同一ダイベースの製品の性能差に設定されることが多い。Polari10ベースの RX 570 と RX 580、Navi10ベースの RX 5700 と RX 5700 XT 等がそうだ。[^9]  
ソフトウェアの最適化によって現在使っているGPUが実質1ランク上がるというのは、消費者からすれば最高と言う他ないだろう。  
今回の検証結果では、Vulkan Compute に限れば上位ダイベースの製品、2か3ランク上の性能を体感できることになる。  
こうなるともう最高という言葉では足りない。  

OSS で高性能なGPU性能を得られる。これこそ **AMD Radeon** の魅力の1つであり、何よりも勝っている点だろう。  

あと願わくば、Vulkan APIを用いたクロスプラットフォームなソフトウェアの増加と普及を……

[^9]: [Radeon RX 5700 / RX 5700 XT Linux Gaming Performance With AMDGPU 5.3 + Mesa 19.2-devel - Phoronix](https://www.phoronix.com/scan.php?page=article&item=rx-5700-july&num=1)

{{< ref >}}

 * [[Mesa-dev] [RFC] ACO: A New Compiler Backend for RADV](https://lists.freedesktop.org/archives/mesa-dev/2019-July/221006.html)
 * [Steam :: Steam for Linux :: Help us test ACO, a new Mesa shader compiler for AMD graphics!](https://steamcommunity.com/games/221410/announcements/detail/1602634609636894200)
 * <https://xdc2019.x.org/event/5/contributions/334/attachments/431/683/ACO_XDC2019.pdf>
 * [コンピュータアーキテクチャの話(313) 1つの命令で複数の演算器を動かす「SIMD」 | マイナビニュース](https://news.mynavi.jp/article/architecture-313/)
 * [レジスタ割り付け - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%AC%E3%82%B8%E3%82%B9%E3%82%BF%E5%89%B2%E3%82%8A%E4%BB%98%E3%81%91)

{{< /ref >}}
