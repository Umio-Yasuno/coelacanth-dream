---
title: "RADV/ACO検証: ncnn Benchmark"
date: 2020-04-30T20:59:29+09:00
draft: false
tags: [ "ACO", "GCN" ]
keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
---

おまけ的な記事。  

これまで [waifu2x-ncnn-vulkan](https://github.com/nihui/waifu2x-ncnn-vulkan) を検証に用いてきたが、そのフレームワークである [ncnn](https://github.com/Tencent/ncnn) はビルド時にライブラリだけでなく、ベンチマークのバイナリも生成される。  
waifu2x-ncnn-vulkan のような一般ユーザーにとって実用的なソフトウェアからは少し外れるが、数少ない純粋な Vulkan Compute が実装されたベンチマークだ、試さない理由はないだろう。  

## インデックス

 * [環境](#env)
 * [計測方法](#how)
    * [準備](#pre)
 * [結果/Result](#result)
    * [感想](#feel)

## 環境 {#env}

| Hardware/Software | Description |
| :--- | :---: |
| CPU | Ryzen 5 2600<br>(6-Core/12-Thread, 3.4GHz)
| Memory | 16GB 2933MHz |
| GPU | AMD Radeon RX 560<br>(16CU/1024SP, 1080MHz,<br>VRAM 4GB, 1750MHz/7Gbps)
| Linux Kernel | 5.4.21 |
| User Mode Driver | Mesa 20.1.0-devel (git-a64d266134) |
| Vulkan Driver | AMD RADV(LLVM) POLARIS11 (LLVM 9.0.1)<br>AMD RADV/ACO POLARIS11 (LLVM 9.0.1) |
| ncnn version | commit: b2d9325c0dec51aafeceb077ff2fadf2510afc19 |

### 計測方法 {#how}
#### 準備 {#pre}
簡単なビルド方法は以下の記事にまとめているため、ここではベンチマークに関して補足をする。  
{{< link >}}[waifu2x-ncnn-vulkan ビルド方法(Linux向け) 【2020/04/29 追記修正】 | Coelacanth's Dream](/posts/2020/04/26/how-to-build-waifu2x-ncnn-vulkan-for-linux/#ncnn){{< /link >}}

これまではGPU **RX 560** だけでの検証だったが、ベンチマーク実行バイナリ `benchncnn` にはCPUで実行することも可能だ。  
そして cmake オプションに AVX2命令セットをサポートした x86環境への最適化を行なうか選択するものがあり、自分が所有している **Ryzen 5 2600** はネイティブという訳ではないがサポートしているため、それを有効にしてビルドを行なった。  
具体的には、cmake 実行時に `-DNCNN_AVX2=ON` を付けた。  

`benchncnn` はデフォルトの状態ではビルドされるようになっており、cmake コマンド実行時に `-DNCNN_BUILD_BENCHMARK=OFF` のオプションを付けなければ、ビルドに選択したディレクトリ下に benchmark/ ディレクトリが存在するはずだ。  
そしてその下に `benchncnn` がある。  
これを実行すれば……となるが、これだけでは不十分であり、深層学習ネットワークのパラメータが記述されたファイル達を `bechncnn` と同一のディレクトリに置く必要がある。  
それらファイルは ncnn/ ルートディレクトリ下の bechmark/ に収められている。  
逆に `benchncnn` をそっちに移してもいい。こっちの方が手軽かも。  
だらだらと言葉で説明してややこしくなってしまった。そこで各ディレクトリ、ファイルの関係性を表したのが下図。  

 * ncnn/
  * \<build dir\>
        * benchmark/
             * benchncnn
  * benchmark/
     * \<param files\>

<span>

検証では実行バイナリとパラメータを収めたファイルを /tmp に置いて行なった。  
また、**Ryzen 5 2600** で実行する際、全12スレッドを使用するのではなく、物理コアのみを使用するように設定した。  
理由をここでは簡潔に書くが、性能が下がるからだ。詳細な理屈は以下の記事で。  
{{< link >}}[Zen検証 ―― 物理6-Core 2CCX vs 6-Thread 1CCX | Coelacanth's Dream](/posts/2020/03/13/zen-phy-6core-2ccx-vs-6thread-1ccx/#前提){{< /link >}}


実行コマンド:  

    Ryzen 5 2600:  
        $ taskset -c 0-5 ./benchncnn 64 6 0 -1 0

    RADV/LLVM:  
        $ ./benchncnn 64 1 0 0 0

    RADV/ACO:  
        $ RADV_PERFTEST="aco" ./benchncnn 64 1 0 0 0

実行コマンドには benchmark/ 下にある README.md を参考とした。[^1]  

結果には、 min 、max 、avg の3つが出力されるが、今回の検証では avg をスコアとして採用した。  
また、実行する項目は CPU **Ryzen 5 2600** の方が多いが、GPU **RX 560** と合わせるため、一部項目の結果を省いている。  
実行結果は gist.github.com にほぼそのまま投稿してある。別環境、別デバイスとの比較にはそちらの方が使いやすい。  
{{< link >}}[ncnn benchmark result](https://gist.github.com/Umio-Yasuno/99356254f0d1eb69864955cc9c8d91a3){{< /link >}}

</span>

[^1]: [ncnn/README.md at master · Tencent/ncnn](https://github.com/Tencent/ncnn/blob/master/benchmark/README.md)

## 結果 {#result}

| benchncnn<br>Result (.avg)<br>(lower is better) | Ryzen 5 2600<br>(6-Core, AVX2) | RX 560<br>RADV/LLVM | RX 560<br>RADV/ACO |
| :--- | :---: | :---: | :---: |
| squeezenet | 8.13 | 6.91 | 3.16 |
| mobilenet | 8.45 | 8.67 | 3.90 |
| mobilenet\_v2 | 9.88 | 8.71 | 4.75 |
| mobilenet\_v3 | 60.50 | 14.27 | 6.69 |
| shufflenet | 14.93 | 4.43 | 3.00 |
| shufflenet\_v2 | 5.66 | 6.08 | 3.90 |
| mnasnet | 112.48 | 11.02 | 5.39 |
| proxylessnasnet | 168.46 | 11.61 | 5.42 |
| googlenet | 32.74 | 28.76 | 12.15 |
| resnet18 | 25.59 | 23.13 | 7.58 |
| alexnet | 14.51 | 22.73 | 12.42 |
| vgg16 | 174.78 | 161.59 | 38.17 |
| resnet50 | 73.55 | 54.83 | 18.83 |
| squeezenet_ssd | 30.50 | 32.41 | 13.46 |
| mobilenet_ssd | 20.53 | 18.85 | 9.24 |
| mobilenet_yolo | 41.87 | 36.49 | 14.38 |
| mobilenetv2_yolov3 | 28.60 | 18.58 | 8.88 |

### 感想 {#feel}
waifu2x-ncnn-vulkan でも見られたように、やはり *RADV/LLVM* より *RADV/ACO* が高速結果を出した。  
18項目中全てで *RADV/LLVM* に勝っており、中でも vgg16 における性能向上が著しく、約4.23倍の速度だ。  

実行において、waifu2x-ncnn-vulkan の程 VRAMを使用しなかったあたりメモリコストが小さく、より *ACOバックエンド* の効果が出たのかもしれない。  

benchmark/ の README.md[^1] にある **RTX 2060** 、**RTX 2080** の結果と比較するとさすがに **RX 560** が負けるが、かなり健闘している方ではないかと思う。その点では結構嬉しい。  
規模が **RX 560** よりも大きく、FP16のPacked実行にも対応している *GFX9* 、*GFX10* のdGPUであれば勝てる可能性がある。  

[ncnn](https://github.com/Tencent/ncnn) はオープンソースで開発されており、広くシステムに対応している。ソースコードからビルドする必要があるとはいえ、Windows、MacOSでも実行可能だ。  
他のRadeon搭載システム環境下での実行結果があれば、*RADV/ACO* の価値をより正しく測れる。そうでなくとも、純粋な Vulkan Compute で実装されたベンチマークとして ncnn は貴重だろう。  
ncnn とそのベンチマーク機能の存在がもっと知られることを願っている。  
