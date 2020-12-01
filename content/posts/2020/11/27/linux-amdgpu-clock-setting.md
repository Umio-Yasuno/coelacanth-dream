---
title: "Linux環境で AMDGPU のクロック、電圧を調節　―― OC/UV"
date: 2020-11-27T23:34:37+09:00
draft: false
tags: [ "Linux_Kernel", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kenrel 部に含まれる AMDGPUドライバーには AMD GPUハードウェアのクロック、電圧を調節する機能があり、今記事はその機能の使い方の紹介。  
普段は新しい情報ばかり取り扱ってるけど、こういった布教?活動的なものも大事だと感じた次第。  

<br>
**GPU のクロック、電圧をデフォルトから変更することは故障する危険が伴うものであり、完全自己責任の元で行なってください。**

{{< pindex >}}

 * [amdgpu.ppfeaturemsk](#ppfeaturemask)
 * [クロック、電圧の調節、上書き](#clk-volt-od)
   * [Vega10 とそれ以前の AMD GPU の場合](#vega10-and-pre)
   * [Vega20 とそれ以降の AMD GPU の場合](#vega20-and-next)
 * [終わりに](#end)

{{< /pindex >}}

## amdgpu.ppfeaturemask {#ppfeaturemask}

クロック、電圧を手動で調節する機能を有効にするにはまず、Kernelパラメーターに `amdgpu.ppfeaturemask=0xffffffff` を渡す必要がある。  
具体的な方法としては、適当なテキストエディアで `/etc/default/grub` を開き、`GRUB_CMDLINE_LINUX_DEFAULT` にその値を追加する。下記では `amdgpu.ppfeaturemask` の値しかないが、実際には起動時のログメッセージ表示を抑制する `quiet` パラメーター等が他にあると思う。あっても AMD GPU の機能を有効とする上で問題となるものは殆ど無いので気にしなくていい。  

      GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.ppfeaturemask=0xffffffff"

`amdgpu.ppfeaturemask` パラメーターの意味としては、AMD GPU の電力管理機能 (Power Play, PP) 各種をとりあえず全て有効にするというもの。[^ppfeaturemask]  
デフォルトの状態でも機能のほとんどが有効にされているが、ただクロック、電圧の上書き (OverDrive, OD) だけは無効化されているため、特別 Kernelパラメーターを渡す必要がある。[^pp-parameter]  

[^ppfeaturemask]: [drm/include: add PP_FEATURE_MASK comments (v3)](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=549750a383bf1c6a4a8ba3634c85e00e7f4585da)
[^pp-parameter]: [drm/amdgpu: Change default value of module parameter amdgpu_pp_feature_mask](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=3d2fc0813f91a908f5c61ac0d08d89f802030d03)

`/etc/default/grub` の内容を変更しても、`update-grub` コマンドを実行しないと反映されないため注意。  
また、ここまでの GRUB 関連の設定変更、コマンド実行にはスーパーユーザー権限が必要となる。  

## クロック、電圧の調節、上書き {#clk-volt-od}

上記設定を適用した状態でシステムを起動させた後は、sysfs API を介した操作で調節、上書きが可能となる。  

まず、以下のコマンドを実行し、クロック、電圧を変更できる状態にする。これもスーパーユーザー権限が必要となる。  

      # echo "manual" > /sys/class/drm/card0/device/power_dpm_force_performance_level

これ以後の操作も `/sys/class/drm/card0/device` 下にある特定の sysfsインターフェイスに `echo` コマンドで値を渡して行なう。長いパスを打つのが面倒なため、カレントディレクトリを `/sys/class/drm/card0/device` へ移動しておくと楽。  

パス内 card の後の数字は GPUインスタンスに割り当てられた数字となり、GPUカード 1枚だけを搭載しているなら基本 0 となる。複数の GPU を搭載している際は注意。  
統合 GPU と混在する環境については、検証できるシステムを持ってないため、申し訳ないことに注意してとしか言えない。  
一応、`/sys/class/drm/card0/device` 下にある `device` 、`revision` の値を、それぞれ AMD GPU の DeviceID (PCI ID) 、RevisionID と照らし合わせることで知ることはできる。  
PCIバスでもそれは可能で、その場合、`/sys/class/drm/card0/device` 下にある `comsumer:` に続く数値と、`lspci | grep VGA` や `lshw -c display` を照らし合わせる。  

クロック、電圧の操作に関しては、[Vega10](/tags/vega10) {{< comple >}} Vega56、Vega64 等のベースとなる GPU のコードネーム {{< /comple >}}、それ以前の GPU と、  
[Vega20](/tags/vega20) {{< comple >}} こちらは Radeon VII 等のベース {{< /comple >}} とそれ以降 ([Navi10](/tags/navi10), [Navi14](/tags/navi14)等) とで異なる。  


### Vega10 とそれ以前の AMD GPU の場合 {#vega10-and-pre}

`pp_od_clk_voltage` に値を渡すことでクロック、電圧を調節が行なえる。  
*Vega10* とそれ以前の AMD GPU の場合、`pp_od_clk_voltage` の中身は以下のようになっている。以下の例は、自分が使ってるシステム内でずっと働いてくれている **RX 560 (Polaris11)** 場合。  


 >      OD_SCLK:
 >      0:        214MHz        715mV
 >      1:        387MHz        721mV
 >      2:        843MHz        737mV
 >      3:       1011MHz        850mV
 >      4:       1080MHz        912mV
 >      5:       1126MHz        962mV
 >      6:       1168MHz       1012mV
 >      7:       1196MHz       1050mV
 >      OD_MCLK:
 >      0:        300MHz        715mV
 >      1:        625MHz        800mV
 >      2:       1750MHz        850mV
 >      OD_RANGE:
 >      SCLK:     214MHz       1800MHz
 >      MCLK:     300MHz       2000MHz
 >      VDDC:     700mV        1150mV

AMD GPU では P-state と呼ぶ各レベルをポイントとしてクロック、電圧が、動作状況に合わせて動的に変更される。  

例えば、最大 SCLK (System Clock)/コアクロックとその電圧を変更したい時は以下のようなコマンドを実行する。  

      # echo "s 7 <設定したいクロック> <設定したい電圧>" > pp_od_clk_voltage

そうすると `pp_od_clk_voltage` の P-State 7 が書き換わるが、まだ反映はされておらず、`c (commit)` を渡すことで反映される。  


      # echo "c" > pp_od_clk_voltage

デフォルトに戻したい時は、`r (reset)` を渡し、その後で `c` を渡せばいい。  


      # echo "r" > pp_od_clk_voltage

MCLK (Memory Clock) も変更することは可能だが、変更には厳しいものがあり、少し変更しただけでディスプレイに現代アートが描かれる、ということがよくあるためオススメはしない。  

`OD_RANGE` はそのままの意味で、クロック、電圧 (VDDC) をそこから外れた値に設定することはできない。  

### Vega20 とそれ以降の AMD GPU の場合 {#vega20-and-next}

*Vega10* とそれ以前の場合と操作は特に変わらないが、`pp_od_clk_voltage` の内容が異なる。  
SCLK は最低/最高クロックの 2つのみが設定でき、MCLK はデフォルトでは最高クロックのみだが、最低クロックを追加することができる。  
電圧については、`OD_VDDC_CURVE` が追加され、そこで 3つのクロック、電圧のポイントを設定することとなる。 (最低 - 中間 - 最高)  
ユーザー側から弄ることのできる範囲は減ったが、ハードウェア側のクロック、電圧の調節機能がそれだけ優秀になったということだろう。  

## 終わりに {#end}

思ってたよりも長ったらしいものになったが、各操作は単純であるため、一度設定を詰めてしまえば、後は適当なシェルスクリプトを書いて、それを自動で実行するようにしておくだけでいい。  

上で自環境の **RX 560** のデフォルト設定を載せたが、クロック、電圧を調整し、最高で 1080MHz 850mV となるように設定した省電力運用となっている。  
GPU性能を必要とする場面が多くないため、消費電力、発熱が小さい方が自分としては嬉しいものがある。  

{{< ref >}}

 * [drm/amdgpu AMDgpu driver — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu.html)
 * [AMDGPU - ArchWiki](https://wiki.archlinux.org/index.php/AMDGPU#Overclocking)

{{< /ref >}}
