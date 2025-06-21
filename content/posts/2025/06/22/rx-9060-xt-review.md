---
title: "Radeon RX 9060 XT 16GB を購入したので簡易レビュー"
date: 2025-06-22T08:50:35+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU", "Diary" ]
tags: [ "gfx1200", "RDNA_4"]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

GIGABYTE Radeon RX 9060 XT GAMING OC 16GB を購入したので、Linux 環境におけるレビューを軽くしてみる。  

約 2年振りの GPU 換装となる。その時に購入したのは SAPPHIRE PULSE RX 6600 8G (RDNA 2, Navi23, CU 28基) であり、RDNA 3 世代の GPU が出てきて前世代のコスパがかなり良くなっていたタイミングだった。  
今後出てくるゲームに対して FullHD でテクスチャ品質等に妥協なく、そして 120-165fps を安定して出したいと思い、今回換装することにした。  

 * [Radeon™ RX 9060 XT GAMING OC 16G Key Features | Graphics Card - GIGABYTE Global](https://www.gigabyte.com/Graphics-Card/GV-R9060XTGAMING-OC-16GD)

## 購入までの経緯
最初は以前と同じように SAPPHIRE PULSE の 16GB モデルを購入するつもりだったが、最終的には GIGABYTE の 16GB モデルを購入した。  
理由はいくつかあり、まず SAPPHIRE の RX 9060 XT は映像出力端子が 2x HDMI + 1x DisplayPort という構成なのが自分には合わなかった。  
他社のモデルがほとんど 1x HDMI + 2x DisplayPort という構成を採っている中、HDMI 2基にしている点は SAPPHIRE 独自の強みだとは思うが、自分は基本 DisplayPort を使うようにしているため、そっちが多い方が嬉しい。  
それと Linux 環境における AMDGPU ドライバーでは、HDMI のフォーラムが仕様を公開することを拒否しているため、HDMI 2.1 の機能が実装されていない。[^hdmi2_1]  
自分は HDMI 2.1 対応のモニターを持っている訳ではないが、かといってこうした状況は HDMI を積極的に使う理由には決してならないだろう。  
後は単に売り切れていて買えなかったのもある。RX 9060 XT の販売開始に遅れ、自分がネットショップを確認した時にはカード長の短い 16GB モデルはほとんど売り切れていた。  
反面ファンを 3基搭載しているカード長の長いモデルは普通に残っており、その中から GIGABYE の 16GB モデルを選んだ。  
GIGABYTE を選んだのは、見た目がそこまで派手ではなく、カード長が極端に長くはない、それと今使っているマザーボードが GIGABYTE だから、とかそれくらいの理由である。  

[^hdmi2_1]: [HDMI Forum Closing Public Specification Access Is Hurting Open-Source GPU Drivers - Phoronix](https://www.phoronix.com/news/HDMI-Closed-Spec-Hurts-Open)

前世代となる RX 7700 XT、RX 7800 XT の価格が下がり、かなりコスパが良くなってはいるが、補助電源が 8-pin で済み、そして最新のアーキテクチャを使えるというのは RX 9060 XT の明確な強みだと思う。  

## GPU クーラー
三連ファン搭載の GPU カードを購入するのは初めてだったため、事前にクリアランスを確認してもなお今の PC ケースで問題なく使えるか少し不安だった。  
まあ実際に取り付けてみたら意外と余裕があった訳だが。  
ただ PowerColor Hellhound のような、特に長いモデルだとかなりギリギリだったかもしれない。  

GIGABYTE Radeon RX 9060 XT GAMING OC は短めの基板にファン 1基分近く長い GPU クーラーを組み合わせており、そしてバックプレートには開口部を設けていることでファンの風が一部吹き抜ける構造になっている。  
これはかなり効果的に、そして直線的に排熱できているらしく、GPU に負荷をしばらく掛けてると PC ケース天板が熱くなる。  
とりあえずの対策として、GPU に近い位置にある PC ケース前面のファンと背面の排気ファンの回転数を上げたところ、天板が少し熱くなりにくくなった、気はする。  

GPU クーラーの冷却性能はかなり優秀で、ゲーム実行時に電力制限である 182W 近くを消費し続ける状況でも GPU のジャンクション温度、メモリ温度ともに 80度前後で保たれていた。  

## 起動
Linux 環境における RX 9060 XT のサポートだが、Linux Kernel、Mesa、ROCm などは最新版であれば問題ないはずだ。  

Debian bookworm (stable) だと Mesa のバージョンが 22.3.6 と古く、RDNA 4 GPU がサポートされていないため注意。  
最初 Debian bookworm 環境で Mesa をビルドしてテストしようとしたのだが、KDE Plasma 5 と Mesa の最新バージョンの相性が悪く、ログイン画面が出なくなった。  
そのため、Debian trixie (testing) で環境を再構築した。  

AMD GPU 向け Vulkan ドライバーとしては Mesa の RADV/ACO ドライバー以外に、AMD がオープンソースで公開している AMDVLK と、プロプライエタリな AMDGPU Pro Vulkan が存在する。  
この内、AMDVLK のみまだ RX 9060 XT に対応したバージョンが公開されていないため、Mesa の RADV/ACO ドライバーを推奨したい。  

 * [GPUOpen-Drivers/AMDVLK: AMD Open Source Driver For Vulkan](https://github.com/GPUOpen-Drivers/AMDVLK)

GIGABYTE Radeon RX 9060 XT GAMING OC の電力制限は 0-182W の範囲で設定可能となっており、デフォルトでは最大の 182W に設定されていた。  
最大動作クロックだが、AMDGPU ドライバーから取得可能な情報だとゲームクロックと思われる 2840MHz が返される。  
だが、実際はそれよりも高いクロックまでブーストし、自分が確認した限りでは 3400MHz に達した。  
Windows 環境でのレビュー記事を読むに、Linux 環境では電力制限の解釈やブーストクロックの制御が異なるのだろう。  

## 性能
とりあえず RDNA 4 アーキテクチャの特徴である行列演算性能を確かめるために `vkpeak` を実行してみた。  
Mesa 25.1.3 ではまだ RADV/ACO ドライバーにおける `VK_KHR_shader_bfloat16` 拡張への対応がまだ取り込まれてなかったため、検証時点での最新版をビルドし、インストールしている。対象コミットは `061bc6151a11597f720535f00e79a5752e24aff6`。  

 * [nihui/vkpeak: A tool which profiles Vulkan devices to find their peak capacities](https://github.com/nihui/vkpeak)

fp32-vec4 の性能が公称ピーク FP32 性能である 25.6 TFLOPS の半分近くとなっているが、これは Dual issue (VOPD 命令) が機能していないためと思われる。`RADV_DEBUG=asm` 環境変数をセットして逆アセンブリ結果を見ても `v_dual_fmac_f32` 命令は使われていない。  
これは `vkpeak` が使用しているコンピュートシェーダー内で使用している式が `c = a * c + b` であり、VOPD 命令の `v_dual_fmac_f32` 命令の `c = a * b + c` の形式に合わず、コンパイラは `d = a * b + c` の形式である `v_fma_f32` 命令を使うしかないからではないかと思われる。  

fp16-matrix は `v_wmma_f16_16x16x16_f16` 命令が使われていることで、fp32-vec4 に対して 8倍、fp16-vec4 に対しては 4倍以上の性能となっている。  

fp64-vec4 は fp32-vec4 の約 1/27、内部的には FP32 性能に対して 1/32 (Dual issue が機能していない場合)、RDNA 3 と同じ演算性能レートと思われる。  

int8-matrix が約 194 TOPS とこれも公称ピーク INT8 性能の半分近くだが、これは RADV/ACO ドライバーが現状 16x16x16 の行列レイアウトのみに対応しており、また公称性能は `v_swmmac_i32_16x16x32_iu8` 命令を使った時のものと考えられるが、RADV/ACO ドライバーはまだその命令に対応していないからだと思われる。  

fp8-matrix, bf8-matrix の性能が 0 GFLOPS となっているが、これは RADV/ACO ドライバーが `VK_EXT_shader_float8` 拡張にまだ対応していないため。しかし、既にドライバー開発者たちによって対応作業が進められており、対応が main ブランチに取り込まれる日は近いと思われる。[^fp8]  

[^fp8]: [radv/gfx12,spirv,nir: VK_EXT_shader_float8 (!35434) · マージリクエスト · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/35434)

 >         device       = AMD Radeon Graphics (RADV GFX1200)
 >         
 >         fp32-scalar  = 17595.14 GFLOPS
 >         fp32-vec4    = 12141.84 GFLOPS
 >         
 >         fp16-scalar  = 16860.02 GFLOPS
 >         fp16-vec4    = 22352.11 GFLOPS
 >         fp16-matrix  = 106866.24 GFLOPS
 >         
 >         fp64-scalar  = 443.76 GFLOPS
 >         fp64-vec4    = 442.17 GFLOPS
 >         
 >         int32-scalar = 2797.28 GIOPS
 >         int32-vec4   = 2790.24 GIOPS
 >         
 >         int16-scalar = 15044.76 GIOPS
 >         int16-vec4   = 24125.62 GIOPS
 >         
 >         int8-dotprod = 53984.70 GIOPS
 >         int8-matrix  = 193619.41 GIOPS
 >         
 >         bf16-dotprod = 13134.32 GFLOPS
 >         bf16-matrix  = 106577.87 GFLOPS
 >         
 >         fp8-matrix   = 0.00 GFLOPS
 >         bf8-matrix   = 0.00 GFLOPS

ゲーム性能だが、重量級ゲームをあまり持っておらず、データがほとんど無い。  
ただ「Monster Hunter: World」と「ユミアのアトリエ ～追憶の錬金術士と幻創の地～」を実行した時のフレームレートを見るに、RX 6600 から倍以上のゲーム性能は出ているらしい。  
「ユミアのアトリエ」は、RX 6600 では FullHD 60fps の安定化にはいくつか設定を妥協する必要があったが、RX 9060 XT では各種設定で「高」を選択しても 109fps (1% Min) / 164fps (97% Percentile) 出ており、XeSS を無効化すると 127fps (1% Min) / 173fps (97% Percentile) 出ていた。  

……本当は RX 6600 のゲーム性能データをしっかり取るべきだとは思っているが、ベンチマーク検証に慣れていない自分にとって、問題が起きないように GPU を付け替え、ベンチマークを実行している間は静かに待ち、データは混在しないようしっかりと分け、そして最後にデータをまとめる、というのは結構ハードルが高い。  
こうして各ニュースサイトのレビュー記事の価値や、ベンチマーク結果を取る人たちの偉大さを知る。  

とはいえ気が向いたら追加の検証や比較を行った記事を書くかもしれない。  

