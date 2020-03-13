---
title: "Zen検証 ―― 物理6-Core 2CCX vs 6-Thread 1CCX"
date: 2020-03-13T01:41:46+09:00
draft: false
tags: [ "Ryzen" ]
keywords: [ "", ]
categories: [ "Hardware", "CPU" ]
noindex: false
---

今更 Ryzen 5 2600 を使って *Zen/+* の検証。  
試したい、というだけの理由で *Zen 2* を買える程のお金がない。さらに言うと、2666MHzでCL1も安定しないような<del>クソ</del>メモリを使っている。  

## 前提
[1つ前の記事](/posts/2020/03/12/private-conkyrc/)で、「なんか1CCXで処理が回されやすいっぽい」とか書いたが、それがどの程度性能に影響を与えるのか気になった。  
6スレッドで処理するとして考えられるパターンは主に、1CCX内の3-Core/6-Thread、それと2CCXに跨る6-Coreの2つがある。  
前者は1CCX内に収まるため短いレイテンシでスレッド間の通信、同期を行なえるが、SMT機能によって1-Coreで2-Threadを実行するため、実行ユニットのリソース競合、スレッド切り替えのオーバーヘッド等で性能向上が見られない、もしくは低下することが起こりうる。片方がメモリからデータをロードしている間、もう片方のスレッドを実行する、という風に実行ユニットの有効活用が可能だが、1-Threadあたりのキャッシュ量も単純に半分となるため、処理によってはキャッシュミスが頻発、メモリアクセスが増える要因にもなる。以下、このパターンを *6-Thread 1CCX* とする。  
後者はCCXを跨るため、その分レイテンシが長くなるが、SMT機能の悪影響をほぼ受けない。以下、このパターンを *6-Core 2CCX* とする。  

ベンチマークについては、整数や浮動小数点の演算性能を測るものでは *6-Thread 1CCX* において上述したリソースの競合が頻発し、性能が低下しやすく、現実的でないと判断した。  
怖くなって一応計測してみたが、やはり *6-Thread 1CCX* のスコアが *6-Core 2CCX* よりもだいぶ低かった。  

|  | 6-Core 2CCX | 6-Thread 1CCX |
| :-- | :---: | :---: |
| Dhrystone (整数演算) (lps) | 267065871.8 | 177565964.9 (-34%) |

lps (Lines Per Second)は1秒あたりの実行回数を表す。  

