---
title: "私的conkyrc"
date: 2020-03-12T20:06:41+09:00
draft: false
tags: [ "", ]
keywords: [ "", ]
categories: [ "Diary", "Software" ]
noindex: false
---

## いつの間にかsysfsからVRAM使用量が読めるように
<img src="/image/2020/03/12/private-conkyrc.webp" style="position: relative;height:90vh; float:right; margin:.3em 0 0 .5em">
前々からデスクトップアクセサリとして[conky](https://github.com/brndnmtthws/conky)を使用しており、そのカスタマイズ性から重宝していた。  
ただGPUの情報を一部読み取れないことだけが問題だった。  

NVIDIA GPUドライバは何故かサポートされており、conky内のオブジェクトから温度、GPUクロック、メモリクロックを読み取れるようになっているが、AMDGPUはそうではなく、  
温度、それとファン回転数は `hwmon` から、GPUクロックとメモリクロックは、conky内のコマンドを実行するオブジェクトを使って `sysfs` を読み取り、表示させていた。  
しかしVRAMを読み取る方法がわからず、どれだけVRAMを使われているか気になったときはターミナルから[radeontop](https://github.com/clbr/radeontop)を実行して確かめる日々を送っていた。  
それが今日、ふと `/sys/class/drm/card0/device/` 下のファイルを確かめたところ、 `mem_info_vram_used` 、`mem_info_vram_total` が追加されていた。  
今使っているLinux Kernelは 5.4.21 だが、もしかしたらもう少し前からあったのかもしれない。  

<div style="clear:left;"></div>
## アップデート
何はともあれ、早速conkyrcをアップデートさせた。  
`mem_info_vram_used` 、`mem_info_vram_total` は Byte で表示されるため、bcコマンドでMiB表示に合わせる。  

	${offset 16}Mem Used :${alignr 4}${exec echo "$(cat /sys/class/drm/card0/device/mem_info_vram_used) / 1024 / 1024" | bc} / ${alignr 2}${exec echo "$(cat /sys/class/drm/card0/device/mem_info_vram_total) / 1024 / 1024" | bc}MiB

ついでに消費電力も表示させる。 `hwmon` が対応してるにはしているが、µW での値であるから同様にbcコマンドで W に整形。  

	${offset 16}Temp:${offset 4}${hwmon 0 temp 1}°C${alignr 6}Power: ${exec echo "scale=2; $(cat /sys/class/hwmon/hwmon0/power1_average) / 1000 / 1000" | bc}${offset 4}W

GPU/メモリクロックも `hwmon` から取得してbcコマンドで整形するようにしたことで、より正確な数値が得られるようになった。  

こうして RX 560 の状況をより監視できるようになったが、感想としてはあんまりVRAM使わないんだな、というもの。  
4096MiBあるのに、ブラウジング程度では200〜400MiBしか使わない。もっと食べていいのに。  
消費電力もプロファイルによるが、大体 5〜13W。これは少ない方が嬉しい。  

現在の .conkyrc をそのまま Github Gist にアップロードしてみたが、上記のコードから察しがつくように、恐ろしく、おぞましい程に見にくい。  
[私的conkyrc](https://gist.github.com/Umio-Yasuno/d6078888c6456ed0c9ead297e7e6623c)  
自分の書き方に問題があるのだと思い、レポジトリ内のWikiにある例を見てみたが似たようなものだった。  
[Configs · brndnmtthws/conky Wiki](https://github.com/brndnmtthws/conky/wiki/Configs)  
Vimで編集しようとするともっと最悪になる。.conkyrc に書き込みがあると起動中の conky に反映されるが、フラッシュが発生し、それがまた気分を最悪にさせる。  
.conkyrc の記述に問題があれば、お前が書いたそれは間違ってるというのを紙に記して、口に腕を突っ込み胃に直接入れるようにして教えてくれる。  
そういった試練を乗り越えて、conky は理想のデスクトップアクセサリとして機能してくれる。  

それぞれの環境でCPUのスレッド数や `hwmon` の割り振りが違うため、自分の .conkyrc を使い回すことはまず出来ないことを注意してほしい。  

## CPU
それとCPU 12スレッドを、物理コアとSMTによるスレッドを見分けられるようにしているが、あまり機能している感じはない。`/proc/cpuinfo` を参考にしたが、上から6スレッドに負荷がかかっているのをよく見る。それはそれで1CCX内で処理を回していることがわかり、良いのかもしれない。  
物理6コア(2CCX 6-Core)と、物理3コア6スレッド(1CCX 3-Core/6-Thread)。性能的にはどちらが優れているのだろう。  
tasksetコマンドで使用するコア/スレッドを指定できるため、いつかやる気が出てきたら試したい。  
