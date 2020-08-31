---
title: "AMD Cezanne APU ベンチマーク結果　―― 8-Core/16-Thread, 3.6GHz、構成は Zen 3 + Vega か"
date: 2020-08-31T23:38:04+09:00
draft: false
tags: [ "Cezzane", "Renoir" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

オープンソースで開発され、クロスプラットフォームで動作するベンチマークソフト [Phoronix Test Suite](http://www.phoronix-test-suite.com/)の結果を集約する、高度なサイト [OpenBenchmarking.org](https://openbenchmarking.org/) から、AMD の次世代 APU と目されるコードネーム *Cezzane* のベンチマーク結果が見つかった。  

 * [Fdfdffdfd Benchmarks [2008302-NE-FDFDFFDFD18] - OpenBenchmarking.org](https://openbenchmarking.org/result/2008302-NE-FDFDFFDFD18)

CPU名は `AMD Eng Sample 100-000000263-30_Y` 、マザーボードは `Artic-CZN` 、CPUマイクロコードのバージョンは `0xa500009`。  
CPUクロックは 3.6GHz とされ、8-Core/16-Thread となっている。  
マザーボード名の `Artic` は *Renoir* にも使われていたもので、何らかのプラットフォーム、モバイル向けやデスクトップ向けかを指し示すものと思われるが、(自分の中で)はっきりとした答えは出てない。`CZN` は *Cezanne* の略だろう。  
マイクロコードの表記は、*Zen/+/2 系* に見られる `0x8` で始まるものと異なるため、それよりも新しい世代の CPU、*Zen 3 アーキテクチャ* の可能性が高い。  

GPU部は *Renoir* と同一とされ、GPUクロックは 1800MHz。  
Chipset と Audio の項目を見るに、GPU部だけでなく I/O部も *Renoir* と同一ではないかと思われる。  

AMD の次世代 APU には他に [Lucienna](/tags/lucienne) が確認されており、こちらは *Renoir* 同様に **Zen 2 + Vega(gfx909)** の構成を取る APU と考えている。  
対し、*Cezanne* は今回確認できた情報から、**Zen 3 + Vega(gfx909?)** の構成であると考えられる。  
ただ、これらがどのタイミングで登場するか、またターゲット帯はそれぞれどうなるか等の謎はある。  
*Renoir* はモバイル(Uシリーズ)、ハイエンドモバイル(Hシリーズ)、デスクトップ向け(Gシリーズ) を単一のダイでカバーしているが、  
*Renoir* の次世代であり、*Renoir* と似た構成を取る *Lucienne* が *Cezanne* の両者がどのように棲み分けるかは想像が難しい。  