## 方法
### ベンチマークソフト
ベンチマークソフトには、歴史ある[byte-unixbench](https://github.com/kdlucas/byte-unixbench) の一部、それとOpenMPを有効にした[m-queens](https://github.com/sudden6/m-queens)、レイテンシの測定に [sysbench](https://github.com/akopytov/sysbench) を採用した。  
出来る限り上述したようなテストとは違うものを選んだつもりだが、やはり現実的なベンチマークというのは難しく、他に適したベンチマークソフトがあれば教えて頂けると嬉しい。それがLinuxに対応しており、計測が特別難しくなければ試すつもりでいる。  

#### byte-unixbench

 * Execl Throughput
 	* 1秒間に実行可能な `execl` 呼び出しの回数を測定するテスト。exec系の関数は現在のプロセスイメージを新しいプロセスイメージに置き換える。
 * Pipe-based Context Switching
 	* 2つのプロセスがパイプを通じて、増加する整数データを交換できる回数を測定するテスト。実際のアプリケーションの動きに近い。
 * Process Creation
 	* プロセスを作成、取得、終了を繰り返すテスト。それは新しいプロセスの制御とメモリ割り当てを繰り返すことであり、計測結果はメモリ帯域幅を反映する。
 * Shell Scripts
 	* プロセスが1分間で、データファイルに一連の変換を行なうシェルスクリプトを開始できる回数を測定するテスト。

<https://github.com/kdlucas/byte-unixbench/blob/master/README.md>

#### m-queens
Phoronix Test Suiteのプロファイルを参考に、以下のコマンドでビルドした。  

	CFLAGS="-O2 -march=znver1" g++-8 -fopenmp main.c -o m-queens.bin

[OpenBenchmarking.org - m-queens v1.1.0 Test [m-queens]](https://openbenchmarking.org/innhold/707926477c5cf3e3b33d1b9375523aee5f3d2020)  
N(クイーンの数は)17とし、計測結果５回の平均をスコアとする。  

### CPU
まず、Ryzen 5 2600のトポロジは以下のようになっている。  
{{< figure src="/image/2020/03/13/r5-2600-topo.webp" title="Ryzen 5 2600 トポロジ" caption="lstopo の実行結果" >}}
物理コアに数字を割り振ってから、SMTによる論理コアに割り振っている。  
そしてLinuxでは `taskset` コマンド[^1]で起動中のプロセス、または新たなコマンドを実行するCPUコアを指定することができる。  
Ryzen 5 2600のトポロジを参考にすると、  

	taskset -c 0-5 ./<ベンチマークの実行バイナリ> -c 6

で *6-Core 2CCX* を、

	taskset -c 0-2,6-8 ./<ベンチマークの実行バイナリ> -c 6

で *6-Thread 1CCX* を指定して実行させられる。  

[^1]: [taskset(1): retrieve/set process's CPU affinity - Linux man page](https://linux.die.net/man/1/taskset)

また、省電力機能による誤差を減らすため、CPUクロックは3.6GHzで固定して計測を行なった。  

## 環境
| | |
| :-- | :---: |
| Linux Kernel | 5.4.21 (Custom) |
| CPU | Ryzen 5 2600 (6-Core/12-Thread, 3.6GHz) |
| RAM | DDR4 16GB 2666MHz |
| Disk | SSD 250GB WDS250G2B0A |
| UnixBench | ver 5.1.3 |

## 結果

| Test | 6-Core 2CCX | 6-Thread 1CCX |
| :-- | :---: | :---: |
| Excel Throughtput (lps) | 24743.5 | 24262.5 (-2%) | 
| Process Creation (lps) | 51217.1 (-13%) | 58572.1 |
| Pipe-based Context Switching (lps) | 1734733.0 | 1279005.9 (-26%) |
| Shell Scripts (1 concurrent) (lpm) | 43287.2 | 35119.4 (-19%) |
| Shell Scripts (8 concurrent) (lpm) | 6677.0 | 5137.8 (-23%) |
| Shell Scripts (16 concurrent) (lpm) | 3363.9 | 2595.7 (-23%) |
| m-queens (Solution/s) | 12357498.9 | 9799726.6 (-21%) |

| sysbench threads | 6-Core 2CCX | 6-Thread 1CCX |
| :-- | :---: | :---: |
| events/s (eps) | 26374.6090 | 25598.8243 |
| Latency (ms) |
| &emsp;&emsp;min | 0.15 | 0.15 |
| &emsp;&emsp;avg | 0.23 | 0.23 |
| &emsp;&emsp;max | 9.23 | 6.15 |
| &emsp;95th percentile | 0.28 | 0.25 |

| sysbench mutex (64threads) | 6-Core 2CCX | 6-Thread 1CCX |
| :-- | :---: | :---: |
| events/s (eps) | 21.2456 | 21.2207 |
| Latency (ms) |
| &emsp;&emsp;min | 1602.09 | 2318.88 |
| &emsp;&emsp;avg | 2842.96 | 2902.36 |
| &emsp;&emsp;max | 3000.67 | 2997.10 |
| &emsp;95th percentile | 2985.89 | 2985.89 |

## 考察
正直、少し意外な結果だった。あんなメモリを使っているため、CCX間通信のオーバーヘッドが大きく、*6-Thread 1CCX* のが勝つのではないかと思っていた。レイテンシだけではなく一部性能でももしかしたらと。    
だが *6-Core 2CCX* が勝った。  
やはり物理コアは正義か。  

byte-unixbench では `Process Creation` のみ *6-Thread 1CCX* が勝っているが、これはテストがメモリ帯域幅を反映する傾向にあり、メモリ割り当てではCPUのキャッシュが効きにくいため、CCX間通信が発生する *6-Core 2CCX* の方が遅くなった……ということなのだろうか。  
[sysbench](https://github.com/akopytov/sysbench) と [stream](https://github.com/jeffhammond/STREAM) でもメモリ帯域を計測したところ、sysbench では *6-Thread 1CCX* が広いメモリ帯域を示したが、stream では逆に *6-Core 2CCX* が勝った。  
しかし stream では差が2%程しかなかったのに対し、sysbench は read で約36%、write で約23%の差が見られた。  
CCXを跨ぐことでメモリ帯域に影響があるが、具体的にどの部分が原因となっているかは今後の課題としたい。  
より転送速度が高いメモリであれば、InfinityFabricの速度も向上するため、その差が縮まるといった結果が見られるかもしれない。  

sysbench では *6-Thread 1CCX* がレイテンシにおいて、`max`、`95th percentile` の値が *6-Core 2CCX* より小さいが、 `events/s` では *6-Thread 1CCX* が負けている。  
1CCX内に収まることで、確かにレイテンシが短くなったが、性能に影響が出る程ではない、またはリソースの競合が発生したか。  

それと気をつけたいのは、今回の検証はCCXを跨ぐ場合のオーバーヘッドがどれだけあるかを確かめるのが主であり、SMTの効果の検証ではないということだ。それなら *6-Core/6-Thread* と *6-Core/12-Thread* を比較するのが適当だろう。  

## 結論
結論として、CCX間通信にオーバーヘッドがあっても、それはSMTによる論理コアで処理するより性能への影響はずっと小さく、物理コアに割り振るメリットで容易にカバーが可能と言える。  
しかし、メモリ帯域に関してそれが確実に当てはまるとは言えない。  

*Zen 2* アーキテクチャであれば、ロード/ストアユニットの増設、L3キャッシュの増量、同CCD内のCCXでもI/Oダイ経由する、といったことからもう少し違う結果、差となるかもしれない。  
そういった点を踏まえて検証してみたいが、2018年発売のRyzen 5 2600を今になって検証しているのだから、*Zen 2* を試せるのは来年あたりだろうか。  

