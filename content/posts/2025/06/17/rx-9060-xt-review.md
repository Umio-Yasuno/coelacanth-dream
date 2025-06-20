---
title: "Radeon RX 9060 XT 16GB を購入したのでレビュー"
date: 2025-06-17T18:07:26+09:00
draft: true
categories: [ "Hardware", "AMD", "GPU", "Diary" ]
tags: [ "gfx1200", "RDNA_4"]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

GIGABYTE Radeon RX 9060 XT GAMING OC 16GB を購入したので、Linux 環境におけるレビューを軽くしてみる。  

約 2年振りの GPU 換装となる。その時に購入したのは SAPPHIRE PULSE RX 6600 8G (RDNA 2, Navi23, CU 28基) であり、RDNA 3 世代の GPU が出てきて前世代のコスパがかなり良くなっていたタイミングだった。  
RX 6600 の限界を感じることが徐々に増え、どうせなら新しいモデルを買おうと前から思ってたのが今回換装した理由となる。  

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

## GPU クーラー
GIGABYTE Radeon RX 9060 XT GAMING OC は短めの基板にファン 1基分近く長い GPU クーラーを組み合わせており、そしてバックプレートには開口部を設けていることでファンの風が一部吹き抜ける構造になっている。  
これはかなり効果的に、そして直線的に排熱できているらしく、GPU に負荷をしばらく掛けてると PC ケース天板が熱くなるほどだ。  
実際の冷却性能もかなり優秀で、電力制限である 182W 近くを消費し続ける状況でも GPU のジャンクション温度、メモリ温度ともに 80度前後で保たれていた。  

## 起動
Linux 環境における RX 9060 XT のサポートだが、Linux Kernel、Mesa、ROCm などは最新版であれば問題ないはずだ。  
ただ、自分の環境は 
