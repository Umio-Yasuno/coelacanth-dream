---
title: "PhoronixによるJCCエラッタ、TAAのベンチマーク結果まとめ"
date: 2019-11-28T05:53:29+09:00
draft: true
tags: [ "Errata", "Vulnerability" ]
keywords: [ "Intel", "JCC", "Errata", "TAA", "Vulnerability"  ]
categories: [ "Hardware", "Intel", "CPU", "Security" ]
---

とある理由からJCCエラッタとTAAの続報、というかPhoronixによるベンチマーク結果をまとめた。  

[The Combined Impact Of Mitigations On Cascade Lake Following Recent JCC Erratum + TAA](https://www.phoronix.com/scan.php?page=article&item=cascadelake-jcc-taa&num=1)  

Phoronixは検証に5パターンを使い、それぞれ


**No Mitigations + Old ucode** : 脆弱性対策を無効にし、マイクロコードもJCCエラッタ修正前のもの。  

**No Mitigations** : 脆弱性対策は無効にしてあるが、マイクロコードは最新のものでありJCCエラッタが修正されてある。  

**Default Mitigations** : 脆弱性対策はLinux Kernelのデフォルト設定にし、マイクロコードは最新のものを使用している。Linux KernelではTAAの対策としてTSXをデフォルトで無効。  

**Defalut Mitigations + TSX** : デフォルトでは無効化されるTSXを有効に。この場合、バッファを上書きするMD_CLEARが対策に使用される。その他脆弱性に関してはデフォルト設定。  

**Defalut Mitigations + No HT + TSX** : TSXは有効、HyperThreaddingは無効。脆弱性対策としてはパターン内で最も堅固。  


となっている。  

CPUは2ソケット Xeon Platinum 8280 (56 Core/ 112 Threads)、  
OSにはUbuntu 19.10、Linux Kernelのバージョンは5.4.0-rc7。  
JCCエラッタ修正による性能低下を抑える、GNU assemblerへのパッチはUbuntu 19.10でのパッケージ変更履歴を見る限り適用されていない。  


結果としては、性能が最も高いのは当然**"No Mitigations + Old ucode"**であり、性能に最も悪影響を受けているのは当然堅固な**"Defalut Mitigations + No HT + TSX"**となっている。  
やはりNo HTによる性能低下は大きく、レンダリングやエンコード、科学技術計算では無視できない程。  

他は概ね性能順に、**"No Mitigations"** > **"Default Mitigations"** > **"Defalut Mitigations + No HT + TSX"**。  
TSXを有効にしたパターンの方がそうでないパターンより影響を受けているのは、MD_CLEARによるものだろうか？  
そして、一般的なワークロード（画像処理、動画エンコード）における**"Default Mitigations"**での影響は微小だと言える。  


PhoronixはJCCエラッタ修正によるブラウザー性能への影響も検証しており、そちらでは平均的に4.2%の悪影響を受けるとある。  
[The Firefox + Chrome Web Browser Performance Impact From Intel's JCC Erratum Microcode Update](https://www.phoronix.com/scan.php?page=article&item=jcc-firefox-chrome&num=3)  
