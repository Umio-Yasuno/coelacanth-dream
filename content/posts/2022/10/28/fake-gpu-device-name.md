---
title: "Linux 環境で GPU デバイス名を偽装する"
date: 2022-10-28T19:27:43+09:00
draft: false
categories: [ "Note", "Diary" ]
# tags: [ "", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Chips and Cheese に MrFreak 氏と clamchowder 氏による、ベンチマークの結果に含まれるハードウェア情報が信用できない理由と `CPUID` 命令の結果を偽装する方法を紹介した記事が掲載されている。  

 * [Why you can’t trust CPUID – Chips and Cheese](https://chipsandcheese.com/2022/10/27/why-you-cant-trust-cpuid/)

本記事は Chips and Cheese の記事にインスパイアされたもので、GPU デバイス名を書き換える方法を紹介したい。  

前提的なことを書くと、ある API のドライバーがオープンソースで公開され、開発されている以上、その API を経由して取得される情報を書き換えるのは非常に容易である。  
`CPUID` 命令の偽装とは異なり、一部レジスタの書き換えや仮想環境を用いる必要もない。  
今回は一部パッチを当てたドライバーのビルドとインストールをして実証したが、コードの変更もビルドもそこまで手間は掛からない。ビルド時間も最近の 6/8-Core CPU であれば 5分程度で終わると思われる。  
そして GPU デバイス名を書き換えることによるメリットは、全く無いと個人的には思っている。こうした偽装する方法が広まることで、ベンチマーク結果から出てきた怪しい情報による騒ぎがインターネットから少しでも減れば良いなとは思うが。  

以下の画像は Basemark GPU での実証。  

{{< figure src="../fake_vk.jpg" >}}

Geekbench5 でも試してみたが、普通に通って結果が掲載されている。  

 * AMD Radeon RX 7300P
    * <https://browser.geekbench.com/v5/compute/5907225>
 * AMD Radeon RX 9999 Series
    * <https://browser.geekbench.com/v5/compute/5907234>

まず GPU デバイス名、あるいはそれに近い情報を得る方法として、基本的に OpenGL では `glGetString(GL_RENDERER)`、Vulkan では `VkPhysicalDeviceProperties` の `deviceName` が使われると思う。  
Mesa3D RadeonSI (OpenGL) と RADV (Vulkan)、2つのオープンソースドライバーではそこに `libdrm` API 経由で取得できる `marketing_name` を用いている。  
そして `libdrm` は AMD GPU の DeviceID と RevisionID から一致する `marketing_name` を `amdgpu.ids` のテーブルから探し、見つかった場合はそれを返す。  
そのため GPU デバイス名を書き換えたい場合は、ローカル環境にインストールされている `amdgpu.ids` を適切に編集するだけで済む。  
しかし、RadeonSI と RADV ではデバイス名の後にソフトウェア情報やチップ情報を追加する。具体的には `(polaris11, LLVM ..., DRM ...)` や `(RADV POLARIS11)` のような文字列がデバイス名の後に続く。  
先に書いたパッチは、その部分を削除するためのものとなる。  
パッチを当てれば、後は `amdgpu.ids` を編集するだけで余計な情報が付いていないデバイス名を偽装できる。  
今回は偽装を分かりやすくするために削除したが、うまく書き換えれば偽装のレベルを上げられる。コード内の文字列部分を書き換えるだけであるため、それも手間はほとんど掛からない。  

環境が無いため検証できていないが、Mesa3D における Intel GPU ドライバーでも DeviceID を元にテーブルを参照してデバイス名を返す実装になっているため、その部分を書き換えれば偽装できると思われる。  
NVIDIA GPU ドライバーはほとんどプロプライエタリではあるが、最近 (2022/05/12) に公開されたオープンソースな Linux Kernel モジュールに `DeviceID, SubSystemID, SubSystemVendorID, DeviceName` のテーブルが含まれており、同様にテーブル部を書き換えることで偽装できる可能性はある。  
Windows 向けドライバーがどうなっているかは知らないが、同様のテーブルを参照する方法であれば可能かもしれない。  

 * [open-gpu-kernel-modules/g_nv_name_released.h at main · NVIDIA/open-gpu-kernel-modules](https://github.com/NVIDIA/open-gpu-kernel-modules/blob/main/src/nvidia/generated/g_nv_name_released.h)

`CPUID` 命令から様々な情報を取得できる CPU と異なり、GPU ではキャッシュ構成等を API から取得できるようになっていないため、デバイス名を偽装することの効果は CPU よりも大きいと考えられる。  
ベンチマークソフト側の対策としては、デバイス名だけでなく、DeviceID 等も結果に含めることで偽装を見破る方法をユーザーに対して増やすことができるように思う。  
Vulkan であれば `VkPhysicalDeviceProperties` に `deviceID` も含まれている。  

ベンチマークスコアまで騙し切ることは難しいと思うが、初期サンプリングだからとか、ドライバーの成熟が進んでいないからだとか、次世代のローエンド帯かミドル帯の GPU だろうといった雑な推測で誤魔化されてしまうのではないかと思ってしまう。  
