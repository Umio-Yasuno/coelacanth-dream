---
title: "RX 5600 XTのスペックが判明?"
date: 2019-12-29T18:43:00+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi10", "GFX10", "gfx1010" ]
keywords: [ "Radeon", "Navi10", "RX 5600" ]
categories: [ "Hardware", "AMD", "GPU" ]
---
近くに発表、販売されると言われているRX 5600シリーズだが、XTの意外なスペックが判明した。  

（追記）確認したところ製品ページが消されていた。やはりまずかったのだろうか。<span class="hide">/image/2019/12/29/rx-5600-xt-produ
cta01.webp /image/2019/12/29/rx-5600-xt-pr
oducta02.webp</span>

| Navi Product | <span style="color:coral">RX 5600 XT ProductA</span> |
| :--- | :---: |
| WGPs | 18 |
| CUs | 36 |
| SPs | 2304 |
| TMUs | 144 |
| ROPs | 48 /56 ? |
||
| Base Clock | 1235 MHz |
| Game Clock | 1460 MHz |
| Boost Clock | 1620 MHz |
||
| Memory Size | 6 GB |
| Memory Interface | 192-bit |
| Memory Speed | 12 Gbps |
| Memory Bandwidth | 288 GB/s |
||
| FP16 (TFLOPS) | 14.92 |
| FP32 (TFLOPS) | 7.46 |

これまではRX 5700より減らした16WGP（32CU）とかだと予想していたが、それに反してRX 5700と同じ18WGP（36CU）だった。  
その分、クロックはだいぶ抑えられており、Game Clock、Boost Clockだけで見ればRX 5500Mに近い程だ。これがリファレンスのクロックかどうかはわからないが、明らかに攻めたクロックではないため、近いクロックではあるはずだ。  
GDDR6メモリもRX 5700/5500シリーズで採用されてきた14Gbpsではなく12Gbpsとなっている。  
ROP数は未だ不明のままだが、WGPがRX 5700と同数となると、64から減らして差別化を図る可能性は高い。  
TGPも不明だが、Recommended PSUが550Wであり、RX 5500とRX 5700の中間のため、TGPも中間の150Wあたりだと思われる。  

ダイ自体は（恐らく）Navi10を使いつつ、それ以外のメモリ部分でコストを削減したモデルと見られる。  
まだ推測の段階ではあるが、コストパフォーマンスはもちろん、ワットパフォーマンスでもかなり優秀なのではないだろうか。  

また、紹介文の中に

 > The AMD Radeon™ RX 5600 XT graphics card is designed for the ultimate 1080p gaming experience. Turn up your settings for higher fidelity and boost gaming performance for higher frame rates with ultra-fast response 

とあり、1080p144fpsをターゲットとしている、という予想は外れてなさそうで嬉しい。  

RX 5600のスペックは見つけられていないが、こちらはWGPを減らしてくると思う。  
製品の差別化において、どちらが上か下かを示すにはWGP（CU）の数を使うのが手っ取り早い。  
RX 5700との差別化でRX 5600 XTのクロック、メモリ周りを抑えるという手法は既に使ってしまったし、これ以上下げるとワットパフォーマンスがかえって悪くなったり、パッと見の印象も悪くなるのではないかと思う。  
ROP数も――理由はわからないが――アピールされないことが多く、差別化でもあまり積極的には出してこない。  
そういうことでWGPを減らすとしてやるなら、これまでの予想を継いで14WGPか16WGPだろうか。  

まあ近い内にRX 5600無印のスペックも明らかになるはずだ。  

<br>

| Navi Product | <span style="color:coral">RX 5500 XT</span> | <span style="color:coral">RX 5600 XT ProductA</span> | <span style="color:coral">RX 5700</span> |
| :--- | :---: | :---: | :---: |
| WGPs | 11 | 18 | 18 |
| CUs | 22 | 36 | 36 |
| SPs | 1408 | 2034 | 2304 |
| TMUs | 88 | 144 | 144 |
| ROPs | 32 | 48 /56 ? | 64 |
||
| Base Clock (MHz) | | 1235 | |
| Game Clock (MHz) | 1717 | 1460 | 1625 |
| Boost Clock (MHz) | 1845 | 1620 | 1725 |
||
| Max Memory Size | 8 GB | 6 GB | 8 GB |
| Memory Interface (bit) | 128 | 192 | 256 |
| Memory Speed (Gbps) | 14 | 12 | 14 |
| Memory Bandwidth (GB/s) | 224 | 288 | 448 |
||
| FP16 (TFLOPS) | 10.4 | 14.92 | 15.9 |
| FP32 (TFLOPS) | 5.2 | 7.46 | 7.95 |
||
| TGP | 130W | ~150W? | 180W |
| Launch Price | $169 | ~$269 ? | $379 |

<hr>
<code>RX 5600関連</code>

 * [Radeon RX 5600推測 ](/posts/2019/12/18/radeon-rx-5600-guess/)
 * [RX 5600情報アップデート & 推測 Part2](/posts/2019/12/22/rx-5600-info-update-guess/)
