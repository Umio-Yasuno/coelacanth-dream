---
title: "Radeon RX 5600推測"
date: 2019-12-18T04:45:32+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "Radeon RX 5600" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

少しずつRX 5600の姿が見え始めた。  

以前より、RX 5700とRX 5500があるのだから中間のRX 5600もあるだろうと言われていたが、  
ようやくそれらしい情報が出てきた。  

[AMD Radeon RX 5600 series spotted at EEC - VideoCardz.com](https://videocardz.com/newz/amd-radeon-rx-5600-series-spotted-at-eec)  
Eurasian Economic Commission (EEC)に登録された製品名から、  
RX 5600シリーズには無印とXTがあり、メモリ容量にはどちらにも6/8GBのラインナップがある、というもの。  

メモリ容量からして、メモリ以外のコア部分を共通の仕様としていると思われる。  

### ダイ推測
まず、個人的にはNavi10のカットダウン版だと考えている。  
6GBという容量はメモリコントローラーを一部無効化し、メモリバス幅を192-bitとすれば可能だ。  

6/8GBという容量は、Navi14でも96/128-bitとし、16Gb GDDR6メモリチップを3/4枚使えば可能ではあるが、  
RX 5500シリーズでROPは32基すべてが有効とされており、上位製品としての差別化要因には1WGP（2CU）とクロックしかない。  
仮にそれで差別化を行なおうとしても、メモリバス幅を減らした分を補い、かつ上回るとは考えにくく、  
RX 5500 XTとRX 5600/XT 6GBとで性能差がほぼない、もしくはRX 5500 XTが勝つという結果になり得る。  
8GBでもRX 5600/XTと数字を変えるほどの性能差はないだろう。  

Navi10のカットダウンであれば、RX 5500 XTの上位製品としてもRX 5700の下位製品としても丁度良く収まるはずだ。  
ただそうなると疑問に思うのは6/8GBの違い、共通部分だ。  

### スペック推測

RX 5600シリーズでメモリバス幅の違いがあるとなると、RX 5600 8GBがRX 5600 XT 6GBに性能で勝つというチグハグなラインナップになりかねない。  
256-bitのまま6GBという構成は、4x 8Gb+4x 4Gbというメモリチップ混載で出来るのかもしれないが、  
GDDR6メモリチップはSamusung、Micron共に8Gbからで、4Gb製品はないことからこの世代では不可能と見ていいだろう。  
[GDDR6 Part Catalog - Micron](https://www.micron.com/products/graphics-memory/gddr6/part-catalog)  
[GDDR6 | Samsung Semiconductor Global Website](https://www.samsung.com/semiconductor/dram/gddr6/)  
SK Hynixも、公式のパーツカタログページこそ見つけられなかったが、各ニュースサイトの情報では8Gbとあり、4Gbを用意しているとは思いにくい。  
192-bitで4x 8Gb+2x 16Gbならあり得るかもしれないが、  
そもそも14Gbpsというメモリ速度でロット違いのチップを載せるのはリスキーな気がする。  
そのため、やはり異なるメモリバス幅と考える。  

有効WGP数に関しては、AMDGPUはこれまでShaderEngineが対称となるような構成を取ってきた。  
RDNAでもそれは有効だが、ShaderEngineあたりのShaderArrayが2基あり、ShaderArrayは対称である必要はなく、  
それとCUではなくWGP（2CU）単位という点で異なる。  
これにより2 ShaderEngineのNavi10で、RX 5500 XT以上RX 5700未満の性能に調整すると、14（28CU）か16WGP（32CU）となる。  
丁度2つのパターンがあり、2WGP（4CU）というのはRX 5700シリーズでの無印、XTの差と同数であり、差別化としては十分なはずだ。  

ただ、それがメモリバス幅64-bit埋められるほどの性能差かは、正直わからない。  

*もしかしたら* ROP数で調整することも考えられる。  
ROPもShaderEngineが対称になるように調整すると、2RB（8ROP）単位となるため、40/48/56/64ROPのパターンがある。  
6GBは56/64ROP、8GBは48/56ROPのようにメモリバス幅の違いを縮めるような調整をするか、  
無印は48/56ROP、XTは56/64ROPとし、無印とXTの差を広げるか、  
または、無印 6GB 56ROP、無印 8GB 48ROP、XT 6GB 64ROP、XT 8GB 56ROPのように、その両方の調整を行なうか。  

対抗先としてはGTX1660シリーズ、RTX 2060となるだろうから、最低でも48ROPにはなると考えているが、それ以上は自信がない。  
どうせなら性能が高い方が良いため、下手にROPを削らないことを願っている。  

<br>
以上をまとめた表。  
メモリ以外のスペックは推測。  

| Navi Product (guess) | RX 5600 6GB? | RX 5600 8GB? | RX 5600 XT 6GB? | RX 5600 XT 8GB? |
| :--- | :---: | :---: | :---: | :---: |
| WGPs | 14 | 14 | 16 | 16 |
| CUs | 28 | 28 | 32 | 32 |
| SPs | 1792 | 1792 | 2048 | 2048 |
| TMUs | 112 | 112 | 128 | 128 |
| ROPs | 48/56? | 48/56? | 56/64? | 56/64? |
||
| Memory Interface (bit) | 192 | 256 | 192 | 256 |
| Memory Speed (Gbps) | 14 | 14 | 14 | 14 |
| Memory Bandwidth (GB/s) | 336 | 448 | 336 | 448 |

TGPはRX 5500 XT（130W）とRX 5700（180W）の中間ということで、150Wくらいになると思われる。  

