---
title: "AMD Zen 3 アーキテクチャで分かってること、いないこと"
date: 2020-10-10T05:01:11+09:00
draft: false
tags: [ "Zen_3", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
toc: false
---

現地時間 2020/10/08 に AMD より、*Zen 3 アーキテクチャ* を採用する Ryzen 5000シリーズの製品ラインナップが発表された。  

 * [AMD Launches AMD Ryzen 5000 Series Desktop Processors: The Fastest Gaming CPUs in the World :: Advanced Micro Devices, Inc. (AMD)](https://ir.amd.com/news-events/press-releases/detail/972/amd-launches-amd-ryzen-5000-series-desktop-processors-the)
 * [Zen 3 Update - Ryzen 5000シリーズの詳報とRadeon RX 6000の性能 | マイナビニュース](https://news.mynavi.jp/article/20201009-1387569/)

Ryzen 5000シリーズは 2020/11/05 より発売開始される。  

| Model | Core/Thread | Base/Boost Clock | TDP |
| :-- | :--: | :--: | :--: |
| Ryzen 9 5950X | 16/32 | 3.4GHz/4.9GHz | 105W |
| Ryzen 9 5900X | 12/24 | 3.7GHz/4.8GHz | 105W |
| Ryzen 7 5800X | 8/16 | 3.8GHz/4.7GHz | 105W |
| Ryzen 5 5600X | 6/12 | 3.7GHz/4.6GHz | 65W |

### Zen 3 アーキテクチャで分かっていること

*Zen 2 アーキテクチャ* では *Zen/+ アーキテクチャ* と比較して IPC は 15% 向上したが、*Zen 3 アーキテクチャ* ではそこからさらに 19% 向上したとしている。  
ブーストクロックにおいても Ryzen 3000シリーズ (*Matisse*) よりも高いクロックを発揮する。  
また、CCX のレイアウトが変更され、それによりレイテンシを大きく削減している。  

*Zen 2 アーキテクチャ* とチップレット構成を採用する *Matisse /Rome* では、同 CCD にある CCX間のアクセスも、I/Oダイ側に存在する SDF(Scalable Data Fabric) を経由して行なうため、大きなレイテンシが発生する。  
アクセス帯域も、CCD と I/Oダイとの接続幅 (Write: 16Byte/Cycle, Read: 32Byte/Cycle) の制約を受けていた。  

{{< figure src="/image/2020/10/10/matisse-other-ccx.webp" caption="画像出典: [AMD Ryzen™ Processor Software Optimization - GPUOpen_Let’sBuild2020_AMD Ryzen™ Processor Software Optimization.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_AMD%20Ryzen%E2%84%A2%20Processor%20Software%20Optimization.pdf)" >}}

それが *Zen 3アーキテクチャ* では CCXあたり 8-Core、L3キャッシュ 32MB となったため、同 CCD 内で言えばレイテンシは解消され、アクセス帯域は大幅に向上したことになる。  

### Zen 3 アーキテクチャで分かっていないこと

プロセスは *Matisse* の時と変わらない TSMC 7nm FinFet としているが、*RDNA /GFX10* GPU 等にも用いられた TSMC N7 でも、EUV露光技術を用いる TSMC N7+ でもトランジスタ構造は FinFet であり、どちらも TSMC 7nm FinFet とすることは可能なはずである。  
他にも TSMC N7 を改良した TSMC N7P プロセスがあるため、Ryzen 5000シリーズの製造プロセスはやはりはっきりしない。  
TSMC N7P では EUV露光技術は使われず、TSMC N7 と同じ DUV となる。  

AMD は IPC 19% 向上の内訳こそ明かしているが、その詳細、マイクロアーキテクチャの変更点はまだ明かしていない。  
コンパイラにもまだ `znver3` をサポートするパッチは投稿されておらず、パッチか AMD の発表を待つ他ない。  
Ryzen 5000シリーズの発売までの間か、*Milan* の発表時あたりに公開されるとは思うが。  

{{< figure src="/image/2020/10/10/zen_2-arch.webp" title="Zen 2 マイクロアーキテクチャ" caption="画像出典: [AMD Ryzen™ Processor Software Optimization - GPUOpen_Let’sBuild2020_AMD Ryzen™ Processor Software Optimization.pdf](http://gpuopen.com/wp-content/uploads/slides/GPUOpen_Let%E2%80%99sBuild2020_AMD%20Ryzen%E2%84%A2%20Processor%20Software%20Optimization.pdf)" >}}

ゆるく推測するならば、内訳に Front End があるため、デコード幅または Opキャッシュからキューに送り込む命令数を増やした可能性が考えられる。  
Execute Engine に関しては、実行ユニットやリオーダーバッファを増やしたと考えられるが、それらをどれだけ増やしたかはやはり AMD による発表を待つしかない。  
