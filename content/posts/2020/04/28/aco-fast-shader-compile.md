---
title: "RADV/ACO 検証: ACOの高速シェーダーコンパイル"
date: 2020-04-28T23:48:27+09:00
draft: false
tags: [ "ACO", "GCN", ]
keywords: [ "RADV", "ACO", "Radeon" ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
---

## インデックス

 * [ACO のシェーダーコンパイル速度検証](#aco-shader-compie-speed-test)
 * [環境](#env)
  * [計測方法](#how)
 * [結果](#result)
  * [推測](#guess)
 * [やはり ACO は優秀](#aco-great)
 

## ACO のシェーダーコンパイル速度検証 {#aco-shader-compie-speed-test}
前回の検証で *ACOバックエンド* は Vulakn Compute による GPGPU用途においても、いやそれどころかグラフィック用途よりもずっと効果を発揮することがわかった。  
{{< link >}}[RADV/ACO 検証: waifu2x-ncnn-vulakan の実行が最大3倍近く高速に | Coelacanth's Dream](/posts/2020/04/26/waifu2x-ncnn-vulkan-speedup-aco/){{< /link >}}
しかし、*ACO* の特徴の1つであるコンパイル速度を検証できていなかった。[^1]  

[^1]: <https://xdc2019.x.org/event/5/contributions/334/attachments/431/683/ACO_XDC2019.pdf#page=34>

そこで、*RADVドライバー* のデバック機能にある `nocache` オプションでシェーダーキャッシュを無効化した際と、通常通りキャッシュを有効とした際のユーザーモードのCPU時間(User CPU time)を計測し、それらを比較して大体のシェーダーコンパイルに掛かった時間を割り出すこととした。  
時間の計測にはtimeコマンドを用いる。  

## 環境 {#env}

| | |
| :--- | :---: |
| CPU | Ryzen 5 2600<br>(6-Core/12-Thread, 3.4GHz)
| Memory | 16GB 2933MHz |
| GPU | AMD Radeon RX 560<br>(16CU/1024SP, 1080MHz,<br>VRAM 4GB, 1750MHz/7Gbps)
| Linux Kernel | 5.4.21 |
| User Mode Driver | Mesa 20.1.0-devel (git-a64d266134) |
| Vulkan Driver | AMD RADV(LLVM) POLARIS11 (LLVM 9.0.1)<br>AMD RADV/ACO POLARIS11 (LLVM 9.0.1) |
| waifu2x-ncnn-vulkan | version<br>20200224 |

### 計測方法 {#how}
実行コマンド:

	RADV/LLVM (nocache):
		$ RADV_DEBUG="nocache" time -p <waifu2x-ncnn-vulkan binary> -i "<input dir (five images)>" -o <output dir> -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"

	RADV/LLVM (cache):
		$ time -p <waifu2x-ncnn-vulkan binary> -i "<input dir (five images)>" -o <output dir> -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"

	RADV/ACO (nocache):
		$ RADV_DEBUG="nocache" RADV_PERFTEST="aco" time -p <waifu2x-ncnn-vulkan binary> -i "<input dir (five images)>" -o <output dir> -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"

	RADV/ACO (cache):
		$ RADV_PERFTEST="aco" time -p <waifu2x-ncnn-vulkan binary> -i "<input dir (five images)>" -o <output dir> -n 1 -s 2 -m "<waifu2x-ncnn-vulkan model>"

waifu2x-ncnn-vulkan の設定はデノイズレベルは 1、拡大率は 2x、tile-size はデフォルトの 400、load:proc:save もデフォルトの 1:2:2。  

/tmp ディレクトリ下にテスト用画像を収めるための入力ディレクトリと、実行後の画像を収める出力ディレクトリを作成。使用する画像は5枚。  
waifu2x-ncnn-vulakn は入力元にディレクトリを指定することで、その中の画像をまとめて処理することができ、それによってプログラムの呼び出しと終了処理の時間が減るため、GPUの処理時間に焦点を当てた結果を得られる。  

それぞれ3回実行し、timeコマンドで得た時間を平均したものを結果とする。  

## 結果 {#result}

| Result<br>(models-cunet) | RADV(LLVM)<br>(nocache) | RADV(LLVM)<br>(cache) | RADV/ACO<br>(nocache) | RADV/ACO<br>(cache) |
| :--- | :---: | :---: | :---: | :---: |
| Real time<br>(lower is better) | 58.17s | 51.69s<br>(-7.48s) | 34.97s | 34.33s<br>(-0.64s) |
| User CPU time<br>(lower is better) | 11.86s  | 4.45s<br>(-7.41s) | 5.35s | 4.56s<br>(-0.79s) |

| Result<br>(models-upconv\_7\_anime\_style\_art\_rgb) | RADV(LLVM)<br>(nocache) | RADV(LLVM)<br>(cache) | RADV/ACO<br>(nocache) | RADV/ACO<br>(cache) |
| :--- | :---: | :---: | :---: | :---: |
| Real time<br>(lower is better) | 25.16s | 22.15s<br>(-3.01s) | 6.71s | 6.45s<br>(-0.26s) |
| User CPU time<br>(lower is better) | 7.52s  | 4.52s<br>(-3.00s) | 4.77s | 4.53s<br>(-0.24s) |

### 推測 {#guess}
モデルを変えてもシェーダーキャッシュ有効時のCPU時間はほぼ変わらず誤差の範囲に収まっており、このことから無事キャッシュヒットしていると考えられる。  
そして *RADV/ACO* は *RADV(LLVM)* の 9.38〜12.50倍の速度でシェーダーコンパイルを完了している。  
*RADV(LLVM)* におけるキャッシュ有効時の高速化はもちろん評価すべきだが、それよりもずっと *RADV/ACO* のコンパイル速度が凄まじい。  
キャッシュ有効時の性能向上が小さいことは、それだけシェーダーコンパイラーとして優秀であることを示す。  

## やはり ACO は優秀 {#aco-great}
*ACOバックエンド* はレジスタ割り付け等の性能だけでなく、コンパイル速度も *LLVM* より優ることが証明された。  
グラフィック用途ではGPGPU用途よりも *ACOバックエンド* の効果が小さいものとなるが、ゲームのような長期的なワークロードではそれも無視できない。  
性能向上、高速化は言い換えれば省電力化だ。それにCPU時間が短ければバックグラウンド処理にも余裕が出てくる。  

*ACOバックエンド* の開発が進み、安定性が向上、そしてデフォルトで有効にされる日が来ることを願って止まない。  

余談となるが、個人的に期待しているソフトウェア側の新機能には *NGG (Next Generation Geometory) / Primitive Shader* がある。  
これは *Vegaアーキテクチャ /GFX9* と同時に登場した機能ではあるが、その世代でのサポートは廃止されてしまった。[^2]  
その代わり *RDNAアーキテクチャ /GFX9* 世代での実装に注力し、開発が進められている。  
そしてPS5でもサポートすることが明言されており、「今度こそ」という気概が感じられる。[^3]  

タイムライン的に 現 *RDNA* ではなく、次 *RDNA 2* くらいに正式サポートが為されそうだが、楽しみであることに変わりはない。  
その時はいい加減自分も堪忍して **RX 560** から卒業することだろう、たぶん。  

[^2]: [Making a GDS Allocation for NGG](https://lists.freedesktop.org/archives/amd-gfx/2018-August/025320.html)
[^3]: [西川善司の3DGE：Mark Cerny氏のPS5技術解説プレゼンテーションを読み解く(前編)。ここまで分かったPS5のSSDとGPUの詳細 - 4Gamer.net](https://www.4gamer.net/games/990/G999027/20200319173/#PS5ではAMDの「プリミティブシェーダ」を採用)

{{< ref >}}

 * [Environment Variables](https://www.mesa3d.org/envvars.html)

{{< /ref >}}
